# JWInventoryAPI

**JWInventoryAPI** is a virtual inventory GUI library for Endstone Bedrock servers, providing chest/hopper/dispenser menus with lock mode, edit mode, realtime sync, and per-slot click callbacks.

### What does it do?
It allows plugins to create virtual container GUIs (chests, double chests, hoppers, dispensers) that players can interact with. These aren't real blocks — they're client-side illusions that your plugin fully controls.

### Why is it useful?
- **For Server Owners:** Plugins that depend on JWInventoryAPI can show interactive chest menus, crate previews, shop GUIs, and inventory editors without placing real blocks in the world.
- **For Developers:** You get a clean API to create menus with per-slot click handlers, lock/edit modes, realtime multi-viewer sync, and menu queuing — all without dealing with raw Bedrock protocol packets yourself.

### How to use it?
Simply install the `.whl` file in your server's `plugins/` directory. It works as a library — other plugins depend on it.

## Features

- **Multiple Container Types:** Chest (27 slots), Double Chest (54 slots), Hopper (5 slots), Dispenser (9 slots)
- **Lock Mode:** Fully read-only, exploit-proof. Players cannot take, move, swap, or drag items under any circumstances.
- **Edit Mode:** Players can freely move items between the GUI and their inventory. Ideal for crate editors, inventory management, etc.
- **Click-Only Mode:** Read-only with per-slot click callbacks. Perfect for shop menus and navigation GUIs.
- **Realtime Sync:** Multiple viewers of the same menu see changes instantly via efficient per-slot updates.
- **Menu Queuing:** Opening a new menu while one is active queues it and opens after the current one closes.

## Developer API

### Creating a Menu

```python
from jwinventoryapi import Menu, MenuType
from endstone.inventory import ItemStack

menu = Menu(MenuType.CHEST, "My Menu")
menu.set_item(4, ItemStack("minecraft:diamond"), on_click=lambda p, s, i, inv: p.send_message("Clicked!"))
menu.send_to(player)
```

### Lock Mode (Crates, Previews, Animations)

```python
menu = Menu(MenuType.CHEST, "Preview")
menu.set_locked(True)
# Players cannot interact with items at all
menu.send_to(player)
```

### Edit Mode (Inventory Editing)

```python
menu = Menu(MenuType.DOUBLE_CHEST, "Edit Rewards")
menu.set_editable(True)
menu.set_close_listener(lambda p: save_items(menu.inventory))
menu.send_to(player)
```

### Per-Slot Click Callbacks

```python
def on_sword_click(player, slot, item, inventory):
    player.send_message("You clicked the sword!")
    menu.close(player)

menu.set_item(0, ItemStack("minecraft:diamond_sword"), on_click=on_sword_click)
```

### Adding Items

```python
# Set at specific slot
menu.set_item(4, ItemStack("minecraft:apple"))

# Add to first empty slot
menu.add_item(ItemStack("minecraft:stone"), on_click=my_callback)
```

### Listeners

```python
menu.set_listener(callback)        # Global click on any GUI slot
menu.set_place_listener(callback)  # Player clicks their own inventory while menu is open
menu.set_open_listener(callback)   # Fires when menu opens
menu.set_close_listener(callback)  # Fires when menu closes
```

### Closing

```python
menu.close(player)   # Close for one player
menu.close_all()     # Close for all viewers
```

### Viewers

```python
viewers = menu.get_viewers()  # list[Player] currently viewing this menu
```

### Batch Updates (for Animations)

When doing rapid visual updates (like crate rolling animations), use batch mode to suppress per-slot packets, then send a single full content update:

```python
inv = menu.inventory
inv.begin_batch()
for i in range(9):
    inv.set_item(i, some_item)
inv._dirty_slots.clear()
inv._batch_mode = False
menu.refresh_contents()
```

## UIInventory API

The `UIInventory` class mirrors Endstone's `Inventory` interface:

```python
inv = menu.inventory

inv.get_item(slot)                    # Get item at slot
inv.set_item(slot, item)              # Set item at slot
inv.add_item(item1, item2, ...)       # Add items to first available slots
inv.remove_item(item)                 # Remove matching items
inv.clear()                           # Clear all slots
inv.clear(slot)                       # Clear specific slot
inv.contains(item)                    # Check if inventory contains item
inv.contains_at_least(item, amount)   # Check minimum amount
inv.first_empty                       # First empty slot index (-1 if full)
inv.is_empty                          # True if all slots empty
inv.size                              # Number of slots
inv.contents                          # list[ItemStack | None]
```

## Menu Types Reference

| Type | Container | Slots | Paired |
|------|-----------|-------|--------|
| `MenuType.CHEST` | Single chest | 27 | No |
| `MenuType.DOUBLE_CHEST` | Double chest | 54 | Yes |
| `MenuType.HOPPER` | Hopper | 5 | No |
| `MenuType.DISPENSER` | Dispenser | 9 | No |

## Example 1

```python
from endstone import Player
from endstone.inventory import ItemStack
from endstone.plugin import Plugin
from jwinventoryapi import Menu, MenuType, UIInventory


class MyPlugin(Plugin):
    api_version = "0.10"
    prefix = "MyPlugin"
    load = "POSTWORLD"

    commands = {
        "shop": {
            "description": "Open the shop",
            "usages": ["/shop"],
        }
    }

    def on_enable(self):
        self.shop_menu = Menu(MenuType.CHEST, "Weapon Shop")

        sword = ItemStack("minecraft:diamond_sword")
        meta = sword.item_meta
        meta.display_name = "Sharp Sword"
        meta.add_enchant("sharpness", 5)
        sword.set_item_meta(meta)

        self.shop_menu.set_item(4, sword, on_click=self.buy_sword)
        self.shop_menu.set_item(8, ItemStack("minecraft:barrier"), on_click=lambda p, s, i, inv: self.shop_menu.close(p))

    def buy_sword(self, player: Player, slot: int, item: ItemStack, inventory: UIInventory):
        player.send_message("Purchased a Diamond Sword!")
        self.shop_menu.close(player)

    def on_command(self, sender, command, args):
        if command.name == "shop" and isinstance(sender, Player):
            self.shop_menu.send_to(sender)
        return True
```

## EXAMPLE 2
```python
from endstone import Player
from endstone.inventory import ItemStack
from endstone.plugin import Plugin
from jwinventoryapi import Menu, MenuType, UIInventory


class MyPlugin(Plugin):
    api_version = "0.10"
    prefix = "MyPlugin"
    load = "POSTWORLD"

    commands = {
        "shop": {
            "description": "Open the shop",
            "usages": ["/shop"],
        }
    }

    def on_enable(self) -> None:
        self.logger.info(f"JWInventoryExample is enable!")

    def on_command(self, sender, command, args) -> bool:
        if command.name == "shop" and isinstanceof(sender, Player)
            self.openMenu(sender)
        return True

    def openMenu(self, player: Player)
        menu = Menu(MenuType.CHEST)
        menu.set_name("Shop Menu")
        inv = menu.inventory
        inv.set_item(0, ItemStack("minecraft:diamond"))
        inv.set_item(1, ItemStack("minecraft.iron_ingot"))
        inv.set_item(3, ItemStack("minecraft:coal"))
        def on_click(player: Player, slot: int, item: ItemStack, uiinv: UIInventory) -> None:
            if slot is None and item.type == "minecraft:air"
                return

            menu.close(player)
            name = item.item_meta.display_name.replace("minecraft:", "")
            self.openShop(player, name)

        menu.set_listener(on_click)
        menu.send_to(player)

    def openShop(self, player: Player, name: str)
        menu = Menu(MenuType.HOPPER)
        menu.set_name("§e{name.upper()}")
        menu.send_to(player)
         

# Menu Creator

A reupload of AlexTheRegent's Menu Creator plugin with quality of life fixes.

## Original
- Author: AlexTheRegent
- Original repository: https://github.com/AlexTheRegent/Sourcemod-Menu-Creator

## Installation

1. Place `menucreator.smx` in your `addons/sourcemod/plugins` folder.
2. Create your menu configuration file at `addons/sourcemod/configs/menu_creator.txt`.
3. Restart your server or load the plugin with `sm plugins load menucreator`.

## Configuration File

The plugin reads `addons/sourcemod/configs/menu_creator.txt` on startup. Each line is a command with arguments separated by `|`. Spaces around the `|` separators are trimmed automatically. Lines starting with `//` are treated as comments.

```
// This is a comment
command | argument1 | argument2
```

### Handle Types

A "handle" is the internal name for any menu, panel, or list you create. There are three types:

| Type | Description |
|------|-------------|
| `menu` | Standard numbered menu with pagination. Players select options by pressing the corresponding number. Supports exit and "Exit Back" buttons. |
| `panel` | Custom-drawn panel with up to 9 items. Allows precise key position control via `setpos`. No automatic pagination. |
| `list` | Static pre-defined selection list. When a player picks an item, a command runs with the selected value substituted in. |

### Config Commands Reference

Commands are processed top to bottom. Most commands apply to the **most recently created handle** (the last `create` line above them).

#### `create` -- Create a new handle

```
create | handle_name | type
```

- `handle_name`: A unique internal name used to reference this handle elsewhere (e.g. in `setback`, `sm_mc_om`, etc.).
- `type`: One of `menu`, `panel`, or `list`.

Every config file must have at least one `create` before any other command.

#### `title` -- Set the title

```
title | Title Text Here
```

Sets the title displayed at the top of the current menu or panel. Supports [text aliases](#text-aliases).

#### `item` -- Add a clickable item

```
item | "Display Text" | "command_to_run"
```

Adds a selectable item. When a player clicks it, the command is executed as a server command. Supports [text aliases](#text-aliases) in the display text, and [user aliases](#user-aliases) + [command aliases](#command-aliases) in the command string.

For panels, items are automatically assigned to the next available key position (1-9).

#### `text` -- Add a non-clickable text line

```
text | Some informational text
```

Adds a line of text that cannot be selected. On panels, it renders as a raw text line. On menus, it appears as a disabled (grayed out) item. Supports [text aliases](#text-aliases).

#### `regcmd` -- Register a console command to open the handle

```
regcmd | sm_mymenu | 
regcmd | sm_adminmenu | b
```

- First argument: The console command players will type (e.g. `sm_mymenu` or `/mymenu` in chat).
- Second argument: Admin flag string (e.g. `b` for generic admin, `z` for root). Leave empty for no restriction (anyone can use it).

You can register multiple commands for the same handle by adding multiple `regcmd` lines.

#### `setback` -- Set the "back" target

```
setback | other_handle_name
```

Links the current handle's "Exit Back" / "Back" button to another handle. When the player presses back, they are taken to `other_handle_name`.

- For menus/lists, this enables the "Exit Back" button in the menu footer.
- For panels, this adds a "Back" text item that opens the target handle.

The target handle must already be defined (i.e., its `create` line must appear **before** this `setback`).

#### `setpos` -- Set the current key position (panels only)

```
setpos | 5
```

Sets the next key position for panel items. Valid values are 1-9. This lets you skip key positions or place items at specific slots. Only works with `panel` type handles.

#### `settime` -- Set the display timeout

```
settime | handle_name | 30
```

- `handle_name`: The handle to set the timeout for.
- Second argument: Time in seconds. Use `0` for no timeout (menu stays open forever).

If not set, the default timeout is `0` (forever).

#### `openonjoin` -- Auto-open on first join

```
openonjoin | 1
```

When set to `1`, `yes`, or `true`, the current handle will automatically be displayed to every player on their first `inventory_post_application` event after connecting. This is useful for welcome menus, rules screens, etc.

The menu only opens once per connection. If the player disconnects and reconnects, they will see it again.

**Note:** This feature hooks the `inventory_post_application` game event, which is available in TF2 and potentially other Source games that use this event. The event is only hooked if at least one handle has `openonjoin` enabled, so the plugin remains compatible with all Source games when the feature is unused.

---

### Text Aliases

These special tokens can be used in `title`, `item`, and `text` display strings:

| Alias | Replaced With |
|-------|---------------|
| `{nl}` | Newline character |
| `{s}` | Literal `\|` (pipe character, since `\|` is the argument separator) |
| `{ }` | Space (useful for leading/trailing spaces that would be trimmed) |

### User Aliases

These tokens can be used in item **commands** and are replaced with the acting player's information at the time the menu item is selected:

| Alias | Replaced With |
|-------|---------------|
| `{cl}` | Client index (integer) |
| `{uid}` | Client user ID (integer) |
| `{name}` | Client name (string) |

### Command Aliases

These tokens affect how the command string is processed by the server:

| Alias | Replaced With | Notes |
|-------|---------------|-------|
| `{q1}` | `;` | Must be at the start of the command. Allows chaining multiple server commands. |
| `{q2}` | `;` | Same as `{q1}`, alternative alias. |

---

## Server Commands

These server commands are registered by the plugin and can be used in item commands to open other handles, lists, URLs, or fake client commands.

### `sm_mc_om` -- Open Menu/Panel

```
sm_mc_om <client_index> <handle_name>
```

Opens the specified handle (menu or panel) for the given client. Typically used in item commands with the `{cl}` alias:

```
item | "Open Submenu" | "sm_mc_om {cl} my_submenu"
```

### `sm_mc_ol` -- Open Static List

```
sm_mc_ol <client_index> <handle_name> <command_on_select>
```

Opens a static list for the client. When the player selects an item, `command_on_select` is executed with the selected value substituted for `{handle_name}`.

```
item | "Pick from list" | "sm_mc_ol {cl} my_list sm_some_command {cl} {my_list}"
```

### `sm_mc_odl` -- Open Dynamic List (Player List)

```
sm_mc_odl <client_index> <list_type> <filter_state> <filter_team> <command_on_select>
```

Opens a dynamically generated player list. The list is populated with currently connected players matching the filters.

**List types:**

| Type | Selected Value |
|------|----------------|
| `clients1`, `clients2` | Client index of the selected player |
| `userids1`, `userids2` | User ID of the selected player |
| `name1`, `name2` | Name of the selected player |

**Filter state:**

| Value | Meaning |
|-------|---------|
| `0` | Dead players only |
| `1` | Alive players only |
| `2` | All players |

**Filter team:**

| Value | Meaning |
|-------|---------|
| `0` | All teams |
| `1` | Team 1 (spectators) |
| `2` | Team 2 |
| `3` | Team 3 |
| `4` | Any non-spectator team (team > 1) |

Example -- select an alive, non-spectator player by user ID:

```
item | "Select Player" | "sm_mc_odl {cl} userids1 1 4 sm_kick {userids1}"
```

### `sm_mc_ourl` -- Open URL (MOTD)

```
sm_mc_ourl <client_index> <url>
```

Opens a URL in the client's MOTD panel.

```
item | "Visit Website" | "sm_mc_ourl {cl} https://example.com"
```

### `sm_mc_fc` -- Fake Client Command

```
sm_mc_fc <client_index> <command>
```

Makes the client execute a console command as if they typed it themselves.

```
item | "Say Hello" | "sm_mc_fc {cl} say Hello everyone!"
```

---

## ConVars

| ConVar | Default | Description |
|--------|---------|-------------|
| `sm_mc_onpostadmin` | `""` | A command to execute as each player after they connect and pass admin checks. Useful for automatically opening a menu on join. Example: `sm_mc_onpostadmin "sm_mymenu"` |

The ConVar is saved to `cfg/sourcemod/menu_creator.cfg` after first run.

---

## Full Config Example

```
// ========================================
// Welcome menu - shown automatically on join
// ========================================
create | welcome | menu
title | Welcome to Our Server!{nl}Please read the rules.
item | "Server Rules" | "sm_mc_om {cl} rules"
item | "Visit our website" | "sm_mc_ourl {cl} https://example.com"
text | { }
text | Thanks for playing!
settime | welcome | 0
openonjoin | 1

// ========================================
// Rules panel
// ========================================
create | rules | panel
title | Server Rules
text | 1. Be respectful
text | 2. No cheating
text | 3. Have fun!
setpos | 9
item | "Back" | "sm_mc_om {cl} welcome"
setback | welcome

// ========================================
// Admin menu - restricted to generic admins
// ========================================
create | admin_menu | menu
title | Admin Menu
item | "Kick a player" | "sm_mc_odl {cl} userids1 2 0 sm_kick #{userids1}"
item | "Slay a player" | "sm_mc_odl {cl} clients1 1 4 sm_slay {clients1}"
item | "Change map to de_dust2" | "changelevel de_dust2"
regcmd | sm_amenu | b

// ========================================
// Public menu - anyone can use it
// ========================================
create | pub_menu | menu
title | Server Menu
item | "Rules" | "sm_mc_om {cl} rules"
item | "Admin Menu" | "sm_mc_om {cl} admin_menu"
regcmd | sm_menu |
regcmd | sm_servermenu |
settime | pub_menu | 30
```

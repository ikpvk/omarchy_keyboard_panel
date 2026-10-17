# Omarchy Panel Position

Adds a **Style → Panel Position** setting to the Omarchy menu. It decides where the shell's bar panels open:

- **Default**: under the panel's bar icon (Omarchy's standard behaviour)
- **Centre**: in the middle of the screen

The setting applies right away, without restarting the shell. It covers every panel built on Omarchy's `KeyboardPanel` component:

- **Built-in:** Agents, Audio, Bluetooth, Clock, Dropbox, Display, Network, Power, Tailscale and Weather
- **Third-party:** any plugin that uses `KeyboardPanel`, such as `crmne.hyprmoncfg`

System tray and Media popups use a different component (`PopupCard`), so they still open under their icons.

Tested on Omarchy **4.0.4-1** (Quattro).

## How it works

Omarchy has no setting for panel placement. Every bar panel gets its position from a single property, `cardOrigin`, in
`/usr/share/omarchy/shell/Ui/KeyboardPanel.qml`. This tool makes a small, clearly marked edit to that property (see
[`KeyboardPanel.qml.patch`](KeyboardPanel.qml.patch)). After the edit:

1. Each panel watches `~/.config/omarchy/panel-position`.
2. When the file contains `centre`, the panel opens centred on its screen. The existing guards still keep it on screen.
3. Any other value, or no file at all, gives Omarchy's original placement.

`KeyboardPanel.qml` belongs to the `omarchy` package, so every Omarchy update replaces it. A `post-update` hook
re-applies the edit after `omarchy update`.

| Piece | Location |
|---|---|
| Tool | `~/.local/bin/omarchy-panel-position` |
| Setting | `~/.config/omarchy/panel-position` (`default` or `centre`) |
| Menu entries | marked block in `~/.config/omarchy/extensions/omarchy-menu.jsonc` |
| Update hook | `~/.config/omarchy/hooks/post-update.d/panel-position` |
| Original-file backup | `~/.local/share/omarchy-panel-position/` |
| Patched file | `/usr/share/omarchy/shell/Ui/KeyboardPanel.qml` |

### Safety checks

- **Only patches the stock file.** The tool runs `pacman -Qkk` to confirm `KeyboardPanel.qml` still matches the omarchy package before editing it.
- **Skips the patch if the code has changed.** If an Omarchy release changes the code the patch looks for, the hook leaves the file alone and sends a desktop notification. Panels then open in their default position, and nothing breaks.
- **Restores only a matching backup.** Uninstall puts back the saved original only when it matches the installed omarchy version. It then checks the result against the package checksum.
- **Asks for root once.** The only root step is writing `KeyboardPanel.qml`. It uses `sudo` in a terminal and `pkexec` otherwise.

## Install

```bash
install -m 755 omarchy-panel-position ~/.local/bin/omarchy-panel-position
omarchy-panel-position install
```

The install command:

1. Creates the setting file with `default`.
2. Adds the menu entries.
3. Installs the update hook.
4. Backs up and patches `KeyboardPanel.qml`. You'll be asked for your password.
5. Restarts the shell.

## Use

Open the Omarchy menu and go to **Style → Panel Position**, then pick **Default** or **Centre**. A ✓ marks the active choice.

You can do the same from a terminal:

```bash
omarchy-panel-position set centre     # or: set default
omarchy-panel-position get            # prints default|centre
omarchy-panel-position status         # position, patch, menu, hook, omarchy version
omarchy-panel-position reapply        # re-patch by hand
```

Run `reapply` if Omarchy was upgraded through `pacman` or `yay` directly. Those upgrades don't run Omarchy hooks.

## Uninstall

```bash
omarchy-panel-position uninstall
rm ~/.local/bin/omarchy-panel-position
```

The uninstall command:

1. Removes the menu block and the update hook.
2. Restores Omarchy's original `KeyboardPanel.qml` and checks it against the package.
3. Deletes the setting and the backup.
4. Restarts the shell.

If there's no backup that matches the installed omarchy version, uninstall stops and does nothing more. In that case,
restore the file by reinstalling the package, but only on a fully updated system:

```bash
sudo pacman -S omarchy
```

## Notes

- **`pacman -Qkk omarchy` lists the file as modified.** That's expected while the patch is applied.
- **Centring only moves the panel.** You don't get a dimmed backdrop like the Coinbase plugin's panel.
- **This could become an upstream setting.** Omarchy already has a `centerOnBar` option in `KeyboardPanel`, so a
  setting such as `"panelPosition": "center"` in `shell.json` would fit naturally. If Omarchy adds one, uninstall
  this tool and use that setting.

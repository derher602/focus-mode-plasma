# focus-mode-plasma
Watchdog that kills apps and mutes notifs on KDE Plasma.

Turn on Plasma's native **Do Not Disturb** and focus mode kicks in:

- **Apps**: distracting apps (Discord, Steam, Spotify, …) are killed every few seconds while focus is on.
- **Notifications**: fully muted, even if DND gets switched back off.
- **Lock**: optionally lock focus mode for N minutes. Until it runs out, focus can only be ended with `force-off` (confirmation + password).

## Requirements

- Distros running **KDE Plasma 6**
- `bash` 4.4+, `python3`
- `kdialog`: popups when there's no terminal
- `kreadconfig6`: reads Plasma's DND state (ships with Plasma)
- `pkexec` / polkit: password prompt for `force-off`
- `python3-pyyaml`: only if you use a YAML list file (JSON needs nothing extra)
- Optional: `flatpak` (to kill Flatpak apps), `kde-inhibit` (otherwise falls back to `python3-dbus` or PyGObject for muting)

```sh
sudo dnf install kdialog python3-pyyaml polkit # For Fedora-based systems, other distros, YMMV
```

## Install

> [!WARNING]  
> This script requires **ROOT**, This code has been tested (i know right) and works correctly (nor do i have the need for your sensitive info). I'm not liable for system destruction, always be careful when running things as root.

```sh

git clone https://github.com/derher602/focus-mode-plasma.git # Clone the repo

# 1. The list file (blocked apps). Root-owned so changing it needs sudo.
sudo install -Dm644 -o root -g root config.example.yaml /etc/focus-mode/config.yaml
#    …or the JSON version:
# sudo install -Dm644 -o root -g root config.example.json /etc/focus-mode/config.json

# 2. The script, somewhere on your PATH
install -Dm755 focus-mode ~/.local/bin/focus-mode

# 3. Start the DND watcher now and at every login (KDE autostart)
focus-mode install
```

## The list file

Blocked apps are kept in `/etc/focus-mode/config.yaml` (or `config.yml` / `config.json`; if several exist, YAML wins).
It's root-owned on purpose: editing it needs `sudo`, so you can't quietly unblock an app in the middle of a focus session.

```yaml
blocked_apps:
  processes:          # killed with `pkill -i -x` (exact name, any case)
    - discord
    - steam
  flatpaks:           # killed with `flatpak kill`
    - com.discordapp.Discord
```

The same thing in JSON:

```json
{
  "blocked_apps": {
    "processes": ["discord", "steam"],
    "flatpaks": ["com.discordapp.Discord"]
  }
}
```

- **Process names**: run the app, then `ps -eo comm | sort -u | grep -i <app>`.
- **Flatpak IDs**: `flatpak list --app`.
- A section you leave out counts as an empty list.

To edit it: `sudo nano /etc/focus-mode/config.yaml`. To check what focus mode will use: `focus-mode list`.
Changes take effect the **next time** focus mode is turned on.

## Usage

The normal flow: toggle **Do Not Disturb** in Plasma's notification tray. When DND turns on, the watcher shows a warning and asks how long to lock. When DND turns off, focus ends (unless it's locked; then it ends when the lock expires).

| Command | What it does |
|---|---|
| `focus-mode install` | Add the watcher to KDE autostart and start it |
| `focus-mode watch` | Run the DND watcher in the foreground (what autostart runs) |
| `focus-mode toggle` | Manual on/off, handy to bind to a keyboard shortcut |
| `focus-mode on [minutes]` | Turn on, optionally locked for N minutes |
| `focus-mode off` | Turn off (refused while locked) |
| `focus-mode force-off` | Turn off even while locked (confirmation + password) |
| `focus-mode status` | Show focus/lock/DND/watcher state |
| `focus-mode list` | Show the blocked apps from the list file |
| `focus-mode dnd-state` | Debug: show Plasma's raw DND value |

Other settings (`GUARD_INTERVAL`, `DEFAULT_LOCK_MIN`, `POLL`) are at the top of `focus-mode`.
The watcher logs to `$XDG_RUNTIME_DIR/focus-mode/watch.log`.

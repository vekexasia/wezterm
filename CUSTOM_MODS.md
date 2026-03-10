# Custom Modifications (vekexasia/wezterm)

This fork includes the following modifications on top of upstream [wezterm/wezterm](https://github.com/wezterm/wezterm).

---

## 1. Notification Click → Focus Originating Pane/Tab

**Source:** cherry-picked from [ferologics/wezterm@90e88e6](https://github.com/ferologics/wezterm/commit/90e88e6) (`fix/notification-focus-pane`)

When a toast notification is triggered (e.g. via OSC 777), clicking it now focuses the pane and tab that sent it. Previously, clicking a notification did nothing beyond dismissing it.

### What changed

- **`wezterm-toast-notification`**: Added an `on_click` callback field to `ToastNotification`. Platform-specific implementations (Windows, macOS, D-Bus/Linux) invoke this callback when the user clicks the notification.
- **`wezterm-gui/src/frontend.rs`**: When a `ToastNotification` alert is received with `focus: true`, an `on_click` closure is attached that calls `mux.focus_pane_and_containing_tab(pane_id)` to switch to the originating pane/tab, **and** brings the OS window to the foreground via `gui_win.window.focus()` — this also switches virtual desktops on Windows when the notification originates from a window on a different desktop.

### New config option: `notification_handling`

Controls whether toast notifications are shown based on which pane/tab/window is currently focused.

```lua
-- In your .wezterm.lua
config.notification_handling = "SuppressFromFocusedTab"
```

| Value                        | Behavior                                                  |
|------------------------------|-----------------------------------------------------------|
| `"AlwaysShow"` *(default)*   | Always show notifications                                 |
| `"NeverShow"`                | Never show any notifications                              |
| `"SuppressFromFocusedPane"`  | Suppress if the notification comes from the focused pane  |
| `"SuppressFromFocusedTab"`   | Suppress if it comes from any pane in the focused tab     |
| `"SuppressFromFocusedWindow"`| Suppress if it comes from any pane in the focused window  |

### Files modified

| File | Summary |
|------|---------|
| `wezterm-toast-notification/src/lib.rs` | Added `on_click: Option<Box<dyn FnOnce() + Send>>` to `ToastNotification`, custom `Debug` impl |
| `wezterm-toast-notification/src/windows.rs` | Invoke `on_click` in the `Activated` handler |
| `wezterm-toast-notification/src/macos.rs` | `CLICK_CALLBACKS` registry, invoke on delegate response |
| `wezterm-toast-notification/src/dbus.rs` | Invoke `on_click` on D-Bus action |
| `wezterm-gui/src/frontend.rs` | Wire up `on_click` to `focus_pane_and_containing_tab`, add `notification_handling` filtering |
| `wezterm-gui/src/scripting/guiwin.rs` | Pass `on_click: None` for Lua-scripted notifications |
| `wezterm-font/src/lib.rs` | Pass `on_click: None` for font fallback notifications |
| `config/src/config.rs` | `NotificationHandling` enum with 5 variants |

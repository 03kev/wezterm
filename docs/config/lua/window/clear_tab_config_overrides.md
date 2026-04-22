# `window:clear_tab_config_overrides(tab)`

Clears any configuration overrides that were set for the specified tab.

This is equivalent to restoring that tab to the inherited configuration from
the base config and any window-level overrides.

When the specified tab is active, clearing the overrides causes the effective
configuration for the window to be reloaded immediately.

Example:

```lua
local wezterm = require 'wezterm'

wezterm.on('reset-active-tab-theme', function(window, pane)
  local tab = window:active_tab()
  if not tab then
    return
  end

  window:clear_tab_config_overrides(tab)
end)

return {
  keys = {
    {
      key = 'R',
      mods = 'CTRL|SHIFT',
      action = wezterm.action.EmitEvent 'reset-active-tab-theme',
    },
  },
}
```

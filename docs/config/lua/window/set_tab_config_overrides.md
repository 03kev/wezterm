# `window:set_tab_config_overrides(tab, overrides)`

Changes the set of configuration overrides for the specified tab.

This method is analogous to
[window:set_config_overrides()](set_config_overrides.md), but the overrides are
associated with a specific tab rather than the window as a whole.

When the specified tab is active, the config file is re-evaluated and any CLI
overrides are applied, followed by the window overrides and then the tab
overrides.

If you are calling this method from inside an event handler that can be
triggered by a configuration reload, take care to only call it when the actual
override values have changed to avoid a loop.

Example:

```lua
local wezterm = require 'wezterm'

wezterm.on('toggle-active-tab-opacity', function(window, pane)
  local tab = window:active_tab()
  if not tab then
    return
  end

  local overrides = window:get_tab_config_overrides(tab) or {}
  if not overrides.window_background_opacity then
    overrides.window_background_opacity = 0.5
  else
    overrides.window_background_opacity = nil
  end
  window:set_tab_config_overrides(tab, overrides)
end)

return {
  keys = {
    {
      key = 'B',
      mods = 'CTRL',
      action = wezterm.action.EmitEvent 'toggle-active-tab-opacity',
    },
  },
}
```

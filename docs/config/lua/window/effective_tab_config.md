# `window:effective_tab_config(tab)`

Returns a lua table representing the effective configuration for the specified
tab.

This method is analogous to
[window:effective_config()](effective_config.md), but resolves the config that
would apply when the specified tab is active.

The returned table includes the base config, any CLI overrides, any window
overrides, and then any overrides that were set specifically for the tab.

Note: changing the returned table does NOT change the effective tab config; it
is just a copy of that information.

If you want to change the configuration for a tab, look at
[set_tab_config_overrides](set_tab_config_overrides.md).

Example:

```lua
local wezterm = require 'wezterm'

wezterm.on('show-active-tab-opacity', function(window, pane)
  local tab = window:active_tab()
  if not tab then
    return
  end

  wezterm.log_info(window:effective_tab_config(tab).window_background_opacity)
end)
```

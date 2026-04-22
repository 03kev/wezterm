# `window:effective_tab_config_overrides(tab)`

Returns a copy of the effective configuration overrides for the specified tab.

This method merges the window overrides with any overrides that were set
specifically for the tab, using the same precedence that is applied when the
tab becomes active.

This differs from
[window:get_tab_config_overrides()](get_tab_config_overrides.md), which returns
only the overrides that were explicitly set on that tab.

If neither the window nor the tab has overrides, this method returns `nil`.

Example:

```lua
local wezterm = require 'wezterm'

wezterm.on('show-active-tab-theme-mode', function(window, pane)
  local tab = window:active_tab()
  if not tab then
    return
  end

  local overrides = window:effective_tab_config_overrides(tab) or {}
  wezterm.log_info(overrides.theme_mode or 'unset')
end)
```

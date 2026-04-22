# `window:get_tab_config_overrides(tab)`

Returns a copy of the current set of configuration overrides that is in effect
for the specified tab.

This method is analogous to
[window:get_config_overrides()](get_config_overrides.md), but the overrides are
associated with a specific tab rather than the window as a whole.

If no overrides have been set for the tab, this method returns `nil`.

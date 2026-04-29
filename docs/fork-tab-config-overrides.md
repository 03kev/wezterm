# Fork note: tab config overrides

This fork carries a small set of changes that make it possible to keep runtime
configuration, such as a light/dark theme, scoped to a single tab instead of the
whole window.

This is not upstream WezTerm behavior. Treat this file as a maintenance note for
the fork.

## Goal

Upstream `window:set_config_overrides()` is window-scoped: changing colors with
that API changes every tab in the window. This fork adds tab-scoped config
overrides so a Lua config can store a different effective config per tab.

The intended theme flow is:

1. A shell or editor in a pane publishes a runtime value, for example
   `theme_mode=light`, using a WezTerm user var.
2. The Lua config receives `user-var-changed`.
3. The Lua config resolves the desired colors and stores them in the current
   tab's config overrides.
4. WezTerm applies that tab's effective config to panes in that tab.
5. When a new split pane is created in that tab, the new process is spawned with
   the tab's effective config.

WezTerm itself should not know what `theme_mode`, `ZSH_THEME_MODE`, or
`NVIM_THEME` mean. Those are dotfile/application-level concepts. The fork only
provides tab-scoped config plumbing.

## Added Lua API

The fork adds window methods for managing tab-specific overrides:

- `window:set_tab_config_overrides(tab, overrides)`
- `window:get_tab_config_overrides(tab)`
- `window:clear_tab_config_overrides(tab)`
- `window:effective_tab_config(tab)`
- `window:effective_tab_config_overrides(tab)`

These mirror the window-level override APIs, but take a `MuxTab` argument.

## Example: per-tab theme

A Lua config can store tab-local colors like this:

```lua
local function apply_tab_theme(window, pane, mode)
  local tab = pane:tab()
  local overrides = window:get_tab_config_overrides(tab) or {}

  overrides.theme_mode = mode
  overrides.colors = colors_for_mode(mode)

  window:set_tab_config_overrides(tab, overrides)
end

wezterm.on("user-var-changed", function(window, pane, name, value)
  if name == "theme_mode" and (value == "light" or value == "dark") then
    apply_tab_theme(window, pane, value)
  end
end)
```

`theme_mode` in this example is just app-owned metadata stored alongside the real
WezTerm config fields. The important WezTerm field is `colors`.

## New split panes

The fork also changes split-pane spawning. For `SplitPane`, the GUI now resolves
the active tab's effective config and passes it through the mux/domain spawn
path. That means config fields such as `set_environment_variables` from the tab
override are applied when constructing the new shell process.

For shell/editor theme integration, the Lua config should put the required env
vars in the tab override:

```lua
overrides.set_environment_variables = overrides.set_environment_variables or {}
overrides.set_environment_variables.ZSH_THEME_MODE = mode
overrides.set_environment_variables.NVIM_THEME = mode
```

This keeps the fork generic: WezTerm propagates tab config, while dotfiles decide
which environment variables are meaningful.

## Existing panes

Changing a tab override updates WezTerm's terminal config for panes in that tab.
This is enough for terminal-owned rendering state, such as configured colors, to
be associated with the tab.

It does not mutate the environment of shell processes that are already running.
For example, a zsh process that started with `ZSH_THEME_MODE=dark` will not
magically have its process environment changed to `light`. If the prompt itself
needs to change in already-open panes, that requires a separate application-level
sync mechanism in the shell/editor config.

In practice:

- New panes can inherit the tab theme via `set_environment_variables`.
- Existing panes get the terminal config update.
- Existing shell prompts may need their own reload/sync logic if they cache theme
  state in process environment variables.

## Implementation map

Relevant fork commits:

- `0624df537 Add per-tab config overrides`
- `b30a082e5 Apply per-tab config to terminal panes`
- `854997846 Fix tab theme reload after active tab changes`
- `76d45756f Apply tab config when spawning split panes`

Important code paths:

- `wezterm-gui/src/scripting/guiwin.rs`: exposes the Lua window methods.
- `wezterm-gui/src/termwindow/mod.rs`: stores per-tab override state and
  recomputes effective config.
- `wezterm-gui/src/termwindow/spawn.rs`: chooses the active tab's effective
  config for split-pane spawns.
- `wezterm-gui/src/spawn.rs`, `mux/src/domain.rs`, `mux/src/lib.rs`: pass the
  effective config through the spawn path so domains can apply it while building
  the command.

## Known limits

- The API is fork-local and may conflict with future upstream names.
- `ConfigHandle` is not comparable, so `SplitSource` no longer derives
  `PartialEq`.
- The split-spawn config path primarily matters for local domains. Remote/tmux
  paths may need separate handling if they need equivalent runtime environment
  propagation.
- Existing process environments are out of scope for the fork.

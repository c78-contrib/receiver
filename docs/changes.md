# Changes — Receiver v0.6.0

## Background execution

The app now stays alive in the background when the window is closed, keeping playback active. Re-launching the app restores the existing window instead of creating a new instance.

- Window hides on close instead of being destroyed, preserving playback
- Instance is now unique (`ApplicationFlags.NON_UNIQUE` removed) so re-launching calls `present()` on the existing window
- `window.vala:34-44` — `close_request` returns `true` (hide) instead of `false` (destroy)
- `application.vala:9,63-65,103` — `db_opened` guard prevents redundant `open_database()` calls on re-activation

## Run in Background (toggle)

A new checkbox menu item lets the user control whether the app stays in the background.

- **GSettings key** `run-in-background` (`b`, default `true`) added to `data/...gschema.xml`
- **Menu item** "Run in Background" added to the hamburger menu between "Song History" and "About Receiver"
- **Stateful action** `win.run-in-background` created in `setup_win_actions()`, synced to GSettings via `notify["state"]`
- **Conditional close**: if nothing is playing (`STOPPED` or `ERROR`), the app closes fully regardless of the flag; the flag is only consulted when playback is active (`PLAYING` or `PAUSED`)

## Window position save/restore (X11)

The window position is now remembered and restored across sessions on X11.

- **GSettings keys** `window-x` and `window-y` (`i`, default `-1` = unset)
- **meson.build**: optional `x11` dependency detected at build time; defines `--define=X11` and links `gtk4-x11`/`x11` Vala packages when available
- `save_window_position()` uses `XGetWindowAttributes` in `close_request`
- `restore_window_position()` uses `XMoveWindow` in `map` signal, deferred via `Idle.add` for WM placement
- Gracefully skipped on Wayland (X11 APIs not available at runtime)

## Translations

126 `.po` files updated to include the new "Run in background" string.

- `.pot` regenerated via `make translations-update`
- All 126 language files received translations for the new string

## Infrastructure

- **Version** bumped from 0.5.0 to 0.6.0 in `meson.build`, `debian/changelog`, and `snap/snapcraft.yaml`
- **AGENTS.md** added with build commands, architecture overview, and repo-specific quirks

---

### Files modified

| File | Change |
|---|---|
| `src/widgets/window.vala` | Background execution, position save/restore, run-in-background toggle, conditional close |
| `src/application.vala` | Single-instance (`NON_UNIQUE` removed), `db_opened` guard |
| `data/io.github.meehow.Receiver.gschema.xml` | Added `window-x`, `window-y`, `run-in-background` keys |
| `meson.build` | Optional X11 dependency, version bump |
| `debian/changelog` | Version bump |
| `snap/snapcraft.yaml` | Version bump |
| `AGENTS.md` | New file — agent instructions |
| `po/*.po` (126 files) | Translations for "Run in background" |
| `po/receiver.pot` | Regenerated template |

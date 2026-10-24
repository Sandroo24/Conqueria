# Build bar, sea mines removed, web font safety (this round)
STATUS: written + statically checked only (balanced brackets, tab indentation, no repeated variable names in one scope, no leftover calls to removed functions).
**Not opened in Godot, no web export made** (no Godot in my environment). Test before you rely on it.

## 1. One main bar
The bottom bar is now ONE row of nine cards, hotkeys 1-9:
`1 Village, 2 Station, 3 Port, 4 Warship, 5 Bomb Launcher, 6 SAM Site, 7 Small Bomb, 8 Nuke, 9 Mega Bomb` (`MAIN_BAR` in main.gd).
The two small arrow buttons and their pop-up menus are gone. (`sub_panel`, `nuke_panel`, `_toggle_sub`, `_toggle_nuke`, `_style_arrow`, `ArrowBtn` still exist as empty / unused code so nothing can hit a null.)
Locked cards (warship without a port, bombs without a ready launcher) still show "needs port" / "launcher Lv N" and dim as before. The bar shrinks on narrow windows (`_compact_ui` now counts 9 cards).

## 2. Sea mines removed completely
Removed: the card, price, tip, hotkey (old 5), validation (`sea` command now only accepts `warship`), placement, bot behaviour (`_bot_mine`, `_count_mines`, the mine option in `bot_build`), the per-tick `step_mines`, the drawing and the `_draw_mine` art, the nuke clean-up code, the `mines` array, `SEA_RANGE`, `MINE_TRIGGER`, the now unused `_sea_reach` helper and `bought["mine"]`.
**Gameplay changed, so `NET_VER` and `PROTO_VERSION` are now 21** (main.gd and server/relay.py). Re-deploy relay.py and re-export; an old cached itch page will get a clear "version mismatch" message instead of desyncing.

## 3. Web text ("Chinese letters")
Cause: GFS Baskerville is a GREEK-only font (it has no A-Z). Browsers have no system fonts, so every missing letter was replaced by whatever glyphs were available, which looked like Chinese. The main.gd in your zip already used only Lora for text, so a build exported from THIS code should not use the Greek font at all. If your itch page still shows it, it is an OLD export or a cached page: re-export, upload the new ZIP and hard-refresh (Ctrl+F5).
Extra safety added in `_make_theme`: it tests that the font can draw every plain character (space to ~) and otherwise uses Godot's built-in font; system-font fallback is switched off for Lora; on the web build the bold / heading weights use a simple synthetic weight instead of the variable-font axis. `fonts/GFSBaskerville.otf` was deleted from the project (unused).
Export tip: export_presets.cfg already uses "all_resources", so `fonts/Lora-Variable.ttf` is included; open the project once in Godot so the font gets imported before exporting.

## Round 2: troops melted too fast when expanding
Cause: taking FREE land cost `80 / 5 * of_k` troops per tile (of_k = how many OpenFront tiles one of our tiles stands for) and, unlike attacks on other nations, no multiplier was applied. At the start the front is small but each tile is expensive, so a 20% send (5,000 troops) was gone in about a second or two.
Fix: new constant `EXPAND_LOSS_MULT = 0.35` (top of main.gd) multiplies the loss per tile of attacks on free land only. Attacks on other nations are unchanged (`ATK_LOSS_MULT` still applies). Speed is unchanged, so the same attack now conquers about 3x as much land before it runs out. Raise the number if expansion now feels too strong (1.0 = old behaviour).
Gameplay changed: `NET_VER` and `PROTO_VERSION` are now 22 (re-deploy relay.py and re-export).

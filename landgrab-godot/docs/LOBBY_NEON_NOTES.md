# Lobby neon pass (Conqueria v0.0.2 -> next)
Done (main.gd only, nothing else touched): neon / hover / light-up animations (CqGlow, CqMapFx, CqPanelBg spark, _cq_hover, _cq_stagger, _cq_flicker),
big framed map preview with a smooth generated picture (_cq_make_preview), username field on the PLAY page (was in Settings; blank = JOIN / Private room refuse
with a red flash), 2:00 timer shown big with a bar (_cq_timer; LOBBY_SECS = 120, server LANDGRAB_LOBBY_SECS = 120), text boxes wrapped / trimmed so lines no longer overlap.
NOT run in Godot. If any text still overlaps, send a screenshot: widths are set from the window size in _build_lobby_ui (cw / prev_h).

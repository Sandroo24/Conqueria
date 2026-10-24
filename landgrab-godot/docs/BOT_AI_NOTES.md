# Bot (tribe) AI upgrade - static checks only, NOT run in Godot
Backup before this pass: `main.gd.bak_before_botai`. Files changed: `main.gd`, `server/relay.py` (only PROTO_VERSION 22 -> 23), this note.

## How the existing systems work (inspected first)
- **Territory**: `start_attack(o, t, frac, focus)`; target 0 = free land. Same troop cost / speed / rules for humans and bots. `focus` steers which tiles are taken.
- **Gold**: `_tick_body()` adds `GOLD_PER_OF_TICK * OF_TICKS_PER_ECON` = 100 x 5 per 0.5 s = **exactly 1000/s** to every living nation, bots included. It is the only place gold is added per tick (checked).
- **Spending**: the only three `gold -=` lines are building (`place_building`), sea units (`place_sea`) and bomb launches. They run only from validated commands.
- **Defeat reward (already existed)**: `eliminate()` gives the conqueror ALL of a defeated bot's gold (half of a human's). Nothing was added.
- **Multiplayer**: the game is LOCKSTEP. The relay only forwards commands; every client runs the identical deterministic simulation (`sim_rng`, tick counter). There is no server-side AI or server-side gold. So "only the authority awards gold" is satisfied the lockstep way: each client applies the same single income line to its own identical copy of the state - nothing is applied twice, no extra network message exists. Moving AI to a real server would be a different architecture, so I did not do that.

## What changed
- `bot_think()` rewritten; new `_bot_pick_target()`; personalities `P_BALANCED / P_AGGRESSIVE / P_EXPANDER / P_DEFENSIVE / P_OPPORTUNIST` (Player.persona, drawn once per match in `_plan_bot_starts` with `sim_rng`).
- Expansion only into FREE land via the ordinary `start_attack(p, 0, frac, focus)` (focus = the chosen tile, so the personality really decides the direction). No special privileges.
- Decision interval 1-2.5 s (Expander every 1-1.5 s, Defensive every 1-2.5 s, others every 1-2 s); each decision scans at most 160 border tiles (a different part of the border each time, `scan_off`) and scores at most 40 spots on a 5x5 window (free land / own land / rival land / rival weakness).
- Blocked memory: an expansion that gained no tiles marks its 8x8 area blocked for 30 s (`Player.blocked`), so bots turn elsewhere; blocked entries are purged.
- Scores: Expander = lots of free land; Defensive = compact growth, avoids rivals; Aggressive = free land touching rivals; Opportunist = free land beside the smallest rivals; Balanced = mix. Troop share per expansion also differs (12-40%).
- Safety net: `validate()` now rejects every bot command except `attack` ("Tribes only expand").
- Removed the DEAD bot building code (nothing called it): `bot_build`, `_bot_try`, `_bot_warship`, `_bot_spot`, `_bot_safety`, `_bot_upgrade_tile`, `_bot_naval`, `_bot_fleet`, `_bot_diplomacy`, `_count_ships`. No bot code path can build, buy, upgrade or fire anything.
- Gold comment corrected (it said "bots 50"; the code already gave bots the full 1000/s).
- NET_VER / PROTO_VERSION 22 -> 23 (bot decisions are part of the shared simulation): redeploy relay.py.

## Conflict with the request (explained, not silently changed)
Your rules say bots never attack ("expand only") but the personalities mention rivals / weak rivals. I kept the design from before: tribes never attack players or each other. Aggressive / Opportunist express "toward rivals" as expanding into free land NEXT TO rivals. If you want real bot-vs-bot or bot-vs-player attacks, say so: that is a combat-balance decision.

## Not done / limitations
- Not run in Godot: syntax checked statically only (brackets, duplicate functions, no references to removed functions).
- Tribes on islands stop when the island is full (no boats, by design).
- Personality is not shown in the UI.
- Balance numbers (weights, fractions, 30 s block) are first guesses.

## Test plan
1. F5, start a match: grey tribes should keep growing for the whole match, different tribes in different directions (watch an Expander vs a Defensive one).
2. Leave it running 10 min at 4x speed: no tribe sits idle while free land touches its border; FPS stays steady (Debug > F3 perf shows bot_think time).
3. Check a tribe's gold on hover: +1000/s, never drops. Conquer one: you receive all its gold ("Conquered ...  +N gold").
4. Two clients online: both see identical tribe growth (no desync message). Update relay.py to version 23 first.

## Troop losses (later change)
`ATK_LOSS_MULT` 0.65 -> 0.45: attackers lose about 30 % fewer troops per conquered tile in fights between nations (free-land expansion keeps `EXPAND_LOSS_MULT` 0.35). Applies to everyone, bots included. NET_VER / PROTO_VERSION 24: redeploy relay.py.

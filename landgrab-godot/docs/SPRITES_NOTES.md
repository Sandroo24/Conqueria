# Building pictures (port, village, city, duchy, SAM site, bomb launcher)

STATUS: assets re-cut and installed; code was already in your project and is unchanged. **Not opened in Godot** (no Godot here).

## Which picture is which
| file in `ui/buildings/` | used for | source image you sent |
|---|---|---|
| `port.png` | Port | the dock with the crane |
| `village.png` | Village | the thatched house on the black background (kept from your project: it was already clean) |
| `city.png` | City | the three timber houses |
| `duchy.png` | Duchy | the walled castle town |
| `sam.png` | SAM site | the camouflage launcher box |
| `silo.png` | Bomb launcher (Missile Silo) | the rocket launcher with orange-tipped missiles |
(Your message called both the 2nd and 3rd picture "city"; I used the 2nd for the village, the first building level. Swap the two files if you meant otherwise.)

## What I changed
- `port.png`, `city.png`, `duchy.png`, `sam.png`, `silo.png` re-cut from your original JPGs. All are real transparent PNGs (RGBA), 380 px wide, like the others.
  - The "transparent" checkerboard in your JPGs is painted-in pixels, not real transparency: removed by flood-filling the two checker tones from the image border, then removing the soft JPEG edge, enclosed gaps that follow the checker grid, and stray specks.
  - Pixels under the transparent area carry the sprite's own colours, so the game's smooth shrink (Lanczos in `_spr_tex`) leaves no grey halo.
  - Fixed in your previous versions: grey checker fragments around the duchy, a white patch on the SAM, floating gaps in the port, and the city was only 216 px wide (blurry when zoomed in).
- `main.gd` is the file you sent, unchanged (checked byte for byte). It already draws the pictures on the map (`draw_building` -> `_draw_sprite`), in the placement ghost and on the build bar (`BldIcon` -> `draw_building_fit`), on a flat disc in the owner's colour, and falls back to the drawn shapes if a file is missing.

## Notes / limits
- The pictures are scaled by width (`SPRITE_W` table in main.gd: port 1.9, village 1.35, city 1.7, duchy 2.1, sam 1.7, silo 1.8). The duchy and city are now a little taller than before (more of the picture is kept); change those two numbers if they feel too big.
- Station, warship, mine and the nuke icons are still the drawn shapes (you sent no pictures for them).
- Godot imports the new PNGs the first time you open the project (a few seconds). No `.import` files are included on purpose.
- `server/` from `landgrab-server-fix.zip` (protocol 20) matches this main.gd (`NET_VER` 20). It is not included here.

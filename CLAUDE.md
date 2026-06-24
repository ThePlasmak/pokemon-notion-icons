# Pokemon Icons for Notion

This repo hosts processed Pokémon box sprites that can be used as page icons in Notion databases.

## The Goal

All Pokémon pages in Notion should have **big, consistent, centered** page icons—and they should not be cut off by cropping.

### Current Processing Pipeline

All sprites: trim transparent padding → **variable (compressed) nearest-neighbor scale per group** → **centered by visual center-of-mass** on 280×280 transparent canvas → losslessly compressed with pingo (-s4).

**Variable scaling (current, gen7x/gen8/gen9):** A pure fixed per-group scale made tiny box sprites (Carbink, Goomy, Voltorb) far too small while big mons filled the whole tile — fills ranged ~0.25–1.00. We now compress that range with a gamma curve so small mons are boosted but stay proportionally smaller than big ones. For each sprite, recover the original-resolution sprite (downscale the existing icon by its known integer factor), then re-upscale so:

```
target_maxdim_fill = CEIL * (orig_maxdim / group_max_orig_maxdim) ** GAMMA
CEIL  = 0.80   # matches the ZA Mega ceiling (e.g. Mega Delphox)
GAMMA = 0.50   # lower = stronger boost to small mons; 1.0 = old linear behaviour
```

Result: every group now spans ~0.36–0.80 max-dim fill (was 0.25–1.00). Biggest mons land at the ZA ~0.80 ceiling; smallest are lifted off the floor; left-to-right size ordering is preserved. Reprocessing script: lossless original recovery via the documented integer factors below + the 5 capped outliers. NEAREST throughout to preserve pixel crispness. (`/tmp/process_icons.py` in the working session.)

- **Gen7x group** (1,206 sprites): recovered at 7x — 1 outlier recovered at 4x: Melmetal
- **Gen8 group** (1,352 sprites): recovered at 5x — 4 outliers recovered at 3x: Eternatus Eternamax, Eternatus, Kyogre, Rillaboom Gmax
- **Gen9 group** (165 sprites): recovered at 5x — no outliers
- **ZA group** (46 sprites): 7x fixed scale, **left unchanged** — already consistently sized (~0.57–0.80 fill) and used as the reference ceiling. Pokémon Legends: Z-A base-game and Mega Dimension mega evolutions from toxicoke's 32×32 box-icon sheets; one default Floette and one default Tatsugiri only.

Includes all forms (megas, regional variants, alternate forms), gender variants (`female/` subdir), and right-facing sprites (`right/` subdir, gen7x only). Subdirs share their group's reference size so a female/right sprite matches its regular counterpart.

**Center-of-mass centering:** Instead of centering the bounding box, we compute the centroid of all opaque pixels (visual center of gravity) and align that to the canvas center. This makes sprites look visually balanced — e.g., a Pokémon with a long tail won't appear off-center. The offset is clamped so no part of the sprite is ever cropped off the canvas edge.

### What Didn't Work

1. **Auto-crop + scale-to-fit 112px on 128×128** — gave each sprite a different pixel size (small Pokémon scaled more than large ones). Toxtricity looked wonky.
2. **Raw 68×56 originals, no processing** — gen8+ sprites looked fine but gen7x sprites were too small for Notion icons.
3. **Raw 68×56 centered on 128×128** — still too small.
4. **Fixed 4x scale WITHOUT trimming on 280×280** — too much transparent padding in the source sprites made everything tiny.
5. **Fixed 5x scale WITH trimming on 280×280 (all sprites same scale)** — still too small because the gen8+ sprites (max 47px trimmed) limited the scale factor for everyone.
6. **Per-group scaling (7x/5x) WITH trimming on 280×280, bounding-box centering** — sprites not visually centered (asymmetric features like tails/crests pulled the apparent center off).
7. **Per-group scaling (7x/5x) WITH trimming on 280×280, center-of-mass centering** — visually centered and consistent but only for 76 team sprites.
8. **Per-group scaling (5x/4x/5x) WITH trimming on 280×280, center-of-mass centering, ALL sprites** — too small. Scale factors lowered to accommodate largest sprites in each group.
9. **Per-group scaling (7x/5x/5x) with 5 outliers individually capped, ALL sprites** — kept original scale factors; only Melmetal (4x), Eternatus/Eternatus Eternamax/Kyogre/Rillaboom Gmax (3x) individually smaller. Problem: tiny box sprites (Carbink, Goomy, Voltorb) were far too small relative to big mons (fills ~0.25–1.00).
10. **Variable gamma-compressed scaling (current, gen7x/gen8/gen9)** — recover each original sprite, then re-upscale to `0.80 * (orig/group_max)^0.5`. Boosts small mons, caps big mons at the ZA ~0.80 ceiling, preserves size ordering. ZA group left unchanged as the reference.

### Cache Busting for Notion

Notion caches external icon URLs aggressively. To force refresh after pushing new sprites, reset all icons via the API with `?v=<timestamp>` appended to the URL. This makes Notion treat it as a new URL and re-fetch.

## Sprite Sources & Repo Structure

| Repo Folder      | Source Repo            | Source Directory         | Sprites | Scale | Art Style |
| ---------------- | ---------------------- | ------------------------ | ------- | ----- | --------- |
| `pokemon-gen7x/` | `msikma/pokesprite`    | `pokemon-gen7x/regular/` | 1,206   | 7x    | SM/USUM   |
| `pokemon-gen8/`  | `msikma/pokesprite`    | `pokemon-gen8/regular/`  | 1,352   | 5x    | SwSh      |
| `pokemon-gen9/`  | `bamq/pokemon-sprites` | `pokemon/regular/`       | 165     | 5x    | SV        |
| `pokemon-za/`    | `toxicoke`             | DeviantArt box-icon sheets | 46      | 7x    | Legends Z-A megas |

Gen7x/gen8/gen9 source sprites are 68×56. ZA source sprites are 32×32 cells extracted from toxicoke's published sheets. Filenames mirror the source repos where applicable and use slugged Mega names for ZA additions.

Subdirectories: `female/` (gender-variant sprites), `right/` (right-facing, gen7x only).

### Notion icon mapping (by game)

| Games (current numbering) | Titles                                                                                 | Icon Source                        |
| ------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------- |
| 1-10                      | Platinum, Ruby, Black, Sun, FireRed, HeartGold, White 2, Alpha Sapphire, Ultra Moon, Y | `pokemon-gen7x/`                   |
| 11-12                     | Shield, Legends Arceus                                                                 | `pokemon-gen8/` (Staraptor: gen7x) |
| 13                        | Scarlet                                                                                | `pokemon-gen9/`                    |

# Pokemon Icons for Notion

This repo hosts processed Pokémon box sprites that can be used as page icons in Notion databases.

## The Goal

All Pokémon pages in Notion should have **big, consistent, centered** page icons—and they should not be cut off by cropping.

### Current Processing Pipeline

All sprites: trim transparent padding → fixed nearest-neighbor scale per group → **centered by visual center-of-mass** on 280×280 transparent canvas → losslessly compressed with pingo (-s4).

- **Gen7x group** (1,206 sprites): 7x scale — 1 outlier capped: Melmetal at 4x
- **Gen8 group** (1,352 sprites): 5x scale — 4 outliers capped at 3x: Eternatus Eternamax, Eternatus, Kyogre, Rillaboom Gmax
- **Gen9 group** (165 sprites): 5x scale — no outliers
- **ZA group** (46 sprites): 7x scale — Pokémon Legends: Z-A base-game and Mega Dimension mega evolutions from toxicoke's 32×32 box-icon sheets; one default Floette and one default Tatsugiri only

Pixel size is consistent within each group, preserving natural size differences between Pokémon. 5 oversized sprites are individually capped one step below their max fit to avoid lowering the scale for everyone else. Includes all forms (megas, regional variants, alternate forms), gender variants (`female/` subdir), and right-facing sprites (`right/` subdir, gen7x only).

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
9. **Per-group scaling (7x/5x/5x) with 5 outliers individually capped, ALL sprites** — current approach. Keeps original scale factors, only Melmetal (4x), Eternatus/Eternatus Eternamax/Kyogre/Rillaboom Gmax (3x) are individually smaller.

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

# Gachaverse Calendar

A lightweight, dependency free calendar built with plain HTML, CSS, and JavaScript for tracking gacha game patch schedules, banner phases, livestream dates, and public holidays.

The calendar is driven by `patch-dates.json`, so most yearly maintenance can be done without changing the JavaScript.

## Features

### Calendar

- Generates all 12 months for the current year.
- Weeks start on Monday.
- Automatically marks today's date.
- On mobile, automatically scrolls to the current month after loading.
- Updates the document title, title meta tag, and footer with the current year.

### Next year preview

When any configured game or holiday contains a date beyond the current year, the calendar automatically adds a preview of the next year below the current calendar.

A full width year heading separates the two years.

The feature is controlled by:

```js
const NEXT_YEAR_PREVIEW = true;
const NEXT_YEAR_PREVIEW_MONTHS = 4;
```

Only dates explicitly stored in the JSON trigger the next year preview. Auto generated patches do not trigger it by themselves.

When the system year advances, the calendar renders the new current year normally. Patch auto generation can continue from the latest configured patch into the new year.

## Game toggles

A toggle is generated for every active game with version data.

- Toggle icons are loaded from `assets/{shorthand}-icon.webp`.
- Active and inactive states are stored in `localStorage` under `gachaverse_toggles`.
- Stored user preference takes priority over the JSON default.
- Disabled games are skipped completely.
- Turning a game off removes its patch, livestream, and phase highlights.
- Turning it back on restores the highlights without rebuilding the calendar.

A game can default to disabled with:

```json
"toggle": false
```

A game can be completely excluded with:

```json
"active": false
```

## Patch and banner phases

Each manually maintained version uses simple ISO dates:

```json
{
    "version": 4.7,
    "livestream": "2026-10-31",
    "phaseOne": "2026-11-11",
    "phaseTwo": "2026-12-02"
}
```

`phaseOne` is required. `livestream` and `phaseTwo` are optional.

### Phase rules

For a version with both phases:

- Patch starts on `phaseOne`.
- Phase 1 runs from `phaseOne` until the day before `phaseTwo`.
- Phase 2 starts on `phaseTwo`.
- Phase 2 ends the day before the next version's `phaseOne`.

For a manually configured version without `phaseTwo`:

- The entire patch is treated as Phase 1.
- It ends the day before the next version starts.

For the latest version, where there is no following version:

- The end date is calculated from the game's patch cycle.

Dates must use:

```text
YYYY-MM-DD
```

For example:

```text
2027-01-13
```

Invalid `phaseOne` values are skipped and logged to the browser console.

## Auto interval and estimated patches

The calendar can fill missing versions automatically.

The default patch cycle is 42 days:

```js
const DEFAULT_INTERVAL = 42;
```

A game can override this using `autoInterval`:

```json
{
    "game": "ExampleGame",
    "autoInterval": 41
}
```

`autoInterval` represents the number of highlighted days after the patch start, so an `autoInterval` of `41` produces a 42 day patch cycle.

### How auto fill works

- Starting from a configured patch, the expected next patch is placed one full cycle later.
- Missing versions between two configured versions can be inserted automatically.
- A gap is only filled when at least half a patch cycle remains before the next manually configured patch.
- The generated version number increases by `0.1`.
- After the latest configured version, patches continue to be generated until the end of the rendered calendar range.
- Auto generated patches always contain two phases.
- Auto generated Phase 2 starts around the halfway point of the cycle.

Manual JSON entries always override the need for an estimated entry at that point in the timeline.

## Hover phase visualization

Hovering over a patch start temporarily highlights the entire patch range.

- Phase 1 uses `highlight-{game}-patch-phase-one`.
- Phase 2 uses `highlight-{game}-patch-phase-two`.
- Phase 2 is displayed with reduced opacity by default.
- The range disappears when the pointer leaves the patch date.

A patch without `phaseTwo` stays entirely in Phase 1.

## Tooltips

Calendar dates can display contextual tooltips.

### Patch

Example:

```text
ABC Patch v4.7
```

### Livestream

Example:

```text
ABC Livestream v4.7
```

### Holiday

Example:

```text
New Year's Day
```

### Today

Hovering today's date also displays the closest upcoming patch among currently enabled games.

Example:

```text
ABC v4.8 in 12 days
```

If multiple games share the same nearest patch date, they are shown together.

Multiple events on the same day stack as separate tooltip lines.

Tooltip positioning is constrained to the viewport and automatically moves below the cursor when there is insufficient room above it.

## Multiple events on the same date

When several games have an event on the same calendar date, their colors are combined into a generated gradient.

Patch and livestream combinations also receive an adjusted border radius so the different event types remain visually distinguishable.

The calculated colors are cached after first use.

## Holidays

Holidays use the same ISO date format:

```json
{
    "game": "Holidays",
    "shorthand": "hol",
    "color": "#aa0000",
    "dates": [
        { "date": "2027-01-01", "name": "New Year's Day" }
    ]
}
```

Holiday entries are always visible and are not controlled by a game toggle.

## JSON configuration

### Game

```json
{
    "game": "ExampleGame",
    "shorthand": "abc",
    "color": "#33ccff",
    "autoInterval": 41,
    "toggle": true,
    "active": true,
    "versions": [
        {
            "version": 4.7,
            "livestream": "2026-10-31",
            "phaseOne": "2026-11-11",
            "phaseTwo": "2026-12-02"
        }
    ]
}
```

### Game properties

| Property | Required | Purpose |
|---|---|---|
| `game` | Yes | Full game name used for labels and accessibility. |
| `shorthand` | Yes | Short unique identifier used by CSS classes, toggles, icons, and event labels. |
| `color` | Yes | Highlight and toggle color. |
| `versions` | Yes | Patch/version definitions. |
| `autoInterval` | No | Custom patch interval. Value `41` means a 42 day cycle. |
| `toggle` | No | Initial toggle state. Defaults to enabled. |
| `active` | No | Set to `false` to exclude the game completely. |

### Version properties

| Property | Required | Purpose |
|---|---|---|
| `version` | Yes | Numeric patch version. |
| `phaseOne` | Yes | Patch start date in `YYYY-MM-DD` format. |
| `phaseTwo` | No | Phase 2 start. Missing means the manually configured patch remains Phase 1 throughout. |
| `livestream` | No | Livestream date in `YYYY-MM-DD` format. |

## Game icons

Each active game expects a WebP icon using its shorthand:

```text
assets/abc-icon.webp
```

The JavaScript preloads these icons and dynamically creates the matching toggle CSS.

For a game with:

```json
"shorthand": "abc"
```

add:

```text
assets/abc-icon.webp
```

## Dynamic game styling

Game colors do not need to be duplicated in the stylesheet.

The JavaScript generates styles for:

```text
toggle-{shorthand}
highlight-{shorthand}-patch
highlight-{shorthand}-livestream
highlight-{shorthand}-patch-phase-one
highlight-{shorthand}-patch-phase-two
```

This allows a new game to be added mainly through JSON plus its icon asset.

## Responsive behavior

The calendar uses CSS Grid.

| View | Calendar columns | Next year preview |
|---|---:|---|
| Desktop | 4 | Jan to Apr |
| Tablet, up to 1024px | 3 | Jan to Mar |
| Mobile, up to 480px | 2 | Jan to Feb |

The JavaScript always generates up to four preview months. CSS hides the excess preview months at smaller breakpoints, so resizing does not require rebuilding the calendar.

## Important constants

Most global behavior can be changed near the top of `main.js`:

```js
const CURRENT_YEAR = new Date().getFullYear();
const DEFAULT_INTERVAL = 42;
const DATA_URL = "patch-dates.json";
const SITE_TITLE_PREFIX = "Gachaverse";
const MOBILE_BREAKPOINT = 480;
const NEXT_YEAR_PREVIEW = true;
const NEXT_YEAR_PREVIEW_MONTHS = 4;
```

### `CURRENT_YEAR`

Normally uses the current system year. For a quick rollover test it can temporarily be changed to:

```js
const CURRENT_YEAR = 2027;
```

This tests year generation and auto interval behavior. The actual `today` marker still uses the browser's real current date.

### `DEFAULT_INTERVAL`

Fallback patch cycle when a game does not provide `autoInterval`.

### `NEXT_YEAR_PREVIEW`

Enables or disables the next year preview.

### `NEXT_YEAR_PREVIEW_MONTHS`

Maximum number of next year months generated before responsive CSS hides extras.

## File structure

```text
.
├── index.html
├── main.js
├── patch-dates.json
└── assets/
    ├── favicon.ico
    ├── favicon-256.ico
    └── abc-icon.webp
```

## Running locally

Because `main.js` loads `patch-dates.json` with `fetch()`, serve the project over HTTP instead of opening `index.html` directly with a `file://` URL.

For example, with Python installed:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

Any basic static file server will work.

## Adding a game

1. Add a game object to `patch-dates.json`.
2. Give it a unique lowercase `shorthand`.
3. Add at least one version with `version` and `phaseOne`.
4. Add `phaseTwo` when that patch has a known second phase.
5. Add `livestream` when applicable.
6. Add `assets/{shorthand}-icon.webp`.
7. Add `autoInterval` only when the game's cycle differs from the default behavior.

No additional JavaScript or game specific CSS should normally be required.

## Yearly maintenance

The project is intended to be manually refreshed each year.

Recommended yearly update:

1. Update confirmed versions and dates in `patch-dates.json`.
2. Add the new year's holidays.
3. Remove historical data you no longer need.
4. Keep at least the latest patch needed to seed auto interval generation.
5. Check games with unusual patch lengths and set `autoInterval` if needed.
6. Verify next year dates if you want the next year preview to appear near year end.

Auto generated versions are convenience estimates. Confirmed schedules should be entered manually when available.

## Notes

- No frameworks or third party JavaScript dependencies are required.
- The application is entirely client side.
- `patch-dates.json` is the primary maintenance file.
- Patch version auto generation increments versions by `0.1`.
- The current format intentionally does not use a `lastMinor` rollover setting. If a game's real version numbering jumps differently, update the confirmed JSON entries during normal maintenance.
- Patch and phase schedules can change. Confirmed game schedules should take priority over auto generated estimates.

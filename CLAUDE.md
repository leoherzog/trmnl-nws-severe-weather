# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TRMNL plugin for displaying National Weather Service (NWS) severe weather alerts. The plugin renders weather alerts in different layout formats using TRMNL Liquid templates.

## Template Structure

The project consists of `.liquid` files. All rendering logic lives in `shared.liquid`; the four size files are thin wrappers that `{% render 'main', ... %}` with a `layout_style` argument so every size shares one source of truth.

- **shared.liquid**: Defines two `{% template %}` partials:
  - `main` — categorizes alerts (active vs upcoming), renders the list, the `title_bar`, and a `<script>` that formats all timestamps client-side.
  - `alert_item` — renders a single alert card; called from both the active and upcoming loops with a `date_style` parameter (`'time'` for active, `'datetime'` for upcoming).
- **full.liquid**: `layout_style: 'full'` — two-column overflow list (active above upcoming).
- **half-horizontal.liquid**: `layout_style: 'half-horizontal'`.
- **half-vertical.liquid**: `layout_style: 'half-vertical'`.
- **quadrant.liquid**: `layout_style: 'quadrant'`.

## Key Data Processing

Templates process NWS alert data with these key operations:

1. **Timestamp Display**: Liquid emits epoch seconds into `data-dt` attributes (`data-dt-style` picks the format); a `<script>` in `main` formats them with `Intl.DateTimeFormat` at render time. TRMNL screenshots the markup in a headless browser, so the script runs server-side. Locale comes from `trmnl.user.locale` (field order and separators adapt per region), timezone from `trmnl.user.time_zone_iana` (DST-correct), and 12/24-hour is forced via `hour12` from the `timemode` custom field. Invalid locale/timezone values fall back to render-environment defaults.
2. **Alert Categorization**: Done in Liquid, comparing UTC epoch seconds (`trmnl.system.timestamp_utc` vs `onset`/`ends` piped through `date: "%s" | plus: 0` — the `plus: 0` coerces the string to a number):
   - Active: `onset ≤ now ≤ ends`
   - Upcoming: `onset > now` (only shown if `includefuture == 'true'`)
3. **Paragraph Normalization**: `alert_item` emits the raw NWS description text (plus `instruction` after a blank line, when present) inside the clamped `.description` span. A script IIFE in `main` splits `textContent` on blank lines, collapses NWS's ~70-char hard-wrap newlines into spaces, and rebuilds the block with `<br>` between paragraphs — so each `* WHAT/WHERE/WHEN` bullet starts on its own line.
4. **Dynamic Description Clamp**: descriptions carry a baseline `data-clamp="2"` (`3` on `lg:`). A second script IIFE (after normalization, before TRMNL's clamp engine initializes) computes the vertical budget from the enclosing `.layout` box (the framework gives `.layout` a fixed CSS height per view type; the *viewport* is always the full 800×480 / 1040×780 screen even for quadrant/half views, and `.columns`/`.column` heights stretch with content — neither may be used as the budget), costs out all items at their *baseline* clamps, then hands out extra description lines one at a time round-robin while free pixels remain, capped at each description's natural length. Showing every alert at baseline always wins over expanding one alert — expansion can never push an alert into the overflow engine's "and N more". If measurement fails, the baseline clamp applies. Logic is regression-tested by stub-DOM tests (see git history for `logic_test2.js` scenarios).

## Data Structure

The templates expect:
- `features`: Array of alert objects from the NWS API GeoJSON response.
- `alert.properties`: Contains `event`, `severity`, `description`, `instruction` (optional), `onset`, and `ends` (falls back to `expires`).
- `trmnl`: TRMNL system object with user settings and plugin configuration.
- `updated`: Last update timestamp (guarded — the "Updated" line is omitted when absent).

## Template Syntax Notes

Uses TRMNL Liquid:
- `{% template name %}` / `{% endtemplate %}` to define a partial, `{% render 'name', param: value %}` to call it (scope-isolated; pass everything the partial needs).
- `{% assign variable = value %}` for variable assignment.
- `{% for item in collection %}` for loops.
- `{{ variable | filter }}` for output with filters.
- Timestamp display is done in JavaScript via `Intl.DateTimeFormat`, not Liquid date filters — see Key Data Processing above.

## Development Guidelines

When modifying templates:
1. Maintain consistent time handling across all layouts.
2. Preserve the alert categorization logic (active vs upcoming).
3. Edit alert-card markup once in the `alert_item` partial — it is shared by both loops.
4. Ensure timezone conversions are applied consistently.
5. Test with both 12-hour and 24-hour time formats.
6. Keep the `title_bar` logo embedded as base64 (no network URLs — they can fail on-device).

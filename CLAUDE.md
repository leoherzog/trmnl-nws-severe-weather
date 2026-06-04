# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TRMNL plugin for displaying National Weather Service (NWS) severe weather alerts. The plugin renders weather alerts in different layout formats using TRMNL Liquid templates.

## Template Structure

The project consists of `.liquid` files. All rendering logic lives in `shared.liquid`; the four size files are thin wrappers that `{% render 'main', ... %}` with a `layout_style` argument so every size shares one source of truth.

- **shared.liquid**: Defines two `{% template %}` partials:
  - `main` — assigns time masks, categorizes alerts (active vs upcoming), renders the list, and the `title_bar`.
  - `alert_item` — renders a single alert card; called from both the active and upcoming loops with a `date_mask` parameter (`mask_time` for active, `mask_datetime` for upcoming).
- **full.liquid**: `layout_style: 'full'` — two-column overflow list (active above upcoming).
- **half-horizontal.liquid**: `layout_style: 'half-horizontal'`.
- **half-vertical.liquid**: `layout_style: 'half-vertical'`.
- **quadrant.liquid**: `layout_style: 'quadrant'`.

## Key Data Processing

Templates process NWS alert data with these key operations:

1. **Time Format Handling**: Supports both 12-hour and 24-hour time formats based on `trmnl.plugin_settings.custom_fields_values.timemode`.
2. **Alert Categorization**: Alerts are classified as:
   - Active: `onset ≤ now ≤ ends`
   - Upcoming: `onset > now` (only shown if `includefuture == 'true'`)
3. **Timezone Conversion**: All times are shifted to the user's local timezone using `trmnl.user.utc_offset`.

## Data Structure

The templates expect:
- `features`: Array of alert objects from the NWS API GeoJSON response.
- `alert.properties`: Contains `event`, `severity`, `description`, `onset`, and `ends` (falls back to `expires`).
- `trmnl`: TRMNL system object with user settings and plugin configuration.
- `updated`: Last update timestamp (guarded — the "Updated" line is omitted when absent).

## Template Syntax Notes

Uses TRMNL Liquid:
- `{% template name %}` / `{% endtemplate %}` to define a partial, `{% render 'name', param: value %}` to call it (scope-isolated; pass everything the partial needs).
- `{% assign variable = value %}` for variable assignment.
- `{% for item in collection %}` for loops.
- `{{ variable | filter }}` for output with filters.
- Custom filter: `l_date` for locale-aware date formatting.

## Development Guidelines

When modifying templates:
1. Maintain consistent time handling across all layouts.
2. Preserve the alert categorization logic (active vs upcoming).
3. Edit alert-card markup once in the `alert_item` partial — it is shared by both loops.
4. Ensure timezone conversions are applied consistently.
5. Test with both 12-hour and 24-hour time formats.
6. Keep the `title_bar` logo embedded as base64 (no network URLs — they can fail on-device).

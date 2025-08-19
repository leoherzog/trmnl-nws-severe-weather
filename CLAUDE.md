# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TRMNL plugin for displaying National Weather Service (NWS) severe weather alerts. The plugin renders weather alerts in different layout formats using Jinja2 templates with Liquid-style syntax.

## Template Structure

The project consists of Jinja2 templates that render NWS weather alerts:

- **full.html.j2**: Two-column layout showing active and upcoming alerts
- **half-horizontal.html.j2**: Horizontal split layout
- **half-vertical.html.j2**: Single column with active alerts stacked above upcoming
- **quadrant.html.j2**: Quarter-screen layout
- **shared.html.j2**: Shared template components (currently empty)

## Key Data Processing

Templates process NWS alert data with these key operations:

1. **Time Format Handling**: Supports both 12-hour and 24-hour time formats based on `trmnl.plugin_settings.custom_fields_values.timemode`
2. **Alert Categorization**: Alerts are classified as:
   - Active: `onset ≤ now ≤ ends`
   - Upcoming: `onset > now` (only shown if `includefuture == 'true'`)
3. **Timezone Conversion**: All times are converted to user's local timezone using `trmnl.user.utc_offset`

## Data Structure

The templates expect:
- `features`: Array of alert objects from NWS API
- `alert.properties`: Contains `event`, `severity`, `description`, `onset`, `ends`
- `trmnl`: TRMNL system object with user settings and plugin configuration
- `updated`: Last update timestamp

## Template Syntax Notes

Uses Liquid-style Jinja2 syntax:
- `{% assign variable = value %}` for variable assignment
- `{% for item in collection %}` for loops
- `{{ variable | filter }}` for output with filters
- Custom filters: `l_date`, `newline_to_br`

## Development Guidelines

When modifying templates:
1. Maintain consistent time handling across all layouts
2. Preserve the alert categorization logic (active vs upcoming)
3. Ensure timezone conversions are applied consistently
4. Test with both 12-hour and 24-hour time formats
5. Verify layout constraints (max-height: 340px for lists)
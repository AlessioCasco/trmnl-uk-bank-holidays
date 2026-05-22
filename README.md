# UK Bank Holidays — TRMNL Private Plugin

A [TRMNL](https://trmnl.com) Private Plugin that shows the next UK bank holiday and how many days remain, sourced live from the official [gov.uk bank holidays feed](https://www.gov.uk/bank-holidays.json). No API key, no server, no maintenance — gov.uk publishes the data, TRMNL polls it, the Liquid template renders it.

Supports England & Wales, Scotland, and Northern Ireland. All four TRMNL layouts (full, half horizontal, half vertical, quadrant) are included.

> **Status:** an approval request has been submitted to list this plugin on the official [TRMNL plugin directory](https://trmnl.com/plugins). Until then, install it manually via the Private Plugin flow below.

## Preview

| Full | Half horizontal |
| --- | --- |
| <img alt="Full layout" src="https://github.com/user-attachments/assets/c058c155-d15d-417b-a64e-1b6dc0617676" width="400"> | <img alt="Half horizontal layout" src="https://github.com/user-attachments/assets/94155389-dc05-48e8-9bfa-47a60ccdf2a8" width="400"> |

| Half vertical | Quadrant |
| --- | --- |
| <img alt="Half vertical layout" src="https://github.com/user-attachments/assets/79025bcc-541a-4e2f-aef7-27eee0a5435c" width="400"> | <img alt="Quadrant layout" src="https://github.com/user-attachments/assets/0c7908c8-f2d8-4ebc-b00a-cc776a2afe42" width="400"> |

## Setup

1. Open TRMNL → **Plugins** → **Private Plugin** → **Add new**.
2. **Strategy:** Polling.
3. **Polling verb:** GET. No headers, no body.
4. **Form Fields:** TRMNL's editor has a single YAML text area labelled "Form Fields" (the exported YAML key is `custom_fields`). Paste this:
   ```yaml
   - keyname: author_bio
     field_type: author_bio
     name: About This Plugin
     description: Live countdown to the next UK bank holiday, sourced from the official gov.uk feed. Pick your region (England & Wales, Scotland, or Northern Ireland) from the dropdown below.
     github_url: https://github.com/AlessioCasco/trmnl-uk-bank-holidays
     category: calendar
   - keyname: region
     field_type: select
     name: Region
     description: Which UK region's bank holidays to display.
     options:
       - "England & Wales": england-and-wales
       - "Scotland": scotland
       - "Northern Ireland": northern-ireland
     default: england-and-wales
   ```
   The `author_bio` block is what shows up on the public recipe page if you publish — TRMNL requires it before allowing publication. The `region` block is the actual user-facing dropdown.
5. **Polling URL:** `https://www.gov.uk/bank-holidays/{{ region }}.json` — TRMNL substitutes the selected dropdown value at fetch time.
6. **Refresh rate:** once a day is plenty — the data only changes once a year.
7. **Markup:** open the file in [`markup/`](./markup) matching the layout you want and paste it into the corresponding markup slot in TRMNL's editor:
   - [`markup/full.liquid`](./markup/full.liquid) — 800×480, headline + list of upcoming
   - [`markup/half_horizontal.liquid`](./markup/half_horizontal.liquid) — 800×240, headline + countdown
   - [`markup/half_vertical.liquid`](./markup/half_vertical.liquid) — 400×480, countdown + list
   - [`markup/quadrant.liquid`](./markup/quadrant.liquid) — 400×240, compact countdown
8. Save, pick your region from the dropdown, and add the plugin to a playlist.

The full plugin config is also in [`settings.yml`](./settings.yml) — useful as a reference, and importable directly via [`trmnlp push`](https://github.com/usetrmnl/trmnlp) if you prefer that workflow.

## What it shows

- **Next bank holiday** for your chosen region — name and date
- **Days remaining** until that holiday (says `today` on the day, `1 day` the day before)
- **Upcoming list** (full and half-vertical layouts only) — the next few holidays so you can plan ahead
- **Notes** from gov.uk (e.g. "Substitute day") when present

## Data source

The plugin polls the GOV.UK bank holidays JSON, which returns:

```json
{
  "division": "england-and-wales",
  "events": [
    { "title": "New Year's Day", "date": "2026-01-01", "notes": "", "bunting": true },
    ...
  ]
}
```

This is the canonical UK government feed used across the public sector and by [`govuk-bank-holidays`](https://github.com/ministryofjustice/govuk-bank-holidays). It's free, public, requires no authentication, and is updated whenever the government changes a holiday (e.g. one-off royal holidays).

## How the countdown works

The Liquid template:
1. Computes today's date at midnight (`'now' | date: '%Y-%m-%d'`).
2. Iterates the events array (already sorted oldest-first by gov.uk) and grabs the first event whose `date >= today`.
3. Subtracts unix timestamps and divides by 86400 to get whole days remaining.

ISO 8601 dates sort lexicographically the same as chronologically, so string comparison is correct.

## Contributing

Issues and PRs welcome. Common things you might want to tweak:

- **Change the upcoming list length** — bump the `found < N` condition in `full.liquid` / `half_vertical.liquid`.
- **Add the bunting flag** — gov.uk marks holidays with a `bunting: true` flag for celebratory days. You could render a small flag icon when `event.bunting` is true.
- **Locale / date format** — adjust the `date: '%A, %-d %B %Y'` strftime patterns.
- **A different region** — if a country publishes a similar JSON feed (e.g. nager.date), the same template shape would work with minor field renames.

## Licence

MIT — see [LICENSE](./LICENSE).

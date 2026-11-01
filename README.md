# Prayer Timetable for Home Assistant

[![HACS Custom](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)
[![Hassfest](https://github.com/haffi96/prayer-automate-ha/actions/workflows/hassfest.yml/badge.svg)](https://github.com/haffi96/prayer-automate-ha/actions/workflows/hassfest.yml)
[![HACS validation](https://github.com/haffi96/prayer-automate-ha/actions/workflows/hacs.yml/badge.svg)](https://github.com/haffi96/prayer-automate-ha/actions/workflows/hacs.yml)
[![Latest release](https://img.shields.io/github/v/release/haffi96/prayer-automate-ha)](https://github.com/haffi96/prayer-automate-ha/releases)

Prayer Timetable is a custom Home Assistant integration that extracts daily Fajr, Dhuhr, Asr, Maghrib, and Isha times from a PDF timetable. It creates timestamp sensors for each prayer, identifies the next prayer, and can export parsed rows as CSV.

> [!NOTE]
> This is a community custom integration. It is not an official Home Assistant integration.

## Installation

### HACS (recommended)

[![Open this repository in HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=haffi96&repository=prayer-automate-ha&category=integration)

1. Open **HACS** in Home Assistant.
2. Open the three-dot menu, select **Custom repositories**, and add `https://github.com/haffi96/prayer-automate-ha` as an **Integration**.
3. Search for **Prayer Timetable** and select **Download**.
4. Restart Home Assistant.

### Manual installation

1. Download this repository.
2. Copy `custom_components/prayer_timetable` to your Home Assistant configuration directory under `custom_components/prayer_timetable`.
3. Restart Home Assistant.

## Setup

1. Open **Settings → Devices & services → Add integration**.
2. Search for **Prayer Timetable**.
3. Choose one PDF source:
   - **Upload PDF** uploads a timetable from your browser and stores it at `/config/prayer_timetable/`.
   - **PDF path** is for a timetable already accessible inside the Home Assistant runtime, for example `/config/prayer_timetable/oxford.pdf`.
4. Optionally set **Year** and **Month** when rows contain only a day number, for example `2026` and `March`.
5. Optionally set **Date format** for unusual date strings, using Python `strptime` syntax such as `%d %B %Y`.

Paths from the machine where you run a browser or development shell are not automatically available in Home Assistant containers. Use **Upload PDF** or a path mounted into Home Assistant, normally under `/config`.

## Entities

The integration creates these sensors:

- Fajr, Dhuhr, Asr, Maghrib, and Isha — timestamp sensors
- Fajr time, Dhuhr time, Asr time, Maghrib time, and Isha time — plain text
  time sensors for dashboards and widgets
- Next prayer — the name of the next prayer today
- Next prayer time — timestamp sensor for the next prayer today
- Next prayer time display — plain text time sensor for dashboards and widgets
- Rows parsed — diagnostic count of parsed timetable rows

## Dashboard card

Add the built-in card from
[`example-dashboard/prayer-times-card.yaml`](example-dashboard/prayer-times-card.yaml)
to a Home Assistant dashboard:

1. Open the dashboard.
2. Select **Edit dashboard**.
3. Select **Add card**.
4. Select **Manual**.
5. Paste the YAML and save.

The example assumes the default entity ids:

```yaml
sensor.prayer_timetable_fajr
sensor.prayer_timetable_dhuhr
sensor.prayer_timetable_asr
sensor.prayer_timetable_maghrib
sensor.prayer_timetable_isha
sensor.prayer_timetable_fajr_time
sensor.prayer_timetable_dhuhr_time
sensor.prayer_timetable_asr_time
sensor.prayer_timetable_maghrib_time
sensor.prayer_timetable_isha_time
sensor.prayer_timetable_next_prayer
sensor.prayer_timetable_next_prayer_time
sensor.prayer_timetable_next_prayer_time_display
```

If your integration instance has another name, replace those ids with the
entities shown in **Settings → Devices & services → Prayer Timetable → Entities**.

There is also an optional richer example at
[`example-dashboard/prayer-times-button-card.yaml`](example-dashboard/prayer-times-button-card.yaml).
It requires the `button-card` custom Lovelace card from HACS.

For a phone or wall tablet, try
[`example-dashboard/prayer-times-panel.yaml`](example-dashboard/prayer-times-panel.yaml).
It shows a large clock above today's prayer times and scales up on screens
768px and wider. It requires the `card-mod` custom card from HACS and
`sensor.time` from the Time & Date integration. Put it in a view set to the
**Panel** type so it fills the screen.

## Android widget

Use the Home Assistant Companion App **Template** widget with the plain text
time sensors:

```jinja
Next: {{ states('sensor.prayer_timetable_next_prayer') | title }}
{{ states('sensor.prayer_timetable_next_prayer_time_display') }}

Fajr: {{ states('sensor.prayer_timetable_fajr_time') }}
Dhuhr: {{ states('sensor.prayer_timetable_dhuhr_time') }}
Asr: {{ states('sensor.prayer_timetable_asr_time') }}
Maghrib: {{ states('sensor.prayer_timetable_maghrib_time') }}
Isha: {{ states('sensor.prayer_timetable_isha_time') }}
```

These display sensors are intended for widgets because Android widget templates
cannot format a timestamp sensor when Home Assistant reports its state as
`unknown`.

## Export parsed rows

Use the `prayer_timetable.export_csv` action to write the currently configured timetable to a CSV file.

```yaml
action: prayer_timetable.export_csv
data:
  entry_id: YOUR_CONFIG_ENTRY_ID
  output_path: /config/prayer_timetable/prayer-times.csv
```

`entry_id` is the integration configuration entry ID. `output_path` must be writable by Home Assistant.

The exported CSV contains:

```csv
date,fajr,dhuhr,asr,maghrib,isha
```

## Troubleshooting

### The PDF file could not be found

Use **Upload PDF**, or provide a path visible inside the Home Assistant runtime. `/home/...` paths on a host are usually not valid in a Home Assistant container; use `/config/...` or configure a volume mount.

### Could not parse prayer rows

PDF timetables differ substantially. The parser first tries tables, then text lines. It works best when each row includes a date or day number followed by five prayer times. Supply Year and Month when the timetable contains only day numbers.

### Integration does not appear after installation

Restart Home Assistant, then refresh the browser. HACS installs integrations under `custom_components`, but Home Assistant must restart to load them.

### Enable debug logging

Add the following to `configuration.yaml`, restart Home Assistant, then inspect the logs:

```yaml
logger:
  logs:
    custom_components.prayer_timetable: debug
```

## Standalone CLI

This repository also includes a command-line extractor for local use:

```bash
uv run prayer-extract timetable.pdf prayer-times.csv --year 2026 --month March
```

## Support and contributing

Report problems and request features in the [issue tracker](https://github.com/haffi96/prayer-automate-ha/issues). Do not attach timetable PDFs containing personal or copyrighted material unless you have permission to share them.

This project is released under the [MIT License](LICENSE).

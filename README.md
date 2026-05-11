# LuxTransit Live for Home Assistant

A Home Assistant YAML package for showing real-time public transport departures in Luxembourg (CFL) using the mobiliteit.lu / ATP HAFAS OpenData API.

The package provides:

- Dynamic start and destination stop search
- Stop ID extraction from the mobiliteit.lu API response
- Real-time departure board data for the selected route
- Line, direction, platform/track, planned time, real-time departure and delay
- A configurable refresh interval
- A manual stop-list reload button
- Ready-to-use Home Assistant dashboard cards

Example route:

```text
Bettembourg, Gare → Luxembourg, Gare Centrale
Next departure: 07:42
Line: RE
Track: 2
Delay: 3 min
```

---

## How it works

The official mobiliteit.lu / ATP OpenData API exposes real-time public transport departures.

This package uses two API endpoints:

1. `location.nearbystops`  
   Loads public transport stops in Luxembourg and extracts the stop name and stop ID.

2. `departureBoard`  
   Loads real-time departures for the selected start stop and filters by destination direction.

The stop dropdowns store values internally like this:

```text
Bettembourg, Gare | 220102018
```

The text before `|` is the visible stop name.  
The text after `|` is the stop ID used by the API.

---

## Requirements

You need:

- Home Assistant
- Access to your YAML configuration
- A mobiliteit.lu / ATP API key
- Home Assistant packages enabled

---

## Requesting an API key

Request a personal API key by email:

```text
opendata-api@atp.etat.lu
```

Example email:

```text
Hello,

I would like to use the public mobiliteit.lu OpenData API in my private Home Assistant installation.

My goal is to display real-time train and bus departures for selected stops in Luxembourg.

I would like to use the HAFAS/OpenData API endpoints for nearby stops and departure boards.

Could you please provide me with a personal API access key?

Planned usage:
- private and non-commercial
- Home Assistant integration
- real-time departures for selected public transport stops
- moderate polling frequency, for example every 60 seconds

Thank you in advance.

Kind regards,
Your Name
```

Official dataset page:  
https://data.public.lu/en/datasets/api-mobiliteit-lu/

---

## Files in this repository

Recommended structure:

```text
.
├── README.md
├── cfl_mobiliteit.yaml
└── dashboard/
    ├── selection-card.yaml
    └── live-departures-card.yaml
```

---

## Installation

### 1. Enable packages in Home Assistant

Open:

```text
/config/configuration.yaml
```

Add this block if you do not already have a `homeassistant:` section:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

If you already have a `homeassistant:` block, only add the `packages` line inside that existing block.

Create the packages directory if it does not exist:

```text
/config/packages/
```

---

### 2. Add your API key to `secrets.yaml`

Open:

```text
/config/secrets.yaml
```

Add:

```yaml
mobiliteit_api_key: "YOUR_API_KEY_HERE"
```

---

### 3. Install the package YAML

Copy `cfl_mobiliteit.yaml` to:

```text
/config/packages/cfl_mobiliteit.yaml
```

---

### 4. Check the configuration

In Home Assistant, go to:

```text
Settings → System → Repairs → Check configuration
```

If the check passes, restart Home Assistant.

---

## Dashboard setup

Add two cards to your Home Assistant dashboard.

### Card 1: Route selection

Use the content of:

```text
dashboard/selection-card.yaml
```

This card lets you:

- reload the stop list
- search for a start stop
- select a start stop
- search for a destination stop
- select a destination stop
- configure the refresh interval
- enable or disable live updates

### Card 2: Live departures

Use the content of:

```text
dashboard/live-departures-card.yaml
```

This card shows:

- selected route
- next departure
- delay status
- line
- direction
- platform/track
- last data update
- upcoming departures

---

## First run

After restarting Home Assistant:

1. Open the dashboard.
2. Press **Haltestellen neu laden**.
3. Wait a few seconds.
4. Enter at least two characters into **Start suchen**, for example:

```text
Bettembourg
```

5. Select the desired start stop from the dropdown.
6. Enter at least two characters into **Ziel suchen**, for example:

```text
Luxembourg
```

7. Select the desired destination stop.
8. The departure card should update automatically.

---

## Refresh behavior

The package updates departure data when:

- the selected start stop changes
- the selected destination stop changes
- the configured refresh interval has passed
- `CFL schnelle Aktualisierung aktiv` is enabled

Default refresh interval:

```text
60 seconds
```

Minimum refresh interval:

```text
30 seconds
```

The stop list is reloaded once per day at:

```text
04:30
```

The stop list can also be reloaded manually with the dashboard button.

---

## Main Home Assistant entities

| Entity | Purpose |
|---|---|
| `input_text.cfl_start_search` | Search field for the start stop |
| `input_select.cfl_start_stop` | Start stop dropdown |
| `input_text.cfl_destination_search` | Search field for the destination stop |
| `input_select.cfl_destination_stop` | Destination stop dropdown |
| `input_button.cfl_haltestellen_laden` | Manual stop-list reload button |
| `input_number.cfl_refresh_seconds` | Departure refresh interval |
| `input_boolean.cfl_fast_refresh_enabled` | Enables/disables automatic departure refresh |
| `sensor.cfl_alle_haltestellen_rohdaten` | Raw stop-list sensor |
| `sensor.cfl_abfahrten_rohdaten` | Raw departure-board sensor |
| `sensor.cfl_route` | Current route |
| `sensor.cfl_nachste_abfahrt` | Next departure summary |

---

## Troubleshooting

### The dropdown still says "Bitte zuerst Haltestellen laden"

Press the **Haltestellen neu laden** button in the dashboard.

Then check:

```text
Developer Tools → States
```

Look for:

```text
sensor.cfl_alle_haltestellen_rohdaten
```

The state should be greater than `0`.

---

### The dropdown is empty

Enter at least two characters into the search field.

Examples:

```text
Be
Lux
Bettembourg
Luxembourg
```

---

### The dropdown shows old or strange values

Reload the stop list and change the search text once.

Example:

```text
Bettembourg
```

Change it briefly to:

```text
Bette
```

Then back to:

```text
Bettembourg
```

---

### No departures are found

Possible reasons:

- start and destination are the same stop
- the API does not recognize the selected destination as a direct direction filter
- the selected stop ID is not the stop you expected
- there are no departures at the moment

As a test, remove this line from the `departureBoard` REST block:

```yaml
direction: "{ states('input_select.cfl_destination_stop').split('|')[-1] | trim if '|' in states('input_select.cfl_destination_stop') else 'unknown' }"
```

This will show all departures from the selected start stop.

---

### Log error: `Template output exceeded maximum size`

This usually means an older version of the dropdown automation is still active.

Go to:

```text
Settings → Automations & Scenes
```

Search for:

```text
CFL Start Dropdown aktualisieren
CFL Ziel Dropdown aktualisieren
```

There should only be one active automation for each.

---

## Notes

The stop list is large. The raw stop-list sensor is excluded from the Home Assistant recorder:

```yaml
recorder:
  exclude:
    entities:
      - sensor.cfl_alle_haltestellen_rohdaten
```

This prevents unnecessarily large database entries.

---

## API and data attribution

This project uses public transport data from mobiliteit.lu / Administration des transports publics / CFL OpenData.

Please follow the official API usage requirements and any conditions provided with your personal API key.

Official dataset page:  
https://data.public.lu/en/datasets/api-mobiliteit-lu/

---

## License

This repository contains Home Assistant YAML configuration examples.

This project is licensed under the MIT License.

This license only applies to the configuration files and documentation in this repository. Transit data is provided by mobiliteit.lu / ATP / CFL and is subject to their own terms and API access rules.

The public transport data itself is provided by mobiliteit.lu / ATP and is not part of this repository.

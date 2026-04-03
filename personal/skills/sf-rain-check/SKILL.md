---
name: sf-rain-check
description: Check whether it is currently raining in San Francisco today. Trigger this skill when the user asks about rain, weather, or precipitation in SF.
---

# SF Rain Check

When the user asks if it's raining in San Francisco today, use the Open-Meteo API (no API key required) to fetch current weather conditions.

## Steps

1. Make a GET request to the Open-Meteo API for San Francisco's current weather:

```
https://api.open-meteo.com/v1/forecast?latitude=37.7749&longitude=-122.4194&current=precipitation,rain,weathercode&timezone=America%2FLos_Angeles
```

2. Parse the response:
   - `current.rain` — rain in mm in the last hour (> 0 means it's raining)
   - `current.precipitation` — total precipitation in mm (includes rain, snow, etc.)
   - `current.weathercode` — WMO weather code (51–67 and 80–82 indicate rain)

3. Report back to the user clearly: whether it is raining, and the current precipitation amount if relevant.

## Example (bash)

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=37.7749&longitude=-122.4194&current=precipitation,rain,weathercode&timezone=America%2FLos_Angeles"
```

## Notes
- Open-Meteo is free and requires no authentication.
- Always use the sandbox (bash_based_code_interpreter) to make the API call — do not guess or hallucinate weather data.
- Report the result in plain language (e.g., "Yes, it is currently raining in SF — 2.3mm of rain in the last hour." or "No, it is not raining in SF right now.").

# Rain Forecast Skill

Use this skill when the user asks about today's rain forecast, whether it will rain, or weather conditions.

## How to Check the Rain Forecast

1. **Get the user's location** — If the user has not provided a location, ask them for their city or region before proceeding.

2. **Geocode the city** — Convert a city name to lat/lon using the Open-Meteo geocoding API (free, no API key required):

```bash
python3 << 'EOF'
import urllib.request, json, urllib.parse

city = "London"  # replace with user's city
url = f"https://geocoding-api.open-meteo.com/v1/search?name={urllib.parse.quote(city)}&count=1"

with urllib.request.urlopen(url) as resp:
    data = json.load(resp)

result = data["results"][0]
print(f"City: {result['name']}, {result.get('country', '')}")
print(f"Latitude: {result['latitude']}")
print(f"Longitude: {result['longitude']}")
print(f"Timezone: {result['timezone']}")
EOF
```

3. **Fetch the forecast** — Use the coordinates from step 2:

```bash
python3 << 'EOF'
import urllib.request, json

LAT = 40.7128   # replace with geocoded latitude
LON = -74.0060  # replace with geocoded longitude
TZ  = "America/New_York"  # replace with geocoded timezone

url = (
    f"https://api.open-meteo.com/v1/forecast"
    f"?latitude={LAT}&longitude={LON}"
    f"&daily=precipitation_sum,precipitation_probability_max,weathercode"
    f"&timezone={TZ}"
    f"&forecast_days=1"
)

with urllib.request.urlopen(url) as resp:
    data = json.load(resp)

daily = data["daily"]
date        = daily["time"][0]
precip_sum  = daily["precipitation_sum"][0]
precip_prob = daily["precipitation_probability_max"][0]
wcode       = daily["weathercode"][0]

def describe_wcode(code):
    if code == 0:   return "Clear sky"
    if code <= 3:   return "Partly cloudy"
    if code <= 49:  return "Fog / drizzle"
    if code <= 67:  return "Rain"
    if code <= 77:  return "Snow"
    if code <= 82:  return "Rain showers"
    if code <= 99:  return "Thunderstorm"
    return "Unknown"

print(f"Date: {date}")
print(f"Condition: {describe_wcode(wcode)}")
print(f"Max rain probability: {precip_prob}%")
print(f"Total precipitation: {precip_sum} mm")
EOF
```

4. **Summarize the result** — Report back to the user in plain language:
   - Whether rain is expected today
   - The maximum probability of rain (%)
   - Expected total precipitation (mm)
   - Overall weather condition

## Example Response Format

> **Rain forecast for [City] — [Date]**
> - Condition: Rain showers
> - Chance of rain: 75%
> - Expected precipitation: 4.2 mm
>
> Bring an umbrella today.

## Notes
- All data comes from [Open-Meteo](https://open-meteo.com/) — free, no API key needed.
- Precipitation probability > 50% generally means rain is likely.
- If the user does not specify a location, ask before fetching.

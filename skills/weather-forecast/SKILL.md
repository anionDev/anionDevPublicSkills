---
name: weather-forecast
description: "Fetches weather data and presents it to the user as a short, human-readable weather report."
purpose: "Test and demonstrate the AI's ability to fetch and process data."
version: 1.0.0
---

# weather-report

## When to use

- The user asks about the weather / forecast (e.g. "what's the weather like tomorrow?", "will it rain this weekend in Hamburg?", "how warm will it get on Monday?").
- The user wants a short, readable summary rather than raw weather data.

## Not for

- Historical weather / climate statistics.
- Severe-weather alerts/warnings (the forecast endpoint used here does not provide them).
- Requests further out than Open-Meteo's forecast horizon (~16 days) — treat these as out of scope rather than guessing.

## Instructions

1. **Determine the location.**
   - The location is **optional** for the user to specify. If the user names a place or gives coordinates, use that.
   - If the user gives coordinates directly, use them as-is (skip geocoding).
   - If the user names a place, resolve it to coordinates via the Open-Meteo Geocoding API (no API key required):
     `GET https://geocoding-api.open-meteo.com/v1/search?name=<PLACE>&count=1&language=en&format=json`
     Use the first result's `latitude`/`longitude`, and note the resolved `name`/`country` so it can be confirmed to the user.
   - If no location is given, try to make a reasonable **estimate** from available context clues first (e.g. timezone offsets visible in the environment/terminal output, language of the conversation/workspace, company or workspace affiliation, a place mentioned earlier in the conversation). Only use this if it leads to a genuinely plausible single location — do not force a guess.
   - If no location is given and no such estimate is possible, **stop and ask the user** for the location before fetching any data — do not fabricate or silently default to an arbitrary place.

2. **Determine the time range.**
   - Default (if the user does not specify a time range): **tomorrow** (the day after the current date), in the location's local time.
   - Otherwise honor what the user asked for, for example "tomorrow", "today", "this weekend", "next N days", a specific weekday/date, or an explicit range.
   - Map the requested range to `forecast_days` and/or `start_date`/`end_date` on the daily API.

3. **Determine which data to fetch.**
   - Default (if the user does not specify which data they care about): **temperature**, **precipitation amount** (rain/snow), and the **general condition** (sunny / overcast / variable).
   - If the user asks for other data (wind, humidity, UV-index, sunrise/sunset, etc.), fetch those too via the matching Open-Meteo `daily`/`hourly` parameters instead of, or in addition to, the defaults.

4. **Call the Open-Meteo forecast API** (no API key required):
   `GET https://api.open-meteo.com/v1/forecast?latitude=<LAT>&longitude=<LON>&daily=<PARAMS>&timezone=auto`
   Always include `timezone=auto` so the returned dates match the location's local time.

   Relevant `daily` parameters for the default report:
   - `temperature_2m_max`, `temperature_2m_min` — temperature range
   - `precipitation_sum` — total rain + snow (mm)
   - `rain_sum`, `snowfall_sum` — rain (mm) / snow (cm) split out, if the user wants them distinguished
   - `weathercode` — used to derive the sunny/overcast/variable condition (see step 5)

   Example call (via terminal):
   ```bash
   curl -s "https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&daily=temperature_2m_max,temperature_2m_min,precipitation_sum,rain_sum,snowfall_sum,weathercode&timezone=auto&forecast_days=2"
   ```

5. **Derive the sunny / overcast / variable condition** from the WMO `weathercode`:
   - `0` → sunny / clear
   - `1`, `2` → mostly sunny / partly cloudy (variable)
   - `3` → overcast
   - `45`, `48` → overcast (fog)
   - `51`–`67`, `80`–`82` → variable, rain likely
   - `71`–`77`, `85`, `86` → variable, snow likely
   - `95`–`99` → variable, thunderstorms likely
   - When a code doesn't clearly fit, default to "variable" and let the precipitation numbers speak for themselves.

6. **Present the result as a short weather report — not raw JSON.**
   - **Always** state clearly for which **location** and which **date/time range** the forecast applies — this is mandatory in every response, e.g. "Tomorrow (2026-07-24) in Berlin, Germany: ...". If the location was an estimate (see step 1) rather than something the user specified, say so explicitly (e.g. "assuming you mean Wolfsburg, Germany — let me know if that's wrong").
   - Include only the data the user asked for (or the defaults).
   - Keep it to 1–3 sentences per day, unless the user explicitly asked for a multi-day breakdown (then give one short line per day).

## Examples

- User: *"What's the weather tomorrow?"* → resolve location (ask if unknown) → fetch tomorrow's `temperature_2m_max/min`, `precipitation_sum`, `weathercode` → reply e.g.: "Tomorrow in Berlin: sunny, 18–27°C, no rain expected."
- User: *"Will it rain this weekend in Hamburg?"* → resolve "Hamburg" → fetch Saturday + Sunday → reply with a precipitation-focused summary per day.
- User: *"What's the wind and humidity like next Monday in Munich?"* → resolve location → fetch `windspeed_10m_max` and a representative `relative_humidity_2m` value for next Monday → reply with just those (defaults are overridden by the explicit request).

## Guardrails

- Never fabricate weather data — always fetch it from Open-Meteo before answering.
- Never fabricate or silently assume a location either. Only use an unstated location if it can be reasonably estimated from real context clues (see step 1); otherwise ask the user first.
- If the location can not be resolved (geocoding returns no results), say so and ask the user for a more specific place name or coordinates.
- For requests beyond Open-Meteo's reliable forecast horizon (~16 days) or for historical data, tell the user this is out of scope rather than presenting speculative numbers as fact.
- Keep the report short and conversational; do not dump raw API JSON at the user.

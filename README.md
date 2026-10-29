[README.md](https://github.com/user-attachments/files/34155068/README.md)
# Weather Consensus

A single-page weather web app. Instead of trusting one forecast, it pulls the same forecast from five independent weather models, compares them, and shows you what the majority agree on, plus how much they disagree.

Everything lives in one file, `index.html`. There is no build step and no server. Host it on GitHub Pages (or open it locally) and it works.

Last updated: 29 Oct 2026 (v2)

---

## What it shows

1. **Headline and next 12 hours.** A one-line summary ("Rain likely 14:00–17:00", "Dry for the next 12 hours"), the current temperature, and an hourly strip. The bar under each hour is the share of sources forecasting rain that hour. If a minority of sources disagree, a note says so ("1 of 5 sources hint at showers around 15:00").
2. **What to wear.** Suggested top for today and tomorrow: T-shirt, jumper, waterproof, umbrella.
3. **Best times.** Pick an activity (cycling, running, walking, washing out) and see good time windows today and tomorrow.
4. **7-day consensus.** One row per day with the rain verdict, high/low, and a traffic light showing how much the sources agree. Tap a day to see every source side by side.
5. **Accuracy tracker.** How each source has performed for your location over time.
6. **Sources.** Switch sources on or off (minimum 3) and turn on accuracy weighting.

---

## Data sources

All data comes from Open-Meteo (https://open-meteo.com), a free weather API that serves several national models through one endpoint. The five sources are:

| Label | Open-Meteo model id |
|---|---|
| Met Office | `ukmo_seamless` |
| ECMWF | `ecmwf_ifs025` |
| NOAA GFS | `gfs_seamless` |
| DWD ICON | `icon_seamless` |
| Météo-France | `meteofrance_seamless` |

These are genuinely different models, so agreement between them carries real information (many weather apps just repackage one model).

One request per location fetches all five models: hourly temperature, rain, wind speed and gusts, plus daily high, low, rain total and max wind, for 7 days. Units are °C, mm and mph. Place search uses Open-Meteo's geocoding API.

---

## How the consensus works

### Rain
- A **day** counts as rainy for a source if it forecasts **1.0 mm or more**. An **hour** counts as rainy at **0.2 mm or more**. (Constants `WET_DAY` and `WET_HOUR`.)
- Each source gets one vote. The majority wins:
  - Hourly: a tie counts as dry.
  - Daily: a tie is shown as "Split: rain or dry".
- The rain amount shown is the **average of the agreeing sources only**, so one outlier doesn't distort it.
- The sources that disagree with the majority are flagged with ⚠ in the day detail.

### Temperature
- The **median** of the sources, which is robust to one source being way off.
- A source whose daily high is more than 2.5° from the median is flagged as a temperature outlier.

### Sources that don't reach far enough
Some models don't forecast as far ahead (Météo-France stops at roughly 4 days). Missing values are ignored and the vote uses whoever is left. Under 3 sources, the day gets an "only N sources" warning. Under 2, it shows "Not enough sources".

### Traffic light (agreement)
Three lamps (red, amber, green) with the lit one showing the level, plus text such as "3/5 agree · 5° spread".

- **Green:** every source agrees on rain and the daily highs are within 3°.
- **Amber:** at least one source disagrees on rain, or highs are 3–6° apart.
- **Red:** the rain call is split, or highs are more than 6° apart.

Days 5–7 also get a "long range" tag, since disagreement there is expected.

---

## What to wear

Tops only, based on consensus values between 07:00 and 21:00 (for today, from now onwards).

A rough "feels like" is the average temperature minus a small allowance for wind above 8 mph.

| Situation | Suggestion |
|---|---|
| Feels 16°+ and dry | T-shirt |
| Feels 16°+, light rain, gusts under 30 mph | T-shirt + umbrella |
| Feels 16°+, heavy rain (2.5 mm/h or more) or gusts 30 mph+ | T-shirt + waterproof |
| Feels under 16° and dry | Jumper |
| Feels under 16° and rain | Jumper + waterproof |
| Dry but gusts 30 mph+ and not warm | Jumper + waterproof (as a windbreaker) |
| Feels under 5° | Always includes a waterproof on top |
| Consensus dry, but some source shows rain | Adds a dashed "(pack it)" waterproof or umbrella |

---

## Best times (activities)

Looks at daytime hours only (07:00–21:00) and lists the windows where the consensus passes the activity's rules:

| Activity | Rules |
|---|---|
| Cycling | no rain, wind 15 mph or less, gusts 28 or less, 3°C or warmer |
| Running | no rain, 2–20°C, gusts 35 or less |
| Walking | no rain, gusts 40 or less, 0°C or warmer |
| Washing out | no source shows rain, wind 3 mph or more, 5°C or warmer, window of at least 3 hours |

---

## Accuracy tracking and weighting

1. The first time you open the app on a given day, it records what each source predicts for **tomorrow** (high temperature and rainy/dry), per location.
2. On a later visit, once that day has passed, it fetches what actually happened and scores each source: average error on the daily high, and how often its rain call was right.
3. After **5 or more scored days** for a source, its track record can be used as a weight. Weight = (0.3 + rain hit rate) / (0.5 + average high error). Sources without enough history get the average weight.
4. Weighting is **off by default**. Turn it on under "Sources" once it unlocks. When on, rain votes and temperatures are weighted.

Caveats:
- "What actually happened" is Open-Meteo's best-estimate analysis, not weather-station readings.
- It only looks back 14 days, so open the app at least every week or two or days are missed.
- History is per location and per browser/device. Records older than 45 days are dropped.

---

## Locations, offline use and storage

- Search for a place, or use the 📍 button. Searched places are saved as chips; tap × to remove. Default location is London.
- The last successful forecast per location is cached, so if you're offline it shows the saved forecast with an "Offline" banner.
- Everything is stored in the browser's `localStorage` (nothing leaves your device except the Open-Meteo requests). Keys:
  - `wc_locs`: saved locations
  - `wc_cur`: current location
  - `wc_settings`: selected sources, weighting on/off, chosen activity
  - `wc_log`: logged forecasts waiting to be scored
  - `wc_acc`: accuracy scores
  - `wc_cache_<lat,lon>`: cached forecast

Clearing site data resets all of this, including accuracy history.

---

## Where to change things

All in the `<script>` section of `index.html`:

- `MODELS`: the sources and their labels
- `WET_DAY`, `WET_HOUR`: what counts as rain
- `MIN_SCORED`: scored days needed before weighting unlocks
- `ACTIVITIES`: activity rules
- `agreeLevel()`: traffic-light thresholds
- `outfitFor()`: outfit rules and temperature cut-offs
- `DEFAULT_LOC`: starting location

---

## Known limits

- No push notifications or alerts (would need a service worker).
- No minute-by-minute rain nowcast (coverage is patchy outside Europe and North America).
- Outfit cut-offs and the wind-chill allowance are rough starting values.
- Open-Meteo's free tier is for non-commercial use. A commercial product would need their paid plan.
- Attribution to Open-Meteo (CC BY 4.0) is in the page footer. Keep it.

---

## Hosting on GitHub Pages

Put `index.html` in the root of the repository (or the folder Pages is serving) and commit. Keep `README.md` beside it for reference. It isn't needed for the app to run.

---

## Change log

- **v2, 29 Oct 2026:** Agreement score changed to a three-lamp traffic light that also counts temperature spread. Added the "What to wear" card and per-day outfit in the day detail.
- **v1, 29 Oct 2026:** First version. Five-model consensus, agreement score, outlier view, rain windows, hourly strip, best-times presets, accuracy tracker with optional weighting, saved locations, offline cache.

https://alphonsomango86.github.io/Weather_Pal/

[README.md](https://github.com/user-attachments/files/34157242/README.md)
# Weather Consensus

A single-page weather web app. Instead of trusting one forecast, it pulls the same forecast from five independent weather models, compares them, and shows you what the majority agree on, plus how much they disagree.

Everything lives in one file, `index.html`. There is no build step and no server. Host it on GitHub Pages (or open it locally) and it works.

Last updated: 29 Oct 2026 (v3)

---

## What it shows

1. **Headline card.** One line summary ("Rain likely 14:00–17:00", "Dry for the next 12 hours") and the current temperature. Below it:
   - A **today vs yesterday** comment ("Today is much colder than yesterday (6° lower), and wetter.").
   - A **traffic light for today** showing how much the sources agree.
   - **Sunrise, sunset and daylight length.**
   - An **hourly strip** for the next 12 hours. The bar under each hour is the share of sources forecasting rain that hour.
2. **What to wear.** Suggested top for today (rest of the day) and tomorrow.
3. **7-day consensus.** One row per day with the rain verdict, a traffic light, and the high/low. Tap a day to see every source side by side, the outfit, sunrise/sunset and the rain window.
4. **Accuracy tracker.** How each source has performed for your location over time.
5. **Sources.** Switch sources on or off (minimum 3) and turn on accuracy weighting.

### Feels-like vs air temperature
Every temperature is shown as **feels-like first, with the forecast air temperature in brackets**, for example `9° (11°)`. Feels-like comes from Open-Meteo's apparent temperature (it accounts for wind and humidity). The outfit advice and the "today vs yesterday" comment use feels-like. The traffic light's temperature spread and the temperature outlier flag use air temperature, because feels-like varies more between models for reasons unrelated to disagreement about the weather.

---

## Default location

Bromley-by-Bow, London (lat 51.5248, lon -0.0118), which sits in zone 2. Weather models don't resolve differences between zones 1 and 2, so one point covers the whole area. If you used an earlier version, its old central London default is swapped for Bromley-by-Bow automatically. Other places you searched for are kept.

To change the default, edit `DEFAULT_LOC` in the script.

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

One request per location fetches all five models for yesterday plus the next 7 days: hourly temperature, feels-like temperature, rain, wind speed and gusts, plus daily highs and lows (air and feels-like), rain total, max wind, sunrise and sunset. Units are °C, mm and mph. Yesterday is only used for the "today vs yesterday" comment. Place search uses Open-Meteo's geocoding API.

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
- The **median** of the sources (or the accuracy-weighted mean if weighting is on), for both feels-like and air temperature.
- A source whose daily air-temperature high is more than 2.5° from the median is flagged as an outlier.

### Sources that don't reach far enough
Some models don't forecast as far ahead (Météo-France stops at roughly 4 days). Missing values are ignored and the vote uses whoever is left. Under 3 sources, the day gets an "only N sources" warning. Under 2, it shows "Not enough sources".

### Traffic light (agreement)
Three lamps (red, amber, green) with the lit one showing the level, plus text such as "3/5 agree · 5° spread". It appears on every day in the 7-day list and, for today, on the headline card.

- **Green:** at least 85% of the votes back the rain call, and the daily air-temperature highs are within 3°.
- **Amber:** a source disagrees on rain (less than 85% backing), or highs are 3–6° apart.
- **Red:** the rain call is split, or highs are more than 6° apart.

With 5 sources and no weighting, 5/5 is green and 4/5 or 3/5 is amber. **With weighting on**, the 85% is measured by weight rather than by head count, so one dissenting source with a poor track record no longer pulls the light down to amber. The headline light says "weighted" when this is in effect.

Days 5–7 also get a "long range" tag, since disagreement there is expected.

---

## Today vs yesterday

Compares the average feels-like temperature between 07:00 and 21:00 today and yesterday, using the same sources and weighting as the rest of the page.

| Difference | Wording |
|---|---|
| Under 1° | "Today feels about the same as yesterday" |
| 1° to under 2.5° | "Today is a little warmer/colder than yesterday (2° higher/lower)" |
| 2.5° to under 5° | "Today is warmer/colder than yesterday (…)" |
| 5° or more | "Today is much warmer/colder than yesterday (…)" |

It adds ", and wetter" if today has rain and yesterday had none (or 4+ more rainy hours), and ", and drier" for the reverse.

---

## Sunrise and sunset

Taken from Open-Meteo in the local time of the location. The headline card shows today's sunrise, sunset and daylight length. Each expanded day in the 7-day list shows that day's times.

---

## What to wear

Tops only, based on the **feels-like** temperature between 07:00 and 21:00 (for today, from now onwards). The average is rounded first, so the thresholds match the number you see.

| Feels-like | Top |
|---|---|
| Above 18° | T-shirt |
| 10° to 18° | Jumper |
| Below 10° | Thick jumper or double layer |

Then wind and rain adjust it:

| Situation | Addition |
|---|---|
| Rain, warm (above 18°), gusts under 30 mph, no heavy rain | + umbrella |
| Rain otherwise (cooler, gusty, or heavy at 2.5 mm/h or more) | + waterproof |
| Dry but gusts 30 mph+ and not warm | + waterproof (as a windbreaker) |
| Feels-like under 5° | + waterproof (to cut the wind) |
| Consensus dry, but some source shows rain | + dashed "(pack it)" waterproof or umbrella |

Thresholds are the constants `WARM_ABOVE` (18) and `COLD_BELOW` (10).

---

## Accuracy tracking and weighting

1. The first time you open the app on a given day, it records what each source predicts for **tomorrow** (high temperature and rainy/dry), per location.
2. On a later visit, once that day has passed, it fetches what actually happened and scores each source: average error on the daily high, and how often its rain call was right.
3. After **5 or more scored days** for a source, its track record can be used as a weight. Weight = (0.3 + rain hit rate) / (0.5 + average high error). Sources without enough history get the average weight.
4. Weighting is **off by default**. Turn it on under "Sources" once it unlocks. When on, rain votes, temperatures and the traffic light are all weighted.

Caveats:
- "What actually happened" is Open-Meteo's best-estimate analysis, not weather-station readings.
- It only looks back 14 days, so open the app at least every week or two or days are missed.
- History is per location and per browser/device. Records older than 45 days are dropped.
- Accuracy is scored on air temperature, not feels-like.

---

## Colour scheme

Light, earthy palette: sandstone background, stone-grey text and borders, clay-brown accent, with moss green, ochre and terracotta for the good/caution/bad states. The traffic-light lamps keep recognisable red, amber and green. A matching dark version is used automatically if the phone is set to dark mode. All colours are variables at the top of the stylesheet (`:root{…}`), so they're easy to adjust.

---

## Locations, offline use and storage

- Search for a place, or use the 📍 button. Searched places are saved as chips; tap × to remove.
- The last successful forecast per location (including yesterday's data) is cached, so if you're offline it shows the saved forecast with an "Offline" banner.
- Everything is stored in the browser's `localStorage` (nothing leaves your device except the Open-Meteo requests). Keys:
  - `wc_locs`: saved locations
  - `wc_cur`: current location
  - `wc_settings`: selected sources, weighting on/off
  - `wc_log`: logged forecasts waiting to be scored
  - `wc_acc`: accuracy scores
  - `wc_cache_<lat,lon>`: cached forecast

Clearing site data resets all of this, including accuracy history.

---

## Where to change things

All in the `<script>` section of `index.html` (colours are in the `<style>` section):

- `DEFAULT_LOC`: starting location
- `MODELS`: the sources and their labels
- `WET_DAY`, `WET_HOUR`: what counts as rain
- `WARM_ABOVE`, `COLD_BELOW`: outfit temperature thresholds
- `MIN_SCORED`: scored days needed before weighting unlocks
- `agreeLevel()`: traffic-light thresholds
- `outfitFor()`: outfit rules
- `compareYesterday()`: wording and thresholds for the today vs yesterday comment

---

## Known limits

- The "Best times" activity section was removed in v3 (it can be brought back).
- No push notifications or alerts (would need a service worker).
- No minute-by-minute rain nowcast (coverage is patchy outside Europe and North America).
- Wind-based outfit rules (gust thresholds) are rough starting values.
- Open-Meteo's free tier is for non-commercial use. A commercial product would need their paid plan.
- Attribution to Open-Meteo (CC BY 4.0) is in the page footer. Keep it.

---

## Hosting on GitHub Pages

Put `index.html` in the root of the repository (or the folder Pages is serving) and commit. Then in the repository go to **Settings → Pages**, set the source to your main branch and the root folder, and your site appears at `https://YOUR-USERNAME.github.io/YOUR-REPO-NAME/`. Keep `README.md` beside it for reference. It isn't needed for the app to run.

---

## Change log

- **v3, 29 Oct 2026:** Default location changed to Bromley-by-Bow. Removed Best times. Added sunrise/sunset/daylight and a today vs yesterday comment. Traffic light added to today's headline, and it now respects source weighting. Outfit rules changed to T-shirt above 18°, jumper at 18° or below, thick jumper or double layer below 10°, based on feels-like. Feels-like is now the main temperature everywhere, with air temperature in brackets. New light, earthy colour scheme.
- **v2, 29 Oct 2026:** Agreement score changed to a three-lamp traffic light that also counts temperature spread. Added the "What to wear" card and per-day outfit in the day detail.
- **v1, 29 Oct 2026:** First version. Five-model consensus, agreement score, outlier view, rain windows, hourly strip, best-times presets, accuracy tracker with optional weighting, saved locations, offline cache.

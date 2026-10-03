# Shadefinder ☀️🌳

A walking router for the Texas State University campus that routes you through **shade** instead of the shortest path.

> The sun decides which paths cost you. Real buildings, real trees, real weather, and a route that changes by the hour.

<!-- Add a screenshot once you take one (see screenshots/README.md):
![Shadefinder showing the shade route against the shortest route](screenshots/hero.png) -->

## What it does

- **Finds the shadiest route** between two spots on campus, within the extra walking time you're willing to spend, and compares it to the shortest route.
- **Shows live shadows** cast by campus buildings and trees for the current position of the sun.
- **Tells you when to leave:** checks the same walk every 30 minutes over the next five hours and highlights the time with the least direct sun.
- **Factors in the weather:** heat, UV, and cloud cover decide how much a minute of sun is worth avoiding.
- **Shows its data:** a panel lists exactly how many path segments, buildings, trees, and covered walkways were loaded.

You pick the start (A) and end (B) from named campus buildings or by tapping the map, then tune two sliders: how many extra minutes you'll walk, and how much the heat bothers you.

## How it works

Everything is fetched live when the page loads; nothing is precomputed.

1. **Campus graph.** Footpaths, building footprints and heights, and trees come from OpenStreetMap via the Overpass API. Paths become a walking graph (1.35 m/s, with stairs counted as slower), and covered walkways count as full shade.
2. **Sun position.** Computed locally with the NOAA low-precision solar algorithm, so no API is needed for it.
3. **Shadows.** Each building's footprint is projected away from the sun by `height / tan(sun elevation)`. Heights come from OSM `height` or `building:levels` tags, and fall back to 8 m when untagged. Trees cast partial shade from their canopy. Every path segment is sampled about every 7 m to measure how much of it is in sun.
4. **Time-aware shade.** Shadows are recomputed every 8 minutes for the next half hour, and each stretch of the route uses the shadow at the time you would actually reach it.
5. **Cost of sun.** Hourly weather from Open-Meteo sets a weight for direct sun: a hotter "feels like" temperature and higher UV raise it, while cloud cover and a low sun bring it down. Each path segment costs `walking time × (1 + weight × fraction in sun)`.
6. **Routing.** Dijkstra's algorithm finds the cheapest route. If it costs more extra time than your budget allows, the sun weight is stepped down until the route fits.

## Run it locally

It's a single `index.html` with no build step, no dependencies, and no API keys. Serve it with any static server:

```bash
git clone https://github.com/aviyannn/shadefinder.git
cd shadefinder
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

The public Overpass servers throttle heavy use. If the map says it couldn't load the data, wait a minute and click **Try again**.

## Use it somewhere else

Change `BBOX` (south, west, north, east) and `CENTER` at the top of the script to any area with decent OpenStreetMap coverage.

## Limitations

- Shadows are only as good as the map data: untagged buildings are assumed to be 8 m tall, and only trees mapped in OpenStreetMap cast shade.
- Shadows are cast onto flat ground, so terrain and elevation are ignored.
- Weather comes from the hourly forecast nearest to the chosen time.

## Built with

Vanilla JavaScript and SVG, with no framework.

- Map data © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors, via the [Overpass API](https://overpass-api.de/)
- Weather by [Open-Meteo](https://open-meteo.com/)

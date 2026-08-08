# BMX Tracks in California

An interactive atlas of BMX race tracks in California — active, unconfirmed, and closed — from the sport's 1969 origin in a Los Angeles city park to today. Built for [Post Millennium Renaissance](https://postmillenniumrenaissance.com/).

Single-file HTML. No build step, no external dependencies, no CDN calls. Open it in a browser and it runs.

---

## What's in it

127 tracks across 9 California regions and 39 counties, presented over seven tabs:

| Tab | What it shows |
|---|---|
| **Overview** | Headline counts, a plain-language key to the four track statuses, and a regional breakdown |
| **Map** | A hand-drawn SVG of California (no map tiles, no Leaflet) with every track plotted |
| **Directory** | A searchable, sortable table of every track and its sourcing |
| **Growth** | Estimated track count by five-year increment, 1970–2025, as a banded line chart |
| **Sanctioning Bodies** | A parallel timeline of BMX's competing governing bodies (NBA, ABA, NBL, UBR, USA BMX) |
| **Search Interest** | US Google Trends data for "BMX," 2004–present, annotated with Olympic and merger events |
| **Method & Sources** | How the data was compiled, what's uncertain, and where every claim came from |

Also included:

- A **collapsed origin card** at the top telling the story of Palms Park (1969), the widely-credited birthplace of BMX — expands on click, doesn't clutter the page by default.
- **Three color schemes** — Powerlite, Robinson, and Auburn, named for BMX-era frame brands. The switcher remembers your choice (`localStorage`) and re-renders every chart, since chart colors are read live from CSS variables rather than baked in.
- **Live search and filtering** across name, city, county, region, sanctioning body, and district code, shared between the Map and Directory tabs.
- A **tiled artwork band** behind the page title (see [Hero artwork](#hero-artwork) below).

---

## Data model: four statuses, not eight

Every track has exactly one status, and all four answer the same question — *could you go and ride there?*

| Status | Meaning |
|---|---|
| `active` | Confirmed racing on a schedule published for the current season |
| `unconfirmed` | Named in an older directory, but no current schedule found |
| `building` | Exists as a project, not yet a racing venue |
| `closed` | No longer operating |

Two things that are *not* statuses, because they don't answer that question, are **flags** layered on top instead:

- `national` — the track hosts a USA BMX national event (shown as a ring around the marker, `NAT` badge in the table)
- `birthplace` — the birthplace of BMX. Exactly one track carries this.

If you're extending this file, a new status should only exist if it answers "could you ride here?" differently than the four above. Anything else — an award, a sanctioning body, a notable event — is a flag.

### Confidence tiers

Every record also carries a confidence tier, independent of status:

- **Verified** — confirmed via a current (2024–2026) source: a live schedule, recent news coverage, an active event page
- **Estimated** — sourced from an archived ABA/NBL statewide directory dated circa 2010–11; a real lead, not individually reconfirmed
- **Draft** — sourced from rider recollection (forum threads) rather than a primary source; treat names and dates as approximate

Currently: 21 verified, 32 estimated, 74 draft. The draft tier is large by design — most closed 1970s–80s tracks survive only in rider memory, and that's worth recording, but it's a different kind of fact than a published schedule. Marker opacity on the map follows this tier directly: the fainter a marker, the thinner the evidence behind it.

---

## Hero artwork

The title band is a full-bleed tiled panel, configured in `CONFIG.hero` near the top of the script:

```js
hero:{
  image:"moose-pattern-tile.jpg",   // relative to this HTML file
  tileWidth:420,                    // px width of one repeat
  scrim:0.78                        // 0–1, darkens the tile for legibility
}
```

**Drop `moose-pattern-tile.jpg` in the same folder as the HTML file in this repo.** Until it's there, the band falls back to a plain dark panel — nothing breaks. Set `image:""` to disable the artwork entirely.

The current tile repeats **left-to-right only** (`background-repeat: repeat-x`), not on both axes. It was tested before shipping: the left and right edges match well, but the top and bottom edges don't line up, so a full tile repeat would show a visible seam. If you swap in a fully seamless tile later, change `background-repeat` to `repeat` in the `.hero-band` CSS rule.

---

## Extending this file

Everything you'd normally want to change lives in the `CONFIG` and `DATA` blocks near the top of the `<script>` — you shouldn't need to touch rendering code.

| To add | Edit |
|---|---|
| A track | Push an object into `DATA.tracks` |
| A region | Add a key to `DATA.regions`, use it on a track's `r` field |
| A color scheme | Add an object to `THEMES` (see the existing three for the required variable set) |
| A tab | Add an entry to `TABS` with a `render()` function returning an HTML string |
| A chart | Call `Chart.xy()`, `Chart.bars()`, or `Chart.gantt()` — all hand-rolled SVG, no dependency |
| A status | Only if it answers "could you ride here?" differently — see [Data model](#data-model-four-statuses-not-eight) |
| A flag | Add a key to `FLAGS`, set `flagkey:true` on any track |
| Growth estimates | `DATA.growth` |
| Sanctioning body timeline | `DATA.leagues` |
| Google Trends data | `DATA.trends` |
| A whole additional state | Add an outline to `MAPS`, add tracks with a matching `state` — the projection and marker code are state-agnostic |

Every track record looks like this:

```js
{n:"Track Name", c:"City", co:"County", r:"region-key",
 s:"active", lat:34.02, lon:-118.41,
 f:"Founded 1976", fy:1976,
 w:"https://example.com", d:"CA14", sanction:"USA BMX",
 conf:"verified", note:"Free-text sourcing note."}
```

`fy` (founded year, numeric) feeds the growth chart's cross-checks; `f` is the human-readable display string and can say "Unknown."

---

## Sources & methodology

Full source list and a worked example of why the confidence tiers matter live in the **Method & Sources** tab inside the tool itself — that's the canonical version, kept in sync with the data. Primary sources include USA BMX's live event pages, the Golden State BMX / NorCal 4-Star Series schedule, an archived ABA/NBL statewide track directory (circa 2010–11), regional newspaper coverage of specific closures and openings, Wikipedia's sanctioning-body histories, and BMXmuseum.com rider-recollection forum threads for pre-2000s tracks.

This is a first-pass compilation, not a registry. California has had well over 300 BMX tracks come and go since 1969; most left almost no durable record. Roughly a third of the currently-listed tracks rest on that single 2010–11 directory and haven't been individually reconfirmed — the single highest-value next step for this dataset.

---

## Tech notes

- Zero dependencies: no CDN scripts, no build tooling. `Chart.xy/bars/gantt` are ~150 lines of hand-rolled SVG generation.
- Theming is CSS custom properties on `:root`; charts read `var(--accent)` etc. at render time, so switching themes re-colors every chart without regenerating data.
- `localStorage` calls are wrapped in `try/catch` — the tool works in private browsing or with storage disabled, it just won't remember your theme choice.
- Map projection is a simple equirectangular projection scoped to each state's bounding box in `MAPS` — no tile server, no API key.

## License / attribution

Internal PMR tool. Track data is compiled from public sources cited in the Method & Sources tab; verify before using anywhere that accuracy matters.

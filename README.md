# The Archives — Social Graphic Generator

Single-page tool that produces the three match graphics we post: **matchup**,
**spoiler warning** and **results**. Fill in the division, both teams, their
logos and the score; get PNGs at 1024 × 1024, either one at a time or all three
as a ZIP.

Everything happens in the browser. There is no server-side code, no database and
no API keys — the whole app is `index.html` plus the artwork in `template.svg`.

This repository holds the app only. Hosting is maintained separately by the
maintainer; changes arrive as pull requests, get reviewed, and are deployed from
here. Nothing needed to run or develop the app lives outside this repo.

## Running it locally

It must be served over HTTP; opening `index.html` from disk will not work because
the app `fetch()`es `template.svg` and the browser blocks that on `file://`.

```bash
python3 -m http.server 8000      # or: npx serve .
# then open http://localhost:8000/
```

Any static server is fine. No build step, no dependencies to install.

## How it works

`template.svg` is one artboard strip, 1024 px wide per graphic:

| Artboard | x offset | Exported as |
| --- | --- | --- |
| matchup | 0 | `<teams>_matchup.png` |
| spoiler warning | 1074 | `<teams>_spoiler-warning.png` |
| results | 2148 | `<teams>_results.png` |

At load, `template.svg` is fetched as text and parsed with `DOMParser`. For each
render, the app:

1. clones the parsed document, replaces the two logo `<use>` placeholders with
   `<image>` elements carrying the chosen logo as a data URL;
2. rewrites the team names and division, measuring the string in Bebas Neue and
   shrinking it to fit its slot (see `centreText`);
3. hands the whole strip to [canvg](https://github.com/canvg/canvg) to rasterise
   the selected artboard onto a 1024 × 1024 canvas;
4. converts the canvas to a PNG blob and downloads it, or zips all three with
   [JSZip](https://stuk.github.io/jszip/).

Because the artwork is edited elsewhere (Affinity), the code deliberately does
not hardcode text positions: `fullTransform`/`pointInRoot` walk the nested
`transform` attributes so measured widths and centres stay correct even when the
design file moves things around. Logo slots are found by element id
(`team1Logo`, `team2Logo`, and the matching name groups).

External runtime dependencies, both from jsDelivr: `canvg@4.0.3`, `jszip@3.10.1`.
The Bebas Neue webfont comes from Google Fonts. **No network, no renders** — the
app says so rather than silently substituting a fallback font.

## Logo URLs and CORS

Logos can be uploaded (used as-is) or pulled from a URL. Many hosts do not send
`Access-Control-Allow-Origin`, so a direct `fetch()` is refused even though the
image is public. When a direct request fails, the app retries through the
same-origin endpoint `/logo-proxy/<host>/<path>`.

That endpoint is a server-side fetcher provided by the deployment, not by the
app. Served anywhere else, the fallback simply 404s and the upload path still
works — so a local checkout is fully usable without it. Which hosts it accepts
is a hosting decision, deliberately allowlisted rather than open.

## Layout

```
index.html            the entire application (markup, styles, script)
template.svg          source artwork: three 1024x1024 artboards side by side
favicon.ico           tab icon, 16/32/48/64, shared with our other tools
apple-touch-icon.png  180px iOS home-screen icon
```

A clone can be served from its root as-is.

## Editing the artwork

`template.svg` is the design source, exported from Affinity — 3.5 MB, with six
embedded base64 images. Keep the artboard order and the 1024 px widths, and keep
the `team1Logo` / `team2Logo` / name group ids that the script looks up; the
positions themselves are free because everything is measured at render time.

## Contributing

Fork, branch, and open a pull request against `main`. Keep changes to the app —
`index.html`, `template.svg`, the icons — so the PR is reviewable and can be
deployed as-is. Please check that a checkout still renders all three artboards
and exports a ZIP before opening it.

## Credits

The favicon (TA emblem) is reused from
<https://thearchiveslol.com/icons/TA_Icon.png> with permission.

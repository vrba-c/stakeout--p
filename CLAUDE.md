# Stakeout

Offline Progressive Web App for surveyors: navigate to points from Gauss-Krüger, UTM or local-grid point files, outdoors (phone GPS or external RTK receiver) and indoors (dead-reckoning). Proof of concept for internal use at Woolpert on the ESMC fab site in Dresden.

## Files (all at repo root, served by GitHub Pages)
- `index.html` — the whole app: HTML, CSS and JS inline in one file
- `sw.js` — service worker for offline caching
- `manifest.webmanifest` — PWA manifest (must keep this exact name; sw.js caches it)
- `icon-192.png`, `icon-512.png`

No build step, no package manager, no framework.

## Rules
- **Bump the cache version in `sw.js` (`stakeout-vN`) on every change**, or phones keep the old version.
- Keep everything self-contained in `index.html`. No CDNs or external requests: the app must work with zero connectivity after first load.
- Never hardcode site parameters (origin, rotation, grid). They come from the user's site setup file.
- Don't break the site setup file or CSV export formats; colleagues share these files between phones. If a format must change, keep reading the old one.
- Keep files at the repo root, not in a subfolder.

## Platform
- Primary target: Chrome on Android (Samsung Galaxy S23 Ultra, Galaxy A37 5G).
- Web Bluetooth only works in Chrome on Android, not iOS Safari.
- Service worker and motion sensors require HTTPS.

## Key features (don't regress these)
- Coordinate transforms for GK / UTM / local site grid
- Indoor dead-reckoning: building-axis heading snapping, calibration walk (A→B sets step length and heading), grid detection and auto-correction at 90° turns
- External RTK receiver over BLE (Nordic UART Service), parsing NMEA GGA (position, fix type) and GST (accuracy)
- Falls back to phone GPS if the receiver loses its fix for 3 s
- "Arrived" threshold: 5 cm with RTK, 0.5 m otherwise
- Automatic RTK-to-indoor handover: learns heading and step length from RTK, takes over when the fix is lost, closes the loop when RTK returns
- Multi-site storage, point condition reports (found / disturbed / covered / missing), CSV exports, indoor-check and position logs
- Draggable, pinnable map with tap-to-select points

## Testing
- Syntax check the inline script: extract the `<script>` contents and run `node --check`.
- Serve locally: `python3 -m http.server 8000`, then test in the browser.
- For tracking and receiver logic, use headless Playwright simulations (simulated walks, fed NMEA sentences) before shipping.
- Real field results beat simulation; treat simulated accuracy as optimistic.

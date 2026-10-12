# Fuel Split

Track a journey by GPS, work out the fuel cost from your car's mpg and the price per litre,
split it between whoever was in the car, and keep a running tab of who owes what.

No build step. Four static files: `index.html`, `manifest.webmanifest`, `sw.js`, `icon.svg`.

## Run it

GPS only works on **HTTPS** (or `http://localhost`). The easy options, all free:

- **Netlify Drop**: go to app.netlify.com/drop and drag this folder in. You get an https link.
- **GitHub Pages**: push the folder to a repo, then Settings, Pages, deploy from the main branch.
- **Cloudflare Pages**: connect the repo, no build command, output directory `/`.

Locally: `python3 -m http.server 8000` in this folder, then open `http://localhost:8000` on the
same computer. To test on your phone use one of the https options above.

On your phone, open the link, then Add to Home Screen. It then opens full screen like an app.

## How GPS tracking works

- Start trip asks for location permission, then reads a position every few seconds.
- It adds up the straight-line distance between fixes (haversine formula).
- It ignores fixes worse than 50 m accuracy, movement under 10 m (stops jitter at traffic lights),
  and jumps faster than 70 m/s.
- The screen wake lock keeps the phone awake while tracking.
- The trip is saved as it goes. If the page is closed or reloaded you get Resume / Use X mi / Discard.
  The gap while the page was closed is not counted.

### Limits you should know about

- **A web page cannot track in the background.** Once the screen locks or you switch app, iPhone
  Safari stops sending location, and Android Chrome often does too. Keep the screen on, with the
  phone in a mount. Real background tracking needs a native app (Expo / React Native, or Capacitor).
- Straight lines between fixes cut corners, so expect it to read about 1 to 3 percent low
  compared with the car's trip meter. Tunnels and weak signal lose distance.
- Data lives in this browser's storage (`localStorage`). Use Details, Back up data, now and then.
  Clearing site data deletes it, and it does not sync between devices.

## Fuel prices

The price field starts at the UK average (set at the top of the script: `AVG`, `AVG_DATE`).
Edit it to what you actually paid. Supermarkets are usually cheaper and motorway services dearer
than the average.

To fetch prices automatically, set `PRICE_URL` in `index.html` to an endpoint that returns:

    { "petrol": 174.9, "diesel": 199.5, "asOf": "week to 12 Oct 2026" }

The UK government's Fuel Finder API (gov.uk, "Access the latest fuel prices and forecourt data via
API or email") needs OAuth client credentials from its developer portal, so call it from a small
server (a Cloudflare Worker is enough), average the stations you care about, and return the JSON
above. Never put the credentials in `index.html`.

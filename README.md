# KoBo AOI Geofence

A field companion tool for [KoBoToolbox](https://www.kobotoolbox.org/) that watches your live GPS location against a drawn **area of interest (AOI)**, alerts you the moment you step outside it, and only unlocks your KoBo data-collection form while you're confirmed inside the boundary.

Built for farm/plot surveys, but the tracker itself is generic — any GeoJSON boundary and any deployed KoBo form will work.

---

## Why this exists

KoBoToolbox forms (XLSForms rendered through Enketo) have no way to:
- continuously watch GPS in the background,
- test whether a point falls inside an arbitrary polygon, or
- play an alert sound.

Forms only react when a question is answered — they don't run a location loop. So the real-time "alarm if I leave the area" behavior has to live in a browser tool with its own GPS watch and geospatial math (this repo), not inside the form itself.

The form still plays a role: it carries a lightweight self-attestation question ("Are you currently inside the area of interest?") and silently logs the enumerator's GPS position at that moment for an audit trail — but the actual detection, alarm, and access control happen in the tool.

---

## What's in this repo

```
kobo-aoi-geofence/
├── tool/
│   └── kobo-geofence.html      # The geofencing + form-embedding web app (open this file in a browser)
├── forms/
│   └── farm_plot_collection.xlsx   # Example XLSForm, ready to upload to KoBoToolbox
├── docs/                       # Space for screenshots / diagrams if you add them
├── LICENSE
└── README.md
```

---

## The tool (`tool/kobo-geofence.html`)

A single self-contained HTML file — no build step, no server required. Open it directly in a browser (works well on a phone browser in the field).

**Three tabs:**

| Tab | What it does |
|---|---|
| **Map** | Shows your AOI boundary and your live position, with an accuracy circle. Follows you as you move. |
| **Form** | Embeds your KoBoToolbox form (via its Enketo web-form link). Locked until you're inside the AOI. |
| **Setup** | Connect to KoBoToolbox, load your AOI, and configure alert settings. |

**Status bar** at the top turns green ("INSIDE") or pulses red ("OUTSIDE — MOVE BACK IN"), shows live distance to the boundary, and sounds a repeating alarm + vibration while you're outside. Tap it to mute/unmute.

### Getting your area of interest in

Pick whichever matches your KoBo project:
- **A GeoJSON file uploaded as form media** — upload it under your project's `Settings > Media` tab, then the tool fetches it via the KoBo API.
- **Drawn as geoshape/geotrace submissions** — the tool can pull `/data.geojson` from your project instead.
- **Manual fallback** — paste GeoJSON directly or upload a `.geojson`/`.json` file in the tool itself. This always works, even if the API calls above are blocked by your server's CORS settings.

### Getting your form to display

The tool tries to fetch your form's Enketo link automatically via the KoBo API. If that fails (common CORS/auth issue), use the manual fallback field in Setup:
1. In KoBoToolbox, go to your project's **FORM** tab → **Collect data**
2. Change the dropdown to **"Embeddable web form code"** (the regular "Online-Offline" link refuses to be embedded on purpose)
3. Click **Copy** and paste the link (or the whole `<iframe>` snippet — the tool extracts the URL either way) into the **Enketo form link (fallback)** field → **Load this form link**

### What's remembered on your device

Once you connect successfully, your server/token/asset UID, your loaded AOI, and your form link are saved locally (via the browser's own storage for this page) so you don't have to redo setup every time you open it. Clear it anytime with the **Reset** button in Setup.

---

## The form (`forms/farm_plot_collection.xlsx`)

An [XLSForm](https://xlsform.org/) ready to upload to KoBoToolbox as-is (**Projects → New → Upload an XLSForm**), or use as a starting point for your own.

**Fields:**
1. *Are you currently inside the area of interest?* (Yes/No gate — must be Yes to continue; also silently logs GPS in the background)
2. Farm owner name
3. Crop type (dropdown)
4. GPS point collection
5. Farm boundary (polygon) — walk the perimeter
6. Area — calculated automatically in **m²** and **hectares** from the traced polygon
7. Survey date (defaults to today)
8. Optional photo — Yes/No, camera only appears if Yes
9. Village / Ward / District

Validated with [`pyxform`](https://github.com/XLSForm/pyxform) (KoBo's own conversion engine) before being added here, so it should import cleanly.

---

## Quick start

1. Build/upload your form in KoBoToolbox (or use `forms/farm_plot_collection.xlsx`) and **deploy** it.
2. Get an API token from your KoBo account (**Account Settings → Security**).
3. Note your form's asset UID from its URL: `.../#/forms/<THIS_PART>/...`
4. Open `tool/kobo-geofence.html` in a browser.
5. Go to **Setup**, enter your server/token/UID, and load your AOI.
6. Switch to **Map**, allow location permission, and confirm you see your position and the boundary.
7. Once inside the AOI, the **Form** tab unlocks — collect data as normal.

---

## Limitations / honest caveats

- This is a client-side tool: your API token is stored in your browser only, never sent anywhere but your KoBo server.
- Some KoBo server configurations block cross-origin API calls (CORS); when that happens, use the manual GeoJSON/Enketo-link fallbacks described above.
- GPS accuracy depends on your device and environment — the tool shows the reported accuracy radius so you can judge how much to trust a "just inside/outside" reading near the boundary.
- The "inside AOI" question in the form is a manual confirmation, not automatic — the form itself cannot verify location. Treat it as a prompt for the enumerator, not a hard guarantee.

---

## License

MIT — see [LICENSE](LICENSE).

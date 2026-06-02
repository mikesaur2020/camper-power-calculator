# Mike's Camper Power Calculator

An installable, offline-capable Progressive Web App (PWA) built specifically for boondocking — generator-only camping with no shore power.

Built for:

- **2025 Coachmen Apex Ultra-Lite 28RBS**
- **WEN DF360iX Dual-Fuel Inverter Generator**
- **GE 15,000 BTU A/C with Micro-Air EasyStart**
- **Starlink Mini**
- **300W Roof Solar System**
- **Dual SRM24 Flooded Lead-Acid Battery Bank (136Ah at 12V)**

The application helps answer:

- How long will my fuel last at this load?
- Which fuel source is currently active?
- Will I make it through the night?
- Can I safely run this appliance combination?
- How much generator headroom do I have?

**Live at:** https://mikesaur2020.github.io/camper-power-calculator/

See [PROJECT_VISION.md](PROJECT_VISION.md) for equipment specs and design decisions.
See [AI_HANDOFF.md](AI_HANDOFF.md) for feature inventory, known issues, and future enhancements.

---

## Key Features

| Feature | Description |
|---|---|
| ⚡ **Generator Load Calculator** | Live load summary across 16 appliances with Good / Near Limit / Over Limit status for both gasoline and propane |
| ⛽ **Live Fuel Tracker** | Active fuel source, propane runtime, gasoline reserve runtime, combined potential runtime, and Overnight Confidence |
| 📚 **Fuel Burn Reference** | Planning tables showing burn rates and runtime estimates at different load levels |
| 🌙 **Overnight Confidence** | High / Moderate / Low confidence based on estimated runtime vs. a full night |
| 🏕️ **Camping Presets** | One-tap presets for Normal A/C, Overnight, Microwave, Coffee Time, Hair Dryer — and user-defined custom presets |
| 📱 **Installable PWA** | Add to Home Screen on iPhone — runs as a standalone app |
| 💾 **Offline Operation** | Fully functional without internet after first load (service worker caching) |
| ☀️ **Solar & Battery Awareness** | Battery Charge State and Charging Strategy (Solar Only / Generator Assist) affect generator load estimates |
| 📍 **Elevation-Aware Planning** | GPS elevation lookup or manual entry; generator output is derated ~3.5% per 1,000 ft |
| 🔁 **Auto Fuel Selection Modeling** | Accurately reflects the WEN DF360iX propane-priority behavior — gasoline shown as reserve, not simultaneously consumed |

---

## Recommended Workflow

1. **Open the Calculator tab** and select a preset (Normal A/C, Overnight, etc.)
2. **Verify appliance selections** — enable or disable individual loads as needed
3. **Open Live Fuel Tracker** (default tab)
4. **Check the active fuel source** — Propane or Gasoline
5. **Review runtime estimates** — propane runtime, gasoline reserve, combined
6. **Review Overnight Confidence** — High, Moderate, or Low for both propane-only and combined scenarios
7. **Make fuel decisions** — refuel, adjust load, or plan a manual propane→gasoline transition as needed

---

## Design Philosophy

This application is intentionally:

- **Camper-specific** — built for one rig, one generator, one use case
- **Offline-capable** — works without internet after first load
- **Self-contained** — no backend, no accounts, no cloud sync
- **Practical** — focused on real camping decisions, not theoretical precision

It does **not** attempt to:

- Replace a generator fuel gauge
- Predict exact fuel consumption
- Replace manufacturer specifications or generator monitoring equipment

All runtime estimates are proportional estimates based on WEN/Home Depot published half-load specs. Real consumption varies with temperature, elevation, A/C duty cycle, fuel quality, generator condition, and ECO mode.

---

## Tab Structure

| # | Tab | Purpose |
|---|---|---|
| 1 | **Live Fuel Tracker** | Default tab — runtime, fuel source, overnight confidence |
| 2 | **Calculator** | Appliance load calculator and generator status |
| 3 | **Real-World Tests** | Track appliance combinations verified in the field |
| 4 | **Fuel Burn Reference** | Planning tables — burn rates at different load levels |
| 5 | **Ambient & A/C** | Duty cycle guidance by temperature and setpoint |
| 6 | **About** | Equipment specs and assumptions |

---

## WEN DF360iX Auto Fuel Selection

Per the owner's manual:

> "LPG is prioritized. If a propane tank with enough LPG is connected, the generator will automatically use LPG. The LPG regulator hose must be disconnected for the generator to switch to gasoline. If the LPG regulator hose is connected and you run out of LPG, the generator will not automatically switch to gasoline."

**What this means in the app:**

| Scenario | Active Fuel | Gasoline |
|---|---|---|
| Propane connected | 🔥 Propane | Reserve only — not consumed |
| Propane not connected | ⛽ Gasoline | Active |
| Neither | — | Generator cannot run |

**To switch from propane to gasoline:**
1. Shut down or allow generator to stop when propane is exhausted
2. Disconnect the LPG regulator hose
3. Restart the generator
4. Generator will now run on gasoline

**Combined runtime** = propane runtime + gasoline reserve runtime — requires the manual 4-step transition above.

---

## Elevation & Generator Derating

Generator output decreases approximately **3.5% per 1,000 ft** above sea level.

| Elevation | Derating | Gas Running | Gas Peak | Propane Running | Propane Peak |
|---|---|---|---|---|---|
| Sea level | 0% | 2,900W | 3,600W | 2,600W | 3,500W |
| Sioux Falls (1,400 ft) | ~5% | 2,758W | 3,424W | 2,473W | 3,329W |
| Denver (5,280 ft) | ~18% | 2,364W | 2,935W | 2,120W | 2,853W |
| 7,000 ft | ~25% | 2,190W | 2,718W | 1,963W | 2,643W |
| 8,000 ft | ~28% | 2,088W | 2,592W | 1,872W | 2,520W |
| 9,000 ft | ~32% | 1,986W | 2,466W | 1,781W | 2,398W |
| 11,000 ft | ~39% | 1,784W | 2,214W | 1,599W | 2,153W |

**Elevation sources:** Presets, GPS (📍 Use My Location via Open Elevation API — requires internet), or manual entry. GPS stores only the resulting elevation in feet — no lat/lon is saved.

---

## Setup

### Run locally

```bash
cd camper-power-calculator
python3 -m http.server 8000
```

Then open: [http://localhost:8000](http://localhost:8000)

### Deploy to GitHub Pages

1. Push to GitHub (main branch, root folder)
2. **GitHub repo → Settings → Pages → Deploy from branch → main → / (root)**
3. Public URL: **https://mikesaur2020.github.io/camper-power-calculator/**

### Install on iPhone

1. Open the URL in **Safari**
2. Tap **Share → Add to Home Screen**
3. App installs as a standalone PWA with its own icon and offline support

---

## File Structure

```
index.html          HTML shell — tabs, panels, modals
style.css           All styles (mobile-first dark theme)
app.js              All logic, state, calculations, and HTML builders
manifest.json       PWA manifest
service-worker.js   Network-first offline caching
icon-192.png        PWA home screen icon (192×192)
icon-512.png        PWA home screen icon (512×512)
PROJECT_VISION.md   Equipment specs and design decisions
AI_HANDOFF.md       Feature inventory, known issues, future enhancements
README.md           This file
```

---

## Updating Watt Values

All appliance definitions are at the top of `app.js` in the `APPLIANCES` array. Edit `running` (watts while on) and `surge` (additional watts at startup), save, and reload. No build process required.

Generator ratings are in `GEN`. Fuel burn specs are in `FUEL`. Battery charging loads are in `BATTERY_LOAD`.

---

## Watt Values Reference

| Appliance | Running W | Startup Surge W |
|---|---|---|
| A/C Cooling (w/ EasyStart) | 1,700 | 750 |
| A/C Fan Only | 250 | 50 |
| 12V Refrigerator | 120 | 120 |
| Starlink Mini | 40 | 0 |
| USB Charging | 50 | 0 |
| Smart TV | 100 | 0 |
| Microwave | 1,500 | 0 |
| Toaster | 1,200 | 0 |
| Coffee Maker | 1,000 | 0 |
| Hair Dryer | 1,500 | 0 |
| Clothes Iron | 1,200 | 0 |
| Water Pump | 120 | 180 |
| Furnace Blower | 350 | 350 |
| Bathroom Vent Fan | 40 | 40 |
| Range Hood Fan/Light | 50 | 25 |
| Interior LED Lighting | 75 | 0 |

---

## Source Assumptions

- **Generator specs**: [WEN DF360iX product page](https://wenproducts.com/products/wen-df360ix-quiet-and-lightweight-3600-watt-dual-fuel-rv-ready-portable-inverter-generator-with-fuel-shut-off-and-co-watchdog)
- **Fuel runtime**: [Home Depot listing #330761409](https://www.homedepot.com/p/WEN-Quiet-and-Lightweight-3600-Watt-Dual-Fuel-RV-Ready-Portable-Inverter-Generator-with-Fuel-Shut-Off-and-CO-Watchdog-DF360iX/330761409) — ~5 hrs gasoline at half-load (1.5 gal), ~11 hrs propane at half-load (20 lb)
- **A/C**: GE 15,000 BTU with Micro-Air EasyStart — EasyStart reduces startup surge only, not running watts (~1,700W)
- **Solar**: 300W roof panel reduces battery charging demand; not counted as generator output
- **Batteries**: 2 × SRM24 flooded lead-acid, 68Ah each = 136Ah at 12V in parallel

---

*All estimates are approximate. Real-world generator output and fuel consumption vary with temperature, elevation, A/C duty cycle, fuel quality, and generator condition. This application is a planning aid — not a substitute for monitoring your generator directly.*

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project: Indoor Drone Mission Creator

Stores and generates Avilon Photon indoor drone mission files from warehouse shelf layout data.

- **Related repos:** Cart-kart/Yusen-Drone-Dashboard, Cart-kart/HD-DHL-Drone-Dashboard
- **IMPORTANT:** X/Y/Z coordinates, transit heights, aisle widths, and bay positions are site-specific. Do not assume values from one site apply to another.

## File Types

| Extension | Role |
|---|---|
| `*.mission` | Drone flight path — comma-separated waypoints fed to Avilon autopilot |
| `TAG_*.json` | AprilTag navigation marker config for the drone |
| `Layout_Locate_*_WMS_L.csv` | WMS shelf layout export — Left side locations |
| `Layout_Locate_*_WMS_R.csv` | WMS shelf layout export — Right side locations |

See [docs/format-reference.md](docs/format-reference.md) for full format specs.

## Folder Structure

```
clients/
├── DHL/
│   ├── MISSION/    ← flight path files by zone/aisle
│   └── JSON/       ← TAG config files per aisle pair
├── Yusen/
│   └── BMW/
│       └── N06/    ← Aisle N06 mission + tags + WMS CSVs
└── Avilon/         ← Avilon internal / test sites
docs/
└── format-reference.md
```

## Key Rules

- `y` is always `0` on all waypoints — drone flies aisle centerline
- Tag `y` = `-(aisle_width / 2)` — always negative (left wall offset)
- Tag pose constants never change: `roll=1.57, pitch=0.0, yaw=-3.14`
- Scan pattern is serpentine — alternates direction each level

## Yusen BMW Location Code Convention

`G[aisle][bay_code][level][slot]`

- Aisle: 2-digit (e.g. `06` for N06)
- Bay codes Left: even, descending (22→02, step −2)
- Bay codes Right: odd, descending (21→01, step −2)
- Level: single digit (1–7); Slot: 10=front, 20=back

Example: `G0622210` = Aisle 06, Left bay1, Level 2, front slot

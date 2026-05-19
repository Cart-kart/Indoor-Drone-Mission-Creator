# Indoor Drone Mission Creator

Flight path and AprilTag configuration files for Avilon Photon indoor drone inventory scanning.

## File Types

| Extension | Role |
|---|---|
| `*.mission` | Drone flight path — comma-separated waypoints fed to Avilon autopilot |
| `TAG_*.json` | AprilTag navigation marker config for the drone |

## Folder Structure

```
clients/
├── DHL/
│   ├── MISSION/    ← flight path files organized by zone/aisle
│   └── JSON/       ← TAG config files per aisle pair
├── Yusen/          ← Yusen customer sites
└── Avilon/         ← Avilon internal / test sites
docs/
└── format-reference.md    ← file format specs
```

## Quick Reference

### .mission format

Each line: `x, y, z, yaw, type, param, [LeftLocation|RightLocation]`

| type | param | meaning |
|---|---|---|
| 2 | 2 | scan point (has location label) |
| 3 | 6 | navigate / turn waypoint |
| 3 | 7 | START marker |
| 3 | 8 | END marker |

- `y` is always `0` on all waypoints (drone flies aisle centerline)

### TAG .json format

```json
{
  "Standalone": { "tag_dictionary_index": 20, "tag_length": 0.16, "tags": [] },
  "MainQuad":   { "tag_dictionary_index": 20, "tag_length": 0.3, "tag_indiv_size": 0.06,
                  "tags": [[tag_id, x, y, z, 1.57, 0.0, -3.14], ...] }
}
```

Tag pose constants (always the same): `roll=1.57, pitch=0.0, yaw=-3.14`

- `x` = position along aisle
- `y` = `-(aisle_width / 2)` — negative, tag is on left wall
- `z` = tag center height above floor

See [docs/format-reference.md](docs/format-reference.md) for full details.

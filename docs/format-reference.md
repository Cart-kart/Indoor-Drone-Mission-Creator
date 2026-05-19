# File Format Reference

## .mission File

Each line: `x, y, z, yaw, type, param, [LeftLocation|RightLocation]`

Trailing comma required on every line.

### type / param codes

| type | param | meaning |
|---|---|---|
| 0 | 0 | home point / land |
| 1 | 0 | takeoff or land (Avilon test style) |
| 2 | 2 | **scan point** — carries `LeftLocation\|RightLocation` label |
| 3 | 6 | navigate / turn waypoint |
| 3 | 7 | START marker |
| 3 | 8 | END marker |
| 0 | 6 | descend step (Yusen style) |
| 3 | 0 | land (DHL style) |

### Scan pattern

Serpentine — drone alternates direction each level so no wasted traversal at level transitions.
Turn waypoints (`3,6`) are placed only when the drone needs to cross to the other end of the aisle.

`y` is always `0` on all lines — drone flies the aisle centerline.

---

## Client Patterns

### DHL

```
0,0,1.7,0,3,6,              ← go to transit height (no home/takeoff line)
0,0,1.7,0,3,7,              ← START level 2
[scan bays left→right at z=2.6]
x,0,transit_h,0,3,6,        ← mid-aisle transit at each tag X position
[continue scan]
far_x,0,transit_h,0,3,8,    ← END
far_x,0,transit_h,0,3,6,
far_x,0,transit_h,0,3,7,    ← START level 3 (same end, direction reversed)
[scan bays right→left at z=4.1]
...
0,0,1.7,0,3,8,
0,0,1.7,0,3,6,
0,0,1,0,3,0,                ← land
```

- Transit waypoints appear at every tag X position along aisle
- Long aisles are split into P1 / P2 / P3 files
- Transit height sits above scan height (e.g. transit=4.89m when scan=4.1m)

**Location code:** `EAB-[level]-[bay]-0-g-l | EAA-[level]-[bay]-0-g-l`

**File naming:**
- Mission: `E[nn]_[LeftAisle]_[RightAisle]_P[n].mission`
- Tag JSON: `T[nn]_[LeftAisle]-[RightAisle].json`

### Yusen BMW

```
0,0,1,0,0,0,           ← home
0,0,1.22,0,0,0,        ← rise to transit height
0,0,1.22,0,3,6,        ← navigate to start
0,0,1.22,0,3,7,        ← START
[scan serpentine across all levels]
0,0,[z],0,3,8,         ← END
0,0,[z],0,3,6,
0,0,3.64,0,0,6,        ← descend step 1
0,0,1.22,0,0,6,        ← descend step 2
0,0,-1.5,0,0,0,        ← land
```

**Location code:** `G[aisle][bay_code][level][slot]` e.g. `G0622210|G0621220`

**File naming:** `[CustomerPrefix][aisle][spec]_[variant].mission`
Example: `YBMWG06_60LO_MAIN.mission` = Yusen, BMW site, Aisle G06, 60-location scan, main variant

### Avilon Indoor Test

```
0,0,1,0,1,0,           ← takeoff (type=1, not type=0)
0,0,1.3,0,3,6,
0,0,1.3,0,3,7,         ← START
[scan with mid-aisle turn-back segments]
0,0,-1.5,0,1,0,        ← land (type=1)
```

**Location code:** `B2U-01-02|B2V-01-02`

---

## TAG_*.json File

```json
{
  "Standalone": {
    "tag_dictionary_index": 20,
    "tag_length": 0.16,
    "tags": []
  },
  "MainQuad": {
    "tag_dictionary_index": 20,
    "tag_length": 0.3,
    "tag_indiv_size": 0.06,
    "tags": [
      [tag_id, x, y, z, 1.57, 0.0, -3.14]
    ]
  }
}
```

### Sections

| Section | Tag size | Use |
|---|---|---|
| `Standalone` | 0.16m | Solo drone — often empty `[]`; DHL uses tag id=18 as origin marker |
| `MainQuad` | 0.3m outer / 0.06m individual | Main quad drone |

### Tag array fields

`[tag_id, x, y, z, roll, pitch, yaw]`

| field | value | meaning |
|---|---|---|
| `tag_id` | integer | unique ArUco marker ID |
| `x` | float | position along aisle (meters from origin) |
| `y` | `-(aisle_width / 2)` | offset to left wall; always negative |
| `z` | float | tag center height above floor |
| `roll` | `1.57` | always — tag mounted vertically on wall |
| `pitch` | `0.0` | always |
| `yaw` | `-3.14` | always — tag faces into aisle toward drone |

### Tag ID numbering (DHL pattern)

| range | tier |
|---|---|
| 300–399 | low height |
| 400–499 | mid height |
| 500–599 | high height |

### Key formula

```
y = -(aisle_width / 2)
```

Example: aisle width 1.832m → y = -(1.832 / 2) = -0.916

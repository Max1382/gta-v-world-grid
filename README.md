# GTA V World Grid Dataset

A large JSONL dataset containing sampled world/map information from **Grand Theft Auto V**.

The dataset scans the GTA V world using a regular coordinate grid and records information returned by the game for each location, including zones, streets, map areas, interiors, ground height, water height and various game hashes.

## Dataset Overview

Current dataset:

* **10,836 map samples**
* **100-unit grid spacing**
* X range: `-4000` to `4500`
* Y range: `-4500` to `8000`
* Z sampling position: `0`
* **87 unique GTA V zones**
* Stored as **JSON Lines (`.jsonl`)**
* One map sample per line

The dataset was generated using an automatic world-grid scan.

## File

```text
world-grid-20260918-212343.jsonl
```

JSONL means that every line is a separate JSON object rather than the entire file being one large JSON array.

This makes the dataset easier to stream, search and process without loading the full file into memory.

## Example Record

```json
{
  "timestamp": "2026-09-18T21:23:43.327",
  "position": {
    "x": -4000.0,
    "y": -4500.0,
    "z": 0.0
  },
  "properties": {
    "timestamp": "2026-09-18T21:23:43.327",
    "reason": "automatic_grid_cell",
    "sample_number": 1,
    "mapping_mode": "automatic_grid",
    "heading": 0.0,
    "zone_code": "SanAnd",
    "zone_localized": "San Andreas",
    "street_hash_signed": -172004387,
    "street_hash_unsigned": 4122962909,
    "street_name": "",
    "crossing_hash_signed": 0,
    "crossing_hash_unsigned": 0,
    "crossing_name": "",
    "interior_id": 0,
    "interior_valid": false,
    "interior_group_id": 0,
    "room_key_signed": 0,
    "room_key_unsigned": 0,
    "map_area_hash_signed": -289320599,
    "map_area_hash_unsigned": 4005646697,
    "ground_z_resolved": false,
    "ground_z": 0.0,
    "water_height_resolved": true,
    "water_height": 0.0,
    "in_vehicle": false,
    "vehicle_model_signed": 0,
    "vehicle_model_unsigned": 0
  }
}
```

## Data Fields

### Position

| Field | Description                     |
| ----- | ------------------------------- |
| `x`   | GTA world X coordinate          |
| `y`   | GTA world Y coordinate          |
| `z`   | Z coordinate used when sampling |

### General Properties

| Field           | Description                     |
| --------------- | ------------------------------- |
| `timestamp`     | Time the sample was recorded    |
| `reason`        | Reason the location was sampled |
| `sample_number` | Sequential sample number        |
| `mapping_mode`  | Mapping/scanning method         |
| `heading`       | Heading at the sampled position |

### Zone Information

| Field            | Description                  |
| ---------------- | ---------------------------- |
| `zone_code`      | Internal GTA zone code       |
| `zone_localized` | Human-readable GTA zone name |

Examples include:

```text
OCEANA  -> Pacific Ocean
MTCHIL  -> Mount Chiliad
CHIL    -> Vinewood Hills
DESRT   -> Grand Senora Desert
AIRP    -> Los Santos International Airport
ALAMO   -> Alamo Sea
ARMYB   -> Fort Zancudo
```

The dataset currently contains **87 unique zone values**.

## Street Information

Each sample may contain GTA street information.

```text
street_hash_signed
street_hash_unsigned
street_name
```

Both signed and unsigned versions of hashes are included to make the dataset easier to use with different GTA/FiveM/native implementations.

Crossing/nearest intersection information is also included:

```text
crossing_hash_signed
crossing_hash_unsigned
crossing_name
```

Street information is not available for every coordinate, particularly locations in the ocean, mountains and other areas without roads.

## Interior Information

The dataset contains:

```text
interior_id
interior_valid
interior_group_id
room_key_signed
room_key_unsigned
```

`interior_valid` can be used to determine whether GTA reported a valid interior for the sampled coordinate.

## Map Area Hashes

Each sample includes:

```text
map_area_hash_signed
map_area_hash_unsigned
```

Both signed and unsigned representations are stored.

These values may be useful when researching how GTA categorises sections of the world internally.

## Ground Data

Ground detection fields:

```text
ground_z_resolved
ground_z
```

`ground_z_resolved` indicates whether a valid ground height was successfully returned.

When available, `ground_z` contains the detected height.

## Water Data

Water detection fields:

```text
water_height_resolved
water_height
```

These can be used to identify coordinates associated with GTA V's water system and determine the returned water height.

## Vehicle Data

The scanner also records:

```text
in_vehicle
vehicle_model_signed
vehicle_model_unsigned
```

The current automatic grid scan was not performed inside a vehicle, so these values will normally be `false` / `0`.

They are retained as part of the mapping format so other types of samples can use the same structure.

## Grid Coverage

The current scan covers approximately:

```text
X: -4000 -> 4500
Y: -4500 -> 8000
```

Coordinates are sampled every:

```text
100 GTA units
```

There are:

```text
86 unique X positions
126 unique Y positions
10,836 total samples
```

This includes large areas outside the main landmass so information about oceans and world boundaries is also present.

## Example Usage

### Python

```python
import json

with open("world-grid-20260918-212343.jsonl", "r", encoding="utf-8") as file:
    for line in file:
        point = json.loads(line)

        x = point["position"]["x"]
        y = point["position"]["y"]

        zone = point["properties"]["zone_localized"]

        print(x, y, zone)
```

### Find Every Sample in Mount Chiliad

```python
import json

with open("world-grid-20260918-212343.jsonl", "r", encoding="utf-8") as file:
    for line in file:
        point = json.loads(line)

        if point["properties"]["zone_code"] == "MTCHIL":
            print(point["position"])
```

### Find Samples With Street Names

```python
import json

with open("world-grid-20260918-212343.jsonl", "r", encoding="utf-8") as file:
    for line in file:
        point = json.loads(line)

        street = point["properties"]["street_name"]

        if street:
            print(
                point["position"]["x"],
                point["position"]["y"],
                street
            )
```

## Possible Uses

This dataset may be useful for:

* GTA V map research
* FiveM development
* GTA modding tools
* Coordinate lookup systems
* Zone detection
* Street lookup databases
* Map visualisation
* Interior research
* GTA hash research
* Water/ground mapping
* Converting coordinates into readable locations
* Building GTA V map APIs
* Creating location databases
* Comparing GTA native results across different parts of the map

## Accuracy

This dataset represents information returned by GTA V at the sampled coordinates.

Because the world is sampled on a **100-unit grid**, it should not be treated as a perfect representation of every possible coordinate.

Zone boundaries, streets, interiors, terrain and water can change between two sampled grid points.

For applications requiring exact information, the dataset can be used as a reference or starting point and supplemented with higher-resolution sampling.

## Format

The dataset uses **JSONL / NDJSON** rather than a standard JSON array.

Correct:

```text
{"position":{"x":0,"y":0,"z":0},"properties":{...}}
{"position":{"x":100,"y":0,"z":0},"properties":{...}}
{"position":{"x":200,"y":0,"z":0},"properties":{...}}
```

Instead of:

```json
[
  {},
  {},
  {}
]
```

This allows the file to be processed one location at a time.

## Contributions

Contributions, additional scans, corrections and tools for working with the dataset are welcome.

Useful contributions could include:

* Higher-resolution scans
* Additional Z-level scans
* Interior mapping
* Visual map generation
* CSV/SQLite conversions
* Zone boundary extraction
* Street mapping
* Data validation
* GTA/FiveM lookup tools

## Disclaimer

This project is an unofficial community resource and is not affiliated with or endorsed

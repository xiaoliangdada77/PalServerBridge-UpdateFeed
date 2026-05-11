# World Spawn Pal Guide

## type: "spawn_pal"

Spawns a wild pal actor at the specified world coordinates. The spawned pal is not owned by any player.
You can reference a `pal/` template for pal attributes and override fields inline.

## Format

```json
{
    "type": "spawn_pal",
    "pal_template": "pal/Debug_SuperAnubis",
    "Location": {"X": 1000, "Y": 2000, "Z": 500},
    "Rotation": {"Pitch": 0, "Yaw": 90, "Roll": 0},
    "AdjustToFloor": true,
    "count": 1,
    "SaveParameter": {
        "CharacterID": "Kirin",
        "Level": 50
    }
}
```

## Field Descriptions

| Field | Type | Description |
|------|------|------|
| **type** | string | Fixed to `"spawn_pal"` |
| pal_template | string | Pal template path, for example `"pal/Debug_SuperAnubis"` |
| **Location** | object | **Required** spawn coordinates |
| Location.X | float | X coordinate |
| Location.Y | float | Y coordinate |
| Location.Z | float | Z coordinate |
| Rotation | object | Rotation, defaults to all zero |
| Rotation.Pitch | float | Pitch |
| Rotation.Yaw | float | Yaw |
| Rotation.Roll | float | Roll |
| AdjustToFloor | bool | Snap to floor, default `true` |
| count | int | Spawn count, default `1` |
| SaveParameter | object | Pal save parameters. Required when not using a template |
| SaveParameter.CharacterID | string | Pal ID. Required when not using a template |

## Notes

- Provide at least one of `pal_template` or `SaveParameter.CharacterID`
- Inline fields inside `SaveParameter` override template values
- All pal attribute fields such as Level, Rank, and PassiveSkillList should be placed inside `SaveParameter`
- `CharacterID` is not accepted at the `data` root; root-level usage returns `Missing SaveParameter.CharacterID`
- Spawned pals are wild actors and are not owned by any player
- `spawn_pal` is a world-spawn action and does not require `steamid`; it can spawn by coordinates even when no players are online

## Examples

Template reference:
```json
{"key": "xxx", "data": {
    "type": "spawn_pal",
    "pal_template": "pal/Debug_SuperAnubis",
    "Location": {"X": 0, "Y": 0, "Z": 500}
}}
```

Inline only:
```json
{"key": "xxx", "data": {
    "type": "spawn_pal",
    "SaveParameter": {
        "CharacterID": "Kirin",
        "Level": 50,
        "IsRarePal": true
    },
    "Location": {"X": 1000, "Y": 2000, "Z": 500},
    "count": 5
}}
```

---
*Generated automatically by PalServerBridge. Do not edit manually.*

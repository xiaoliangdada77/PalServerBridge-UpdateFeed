# Gift Template Guide

## Directory: templates/gift/

Gift templates contain multiple items and pals and grant them in order. This is a second-layer template and may reference other templates.

## Format

```json
{
    "type": "gift",
    "items": [
        {"template": "pal/max_anubis"},
        {"template": "item/legend_sphere"},
        {"type": "pal", "CharacterID": "Kirin", "Level": 30, "count": 2},
        {"type": "item", "ItemId": "Gold_Coin", "Count": 1000}
    ]
}
```

## Notes

- Each entry in `items` can be:
- **Template reference**: `{"template": "pal/xxx"}` loads from a file
- **Inline definition**: `{"type": "pal", "CharacterID": "xxx", ...}` defines the reward directly
- **Template plus overrides**: `{"template": "pal/xxx", "Level": 30}` loads and overrides selected fields
- Gifts may reference other gift or lottery templates
- Circular references are detected and blocked automatically

---
*Generated automatically by PalServerBridge. Do not edit manually.*

# Lottery Template Guide

## Directory: templates/lottery/

Lottery templates contain multiple rewards and randomly choose one by weight. This is a second-layer template and may reference other templates.

## Format

```json
{
    "type": "lottery",
    "items": [
        {"template": "pal/max_anubis", "Weight": 100},
        {"type": "item", "ItemId": "Gold_Coin", "Count": 500, "Weight": 1000},
        {"template": "gift/starter_pack", "Weight": 50}
    ]
}
```

## Field Descriptions

| Field | Type | Description |
|------|------|------|
| **type** | string | Fixed to `"lottery"` |
| **items** | array | Reward list |
| items[].Weight | int | Weight, default `1000`; higher means more likely |

## Probability

Probability = item Weight / total Weight of all items

Example: Weight 100 + 1000 + 50 = 1150
- Maxed Anubis: 100/1150 = 8.7%
- 500 gold coins: 1000/1150 = 87.0%
- Starter pack: 50/1150 = 4.3%

## Notes

- Each item may be a template reference or inline definition
- Lotteries may reference gifts or other lotteries
- Circular references are detected and blocked automatically

---
*Generated automatically by PalServerBridge. Do not edit manually.*

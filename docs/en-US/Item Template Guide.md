# Item Template Guide

## Directory: templates/item/

Item templates define one item and its quantity. This is a first-layer template and must not reference other templates.

## Format

```json
{
    "type": "item",
    "ItemId": "SphereItem_Pal_Legend",
    "Count": 10
}
```

## Field Descriptions

| Field | Type | Description |
|------|------|------|
| **type** | string | Fixed to `"item"` |
| **ItemId** | string | **Required** item code |
| Count | int | Amount, default `1` |

## Common Item Codes

| Code | Name |
|------|------|
| SphereItem_Pal_Legend | Legendary Sphere |
| Gold_Coin | Gold Coin |
| Money | Currency |

---
*Generated automatically by PalServerBridge. Do not edit manually.*

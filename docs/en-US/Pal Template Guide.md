# Pal Template Guide

## Directory: templates/pal/

Pal templates define the full attributes of a single pal. This is a first-layer template and must not reference other templates.

## Format

```json
{
    "type": "pal",
    "count": 1,
    "SaveParameter": {
        "CharacterID": "Anubis",
        "Level": 50,
        "IsRarePal": true,
        "Rank": 4,
        "PassiveSkillList": ["Legend", "Rare"],
        "EquipWaza": ["SandTornado"],
        "NickName": "MyPal"
    },
    "StaticParameter": {
        "CaptureSuccessRate": 0.5
    }
}
```

## Top-level Fields

| Field | Type | Description |
|------|------|------|
| **type** | string | Fixed to `"pal"` |
| count | int | Amount, default `1` |

## SaveParameter Fields

| Field | Type | Description |
|------|------|------|
| **CharacterID** | string | **Required** Pal ID (for example "Anubis") |
| Level | int | Level |
| Gender | int | Gender: 0 male, 1 female |
| IsRarePal | bool | Lucky / rare pal |
| Rank | int | Condense rank |
| Rank_HP | int | Soul boost - HP |
| Rank_Attack | int | Soul boost - attack |
| Rank_Defence | int | Soul boost - defense |
| Rank_CraftSpeed | int | Soul boost - work speed |
| Talent_HP | int | IV - HP |
| Talent_Melee | int | IV - melee attack |
| Talent_Shot | int | IV - ranged attack |
| Talent_Defense | int | IV - defense |
| Exp | int64 | Experience |
| RankUpExp | int | Condense experience |
| NickName | string | Nickname |
| PassiveSkillList | string[] | Passive skills |
| EquipWaza | string[] | Equipped active skills |
| MasteredWaza | string[] | Learned skills |
| HP | number | Current HP |
| MaxHP | number | Max HP |
| MP | number | Current MP |
| MaxMP | number | Max MP |
| SP | number | Max stamina |
| FullStomach | float | Current fullness |
| MaxFullStomach | float | Max fullness |
| SanityValue | float | Sanity |
| Support | int | Support level |
| CraftSpeed | int | Work speed |
| PhysicalHealth | int | Physical condition |
| WorkerSick | int | Sickness state |
| HungerType | int | Hunger type |
| VoiceID | int | Voice ID |
| IsFavoritePal | bool | Favorite flag |
| FavoriteIndex | int | Favorite order |
| UniqueNPCID | string | Unique NPC ID |
| SkinName | string | Skin name |
| ShieldHP | number | Shield HP |
| ShieldMaxHP | number | Max shield HP |
| FullStomachDecreaseRate_Tribe | float | Fullness drain rate |
| BaseCampWorkerEventProgressTime | float | Base event progress |
| PalReviveTimer | float | Revive timer |
| DyingTimer | int | Downed timer |
| FriendshipPoint | int | Friendship points |
| FriendshipOtomoSec | int | Companion friendship time |
| FriendshipActiveOtomoSec | int | Active companion friendship time |
| FriendshipBasecampSec | int | Base friendship time |
| ArenaRankPoint | int | Arena points |
| UnusedStatusPoint | int | Unused status points |
| BaseCampWorkerEventType | int | Base event type |
| CurrentWorkSuitability | int | Current work suitability |
| bImportedCharacter | bool | Imported character |
| bApplyShieldDamage | bool | Apply shield damage |
| bAppliedDeathPenarty | bool | Death penalty applied |
| bEnablePlayerRespawnInHardcore | bool | Hardcore respawn enabled |
| bFavoriteChangedByFriendship | bool | Favorite changed by friendship |
| bDisableSaleInPalLost | bool | Disable sale when pal lost |
| FoodWithStatusEffect | string | Food with status effect |
| ExtraWorkSuitabilities | object | Work suitability overrides |

## StaticParameter Fields (optional, only used by spawn_pal)

| Field | Type | Description |
|------|------|------|
| IsPal | bool | Whether this is a pal |
| IsBoss_Database | bool | Whether marked as boss |
| IsTowerBoss_Database | bool | Whether tower boss |
| IsRaidBoss_Database | bool | Whether raid boss |
| IsPredatorBoss_Database | bool | Whether predator boss |
| IsLegend_Database | bool | Whether legendary |
| IsRaidBoss_BP | bool | Whether raid boss in BP |
| IsUncapturable | bool | Uncapturable |
| CaptureSuccessRate | float | Capture success rate |
| Weight_KG | float | Weight (kg) |
| SkillEffectScale | float | Skill effect scale |
| Size | string | Body size |
| MovementType | string | Movement type |
| SpawnedCharacterType | string | Spawned character type |

---
*Generated automatically by PalServerBridge. Do not edit manually.*

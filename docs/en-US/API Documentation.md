# PalServerBridge API Documentation

## Authentication

All `POST /api/` endpoints require a `key` field inside the request body.
Configure the key with `api_key` in `config.json`.
Authentication is enforced centrally by the `set_pre_routing_handler` middleware, so individual routes do not repeat the same validation.

## Common Response Format

Success: `{"status": "success", "message": "...", "details": {...}}`
Failure: `{"status": "error", "message": "..."}`

Click a path in the endpoint table below to jump to its details.

---

## Endpoints (22)

| Method | Path | Summary |
|------|------|------|
| GET | [/ping](#endpoint-get-ping) | Health check |
| POST | [/api/players](#endpoint-post-api-players) | List online players |
| POST | [/api/capabilities](#endpoint-post-api-capabilities) | Get plugin capabilities |
| POST | [/api/guilds](#endpoint-post-api-guilds) | List guilds |
| POST | [/api/player_pals](#endpoint-post-api-player-pals) | Get player pals |
| POST | [/api/player_items](#endpoint-post-api-player-items) | Get player items |
| POST | [/api/execute](#endpoint-post-api-execute) | Unified execution endpoint |
| POST | [/api/delete_item](#endpoint-post-api-delete-item) | Delete player items |
| POST | [/api/delete_pal](#endpoint-post-api-delete-pal) | Delete player pals |
| POST | [/api/get_relic](#endpoint-post-api-get-relic) | Give Lifmunk effigies |
| POST | [/api/get_pal_egg](#endpoint-post-api-get-pal-egg) | Give pal egg |
| POST | [/api/add_tech_points](#endpoint-post-api-add-tech-points) | Add technology points |
| POST | [/api/add_boss_tech_points](#endpoint-post-api-add-boss-tech-points) | Add ancient technology points |
| POST | [/api/add_player_exp](#endpoint-post-api-add-player-exp) | Add player EXP |
| POST | [/api/add_party_exp](#endpoint-post-api-add-party-exp) | Add party EXP |
| POST | [/api/add_all_player_exp](#endpoint-post-api-add-all-player-exp) | Add EXP to all players |
| POST | [/api/kick_player](#endpoint-post-api-kick-player) | Kick player |
| POST | [/api/chat](#endpoint-post-api-chat) | Public chat |
| POST | [/api/private_message](#endpoint-post-api-private-message) | Private message |
| POST | [/api/announce](#endpoint-post-api-announce) | System announcement |
| POST | [/api/resources](#endpoint-post-api-resources) | Export all resources |
| POST | [/api/field_definitions](#endpoint-post-api-field-definitions) | Get configurable field definitions |

---

<a id="endpoint-get-ping"></a>

## GET /ping

**Health check**

Returns pong to verify that the service is alive. Authentication is not required.

Response example:
```json
pong
```

---

<a id="endpoint-post-api-players"></a>

## POST /api/players

**List online players**

Returns all online players, including names, UIDs, locations, and rotations.

GUID fields:
- `player_uid`: preserves Palworld `GetPlayerList`'s original dashless format, for example `DF11EBAA000000000000000000000000`, for backward compatibility
- `player_uid_dashed`: normalized `8-8-8-8` format, for example `DF11EBAA-00000000-00000000-00000000`, recommended for matching engine-side `FGuid::ToString()` values or GUIDs emitted by Lua events

Request example:
```json
{
  "key": "xxx"
}
```

Response example:
```json
{
  "players": [
    {
      "location": {
        "X": 0,
        "Y": 0,
        "Z": 0
      },
      "nick_name": "xxx",
      "platform": 0,
      "player_name": "xxx",
      "player_uid": "DF11EBAA000000000000000000000000",
      "player_uid_dashed": "DF11EBAA-00000000-00000000-00000000",
      "rotation": {
        "Pitch": 0,
        "Roll": 0,
        "Yaw": 0
      },
      "user_id": "steam_xxx"
    }
  ],
  "status": "success"
}
```

---

<a id="endpoint-post-api-capabilities"></a>

## POST /api/capabilities

**Get plugin capabilities**

Returns supported capability flags for safe feature gating by upper layers.

Request example:
```json
{
  "key": "xxx"
}
```

Response example:
```json
{
  "capabilities": {
    "precise_pal_delete_slot": true
  },
  "status": "success"
}
```

---

<a id="endpoint-post-api-guilds"></a>

## POST /api/guilds

**List guilds**

Returns guilds, members, and base camp locations for interactive maps and server management pages.

Request example:
```json
{
  "key": "xxx"
}
```

Response example:
```json
{
  "guilds": [
    {
      "base_camp_count": 1,
      "base_camps": [],
      "guild_id": "xxx",
      "guild_name": "xxx",
      "member_count": 1,
      "members": [],
      "online_member_count": 1
    }
  ],
  "status": "success"
}
```

---

<a id="endpoint-post-api-player-pals"></a>

## POST /api/player_pals

**Get player pals**

Gets the specified player's pal data (storage and party), including full attributes such as SaveParameter.

JSON parameter: `steamid` (required)

Request example:
```json
{
  "key": "xxx",
  "steamid": "steam_xxx"
}
```

Response example:
```json
{"status":"success","data":{"storage":[...],"party":[...]}}
```

---

<a id="endpoint-post-api-player-items"></a>

## POST /api/player_items

**Get player items**

Gets all item data for the specified player, including SlotIndex, ItemId, StackCount, Permission, and related fields.

JSON parameter: `steamid` (required)

Request example:
```json
{
  "key": "xxx",
  "steamid": "steam_xxx"
}
```

Response example:
```json
{
  "data": [
    {
      "CorruptionProgressValue": 0,
      "ItemId": "xxx",
      "Permission": {},
      "SlotIndex": 0,
      "StackCount": 1
    }
  ],
  "status": "success"
}
```

---

<a id="endpoint-post-api-execute"></a>

## POST /api/execute

**Unified execution endpoint**

Dispatches to the correct action by the `type` field, such as giving items, giving pals, or spawning wild pals. Template references are supported.

- `steamid`: target player SteamID; required for player-context actions, not required for `spawn_pal`
- `type`: operation type, such as `item`, `pal`, or `spawn_pal` (required unless using template)
- `template`: template path, such as `pal/Anubis` or `spawn_pal/Debug_GYM_ElecPanda_Otomo` (optional, replaces type+data)
- `data`: operation parameters, depending on type
- `count`: amount, default `1`

Note: `spawn_pal` spawns by world coordinates and is not bound to a player, so `steamid` can be omitted.

Request example:
```json
{
  "count": 1,
  "data": {
    "CharacterID": "Anubis",
    "IsRarePal": false,
    "Level": 50,
    "Rank": 4,
    "SaveParameter": {
      "NickName": "Test"
    }
  },
  "key": "xxx",
  "steamid": "xxx",
  "type": "pal"
}
```

Response example:
```json
{
  "details": {},
  "message": "Operation succeeded",
  "status": "success"
}
```

---

<a id="endpoint-post-api-delete-item"></a>

## POST /api/delete_item

**Delete player items**

Deletes items from the specified player. The operation is rejected when the player does not have enough items.

- `steamid`: target player SteamID (required)
- `item_code`: item code, such as `PalSphere` (required)
- `amount`: amount to delete, default `1`

Note: this endpoint uses the player inventory data path rather than directly mutating an online Actor. Whether offline writes reach the save data depends on whether the server can resolve the target inventory object from `player_uid`.

Request example:
```json
{
  "amount": 5,
  "item_code": "PalSphere",
  "key": "xxx",
  "steamid": "xxx"
}
```

Response example:
```json
{
  "deleted": 5,
  "status": "success",
  "success": true
}
```

---

<a id="endpoint-post-api-delete-pal"></a>

## POST /api/delete_pal

**Delete player pals**

Filters and deletes pals by SaveParameter. The operation is rejected when not enough matching pals are found.

- `steamid`: target player SteamID (required)
- `SaveParameter`: filter object; `CharacterID` is required, Level/Gender/etc. are optional
- `count`: amount to delete, default `1`
- `from_party`: delete from party, default false (storage)
- `slot_index`: optional; when provided, precise slot deletion is tried first

Request example:
```json
{
  "SaveParameter": {
    "CharacterID": "Anubis"
  },
  "count": 1,
  "from_party": false,
  "key": "xxx",
  "slot_index": 12,
  "steamid": "xxx"
}
```

Response example:
```json
{
  "deleted": 1,
  "status": "success",
  "success": true
}
```

---

<a id="endpoint-post-api-get-relic"></a>

## POST /api/get_relic

**Give Lifmunk effigies**

Directly modifies the player's RelicPossessNum field to add Lifmunk effigies.

- `steamid`: target player SteamID (required)
- `count`: amount, default `1`

Request example:
```json
{
  "count": 3,
  "key": "xxx",
  "steamid": "xxx"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-get-pal-egg"></a>

## POST /api/get_pal_egg

**Give pal egg**

Gives an egg for the specified pal. Note: directly granted eggs may not hatch correctly.

- `steamid`: target player SteamID (required)
- `character_id`: pal ID, such as `Anubis` (required)

Request example:
```json
{
  "character_id": "Anubis",
  "key": "xxx",
  "steamid": "xxx"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-add-tech-points"></a>

## POST /api/add_tech_points

**Add technology points**

Directly modifies the player's TechnologyPoint field.

- `steamid`: target player SteamID (required)
- `points`: technology points, default `1`

Request example:
```json
{
  "key": "xxx",
  "points": 10,
  "steamid": "xxx"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-add-boss-tech-points"></a>

## POST /api/add_boss_tech_points

**Add ancient technology points**

Directly modifies the player's bossTechnologyPoint field.

- `steamid`: target player SteamID (required)
- `points`: ancient technology points, default `1`

Request example:
```json
{
  "key": "xxx",
  "points": 5,
  "steamid": "xxx"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-add-player-exp"></a>

## POST /api/add_player_exp

**Add player EXP**

Adds EXP to the specified player. Two modes are supported:
- Default: directly modifies the Exp field (safer, but does not trigger level-up logic)
- `use_debug_method=true`: uses the debug method (levels up automatically, but may be blocked by anti-cheat)

Parameters:
- `steamid`: target player SteamID (required)
- `exp`: EXP amount, default `100`
- `use_debug_method`: use debug method, default false

Request example:
```json
{
  "exp": 1000,
  "key": "xxx",
  "steamid": "xxx",
  "use_debug_method": false
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-add-party-exp"></a>

## POST /api/add_party_exp

**Add party EXP**

Adds EXP to every pal in the specified player's party.

- `steamid`: target player SteamID (required)
- `exp`: EXP amount, default `100`
- `use_debug_method`: use debug method, default false

Request example:
```json
{
  "exp": 500,
  "key": "xxx",
  "steamid": "xxx",
  "use_debug_method": false
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-add-all-player-exp"></a>

## POST /api/add_all_player_exp

**Add EXP to all players**

Adds EXP to all currently online players.

- `exp`: EXP amount, default `100`
- `use_debug_method`: use debug method, default false

Request example:
```json
{
  "exp": 1000,
  "key": "xxx",
  "use_debug_method": false
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-kick-player"></a>

## POST /api/kick_player

**Kick player**

Kicks the specified player from the server.

- `steamid`: target player SteamID (required)
- `reason`: kick reason, default "Kicked by admin"

Request example:
```json
{
  "key": "xxx",
  "reason": "Rule violation",
  "steamid": "xxx"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-chat"></a>

## POST /api/chat

**Public chat**

Sends a public chat message as the specified sender.

- `message`: message text (required)
- `category`: category 1=global 2=guild 3=nearby, default `1`
- `sender`: sender name, default "Server"

Request example:
```json
{
  "category": 1,
  "key": "xxx",
  "message": "Hello World",
  "sender": "Server"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-private-message"></a>

## POST /api/private_message

**Private message**

Sends a private message to the specified player.

- `steamid`: target player SteamID (required)
- `message`: message text (required)

Request example:
```json
{
  "key": "xxx",
  "message": "Hello",
  "steamid": "xxx"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-announce"></a>

## POST /api/announce

**System announcement**

Sends a system announcement in invasion-alert style to all online players.

- `message`: announcement text (required)

Request example:
```json
{
  "key": "xxx",
  "message": "The server will restart in 10 minutes"
}
```

Response example:
```json
{
  "status": "success"
}
```

---

<a id="endpoint-post-api-resources"></a>

## POST /api/resources

**Export all resources**

Exports documentation and template resources. Supports viewing current-language docs or exporting all language docs at once.

Request example:
```json
{
  "key": "xxx"
}
```

Response example:
```json
{"status":"success","timestamp":"...","resources":{"docs":[...],"templates":[...]},"summary":{...}}
```

---

<a id="endpoint-post-api-field-definitions"></a>

## POST /api/field_definitions

**Get configurable field definitions**

Returns all configurable SaveParameter and StaticParameter fields, including field names, types, and required flags.

Request example:
```json
{
  "key": "xxx"
}
```

Response example:
```json
{"status":"success","SaveParameter":[{"key":"CharacterID","type":"string","required":true},...]}
```

---

*Generated automatically by PalServerBridge. Do not edit manually.*

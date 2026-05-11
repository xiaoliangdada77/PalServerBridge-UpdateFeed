# PalServerBridge API 接口文档

## 认证

所有 `POST /api/` 接口需要在请求体中包含 `key` 字段。
在 `config.json` 的 `api_key` 中配置密钥。
认证通过 `set_pre_routing_handler` 中间件统一拦截，路由内无需重复验证。

## 通用响应格式

成功: `{"status": "success", "message": "...", "details": {...}}`
失败: `{"status": "error", "message": "..."}`

---

## 接口列表 (22 个)

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/ping` | 健康检查 |
| POST | `/api/players` | 获取在线玩家列表 |
| POST | `/api/capabilities` | 获取插件能力标记 |
| POST | `/api/guilds` | 获取公会列表 |
| POST | `/api/player_pals` | 获取玩家帕鲁数据 |
| POST | `/api/player_items` | 获取玩家物品数据 |
| POST | `/api/execute` | 统一执行入口 |
| POST | `/api/delete_item` | 删除玩家物品 |
| POST | `/api/delete_pal` | 删除玩家帕鲁 |
| POST | `/api/get_relic` | 给予翠叶鼠雕像 |
| POST | `/api/get_pal_egg` | 给予帕鲁蛋 |
| POST | `/api/add_tech_points` | 给予科技点 |
| POST | `/api/add_boss_tech_points` | 给予古代科技点 |
| POST | `/api/add_player_exp` | 给予玩家经验 |
| POST | `/api/add_party_exp` | 给予队伍经验 |
| POST | `/api/add_all_player_exp` | 给予所有玩家经验 |
| POST | `/api/kick_player` | 踢出玩家 |
| POST | `/api/chat` | 公屏聊天 |
| POST | `/api/private_message` | 私聊消息 |
| POST | `/api/announce` | 系统公告 |
| POST | `/api/resources` | 导出所有资源 |
| POST | `/api/field_definitions` | 获取可配置字段定义 |

---

## GET /ping

**健康检查**

返回 pong，用于检测服务是否存活。无需认证。

响应示例:
```json
pong
```

---

## POST /api/players

**获取在线玩家列表**

返回当前在线的所有玩家信息，包括名称、UID、位置、朝向等。

GUID 字段:
- `player_uid`: 保留 Palworld `GetPlayerList` 的原始无短横线格式，例如 `DF11EBAA000000000000000000000000`，用于向后兼容
- `player_uid_dashed`: 规范化的 `8-8-8-8` 格式，例如 `DF11EBAA-00000000-00000000-00000000`，推荐用于和引擎侧 `FGuid::ToString()` 或 Lua 事件 GUID 匹配

请求示例:
```json
{
  "key": "xxx"
}
```

响应示例:
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

## POST /api/capabilities

**获取插件能力标记**

返回当前 DLL 插件已支持的能力开关，供上层做安全特性门控。

请求示例:
```json
{
  "key": "xxx"
}
```

响应示例:
```json
{
  "capabilities": {
    "precise_pal_delete_slot": true
  },
  "status": "success"
}
```

---

## POST /api/guilds

**获取公会列表**

返回公会列表、成员信息与据点位置，用于互动地图和服务器管理页展示。

请求示例:
```json
{
  "key": "xxx"
}
```

响应示例:
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

## POST /api/player_pals

**获取玩家帕鲁数据**

获取指定玩家的帕鲁数据（仓库+队伍），包括 SaveParameter 等完整属性。

JSON参数: `steamid` (必填)

请求示例:
```json
{
  "key": "xxx",
  "steamid": "steam_xxx"
}
```

响应示例:
```json
{"status":"success","data":{"storage":[...],"party":[...]}}
```

---

## POST /api/player_items

**获取玩家物品数据**

获取指定玩家的所有物品数据，包括 SlotIndex、ItemId、StackCount、Permission 等。

JSON参数: `steamid` (必填)

请求示例:
```json
{
  "key": "xxx",
  "steamid": "steam_xxx"
}
```

响应示例:
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

## POST /api/execute

**统一执行入口**

根据 type 字段自动分发到对应功能（生成帕鲁、给予物品等）。支持模板引用。

- `steamid`: 目标玩家 SteamID；玩家上下文动作必填，`spawn_pal` 不需要
- `type`: 操作类型，如 `item`、`pal`、`spawn_pal` (必填，或使用 template)
- `template`: 模板路径，如 `pal/Anubis` 或 `spawn_pal/Debug_GYM_ElecPanda_Otomo` (可选，替代 type+data)
- `data`: 操作参数 (根据 type 不同而不同)
- `count`: 数量 (默认 1)

说明: `spawn_pal` 是世界坐标生成，不绑定玩家，因此可省略 `steamid`。

请求示例:
```json
{
  "count": 1,
  "data": {
    "CharacterID": "Anubis",
    "IsRarePal": false,
    "Level": 50,
    "Rank": 4,
    "SaveParameter": {
      "NickName": "测试"
    }
  },
  "key": "xxx",
  "steamid": "xxx",
  "type": "pal"
}
```

响应示例:
```json
{
  "details": {},
  "message": "操作成功",
  "status": "success"
}
```

---

## POST /api/delete_item

**删除玩家物品**

删除指定玩家的物品，数量不足时拒绝执行。

- `steamid`: 目标玩家 SteamID (必填)
- `item_code`: 物品代码，如 `PalSphere` (必填)
- `amount`: 删除数量 (默认 1)

说明: 这个接口走的是玩家库存数据路径，而不是直接改在线 Actor。离线写入是否会落到存档，取决于服务端能否通过 `player_uid` 解析到对应的库存对象。

请求示例:
```json
{
  "amount": 5,
  "item_code": "PalSphere",
  "key": "xxx",
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "deleted": 5,
  "status": "success",
  "success": true
}
```

---

## POST /api/delete_pal

**删除玩家帕鲁**

按 SaveParameter 条件筛选并删除帕鲁，数量不足时拒绝执行。

- `steamid`: 目标玩家 SteamID (必填)
- `SaveParameter`: 筛选条件，`CharacterID` 必填，可选 Level、Gender 等
- `count`: 删除数量 (默认 1)
- `from_party`: 是否从队伍中删除 (默认 false，从仓库删除)
- `slot_index`: 可选，指定槽位后优先按槽位精确删除

请求示例:
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

响应示例:
```json
{
  "deleted": 1,
  "status": "success",
  "success": true
}
```

---

## POST /api/get_relic

**给予翠叶鼠雕像**

直接修改玩家的 RelicPossessNum 字段，增加指定数量的翠叶鼠雕像。

- `steamid`: 目标玩家 SteamID (必填)
- `count`: 数量 (默认 1)

请求示例:
```json
{
  "count": 3,
  "key": "xxx",
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/get_pal_egg

**给予帕鲁蛋**

给予指定帕鲁的蛋（注意：直接给予的蛋可能无法正常孵化）。

- `steamid`: 目标玩家 SteamID (必填)
- `character_id`: 帕鲁 ID，如 `Anubis` (必填)

请求示例:
```json
{
  "character_id": "Anubis",
  "key": "xxx",
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/add_tech_points

**给予科技点**

直接修改玩家的 TechnologyPoint 字段。

- `steamid`: 目标玩家 SteamID (必填)
- `points`: 科技点数量 (默认 1)

请求示例:
```json
{
  "key": "xxx",
  "points": 10,
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/add_boss_tech_points

**给予古代科技点**

直接修改玩家的 bossTechnologyPoint 字段。

- `steamid`: 目标玩家 SteamID (必填)
- `points`: 古代科技点数量 (默认 1)

请求示例:
```json
{
  "key": "xxx",
  "points": 5,
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/add_player_exp

**给予玩家经验**

给予指定玩家经验值。支持两种模式：
- 默认: 直接修改 Exp 字段（安全，但不触发升级）
- `use_debug_method=true`: 调用 Debug 方法（自动升级，但可能被反作弊拦截）

参数:
- `steamid`: 目标玩家 SteamID (必填)
- `exp`: 经验值 (默认 100)
- `use_debug_method`: 是否使用 Debug 方法 (默认 false)

请求示例:
```json
{
  "exp": 1000,
  "key": "xxx",
  "steamid": "xxx",
  "use_debug_method": false
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/add_party_exp

**给予队伍经验**

给予指定玩家队伍中所有帕鲁经验值。

- `steamid`: 目标玩家 SteamID (必填)
- `exp`: 经验值 (默认 100)
- `use_debug_method`: 是否使用 Debug 方法 (默认 false)

请求示例:
```json
{
  "exp": 500,
  "key": "xxx",
  "steamid": "xxx",
  "use_debug_method": false
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/add_all_player_exp

**给予所有玩家经验**

给予当前所有在线玩家经验值。

- `exp`: 经验值 (默认 100)
- `use_debug_method`: 是否使用 Debug 方法 (默认 false)

请求示例:
```json
{
  "exp": 1000,
  "key": "xxx",
  "use_debug_method": false
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/kick_player

**踢出玩家**

将指定玩家踢出服务器。

- `steamid`: 目标玩家 SteamID (必填)
- `reason`: 踢出原因 (默认 "Kicked by admin")

请求示例:
```json
{
  "key": "xxx",
  "reason": "违规行为",
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/chat

**公屏聊天**

以指定身份发送公屏聊天消息。

- `message`: 消息内容 (必填)
- `category`: 类别 1=全局 2=公会 3=附近 (默认 1)
- `sender`: 发送者名称 (默认 "Server")

请求示例:
```json
{
  "category": 1,
  "key": "xxx",
  "message": "Hello World",
  "sender": "Server"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/private_message

**私聊消息**

向指定玩家发送私聊消息。

- `steamid`: 目标玩家 SteamID (必填)
- `message`: 消息内容 (必填)

请求示例:
```json
{
  "key": "xxx",
  "message": "你好",
  "steamid": "xxx"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/announce

**系统公告**

发送系统公告（入侵警报风格），所有在线玩家可见。

- `message`: 公告内容 (必填)

请求示例:
```json
{
  "key": "xxx",
  "message": "服务器将在10分钟后重启"
}
```

响应示例:
```json
{
  "status": "success"
}
```

---

## POST /api/resources

**导出所有资源**

导出所有文档和模板资源，支持查看当前语言文档或一次性导出全部语言文档。

请求示例:
```json
{
  "key": "xxx"
}
```

响应示例:
```json
{"status":"success","timestamp":"...","resources":{"docs":[...],"templates":[...]},"summary":{...}}
```

---

## POST /api/field_definitions

**获取可配置字段定义**

返回 SaveParameter 和 StaticParameter 的所有可配置字段，包括字段名、类型、是否必填等。

请求示例:
```json
{
  "key": "xxx"
}
```

响应示例:
```json
{"status":"success","SaveParameter":[{"key":"CharacterID","type":"string","required":true},...]}
```

---

*由 PalServerBridge 自动生成，请勿手动编辑*

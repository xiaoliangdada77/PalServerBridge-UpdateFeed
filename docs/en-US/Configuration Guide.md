# PalServerBridge Configuration Guide

## config.json

| Field | Type | Default | Description |
|------|------|--------|------|
| http_port | int | 8888 | HTTP server port |
| api_key | string | "change_me_to_a_secure_key" | API key (must be changed) |
| log.global_level | string | "info" | Global minimum log level |
| log.console_level | string | "info" | UE4SS and game console log level |
| log.file_level | string | "debug" | File log level |
| log.file_enabled | bool | true | Enable file logging |
| log.max_files | int | 20 | Maximum retained log files |
| log.async_write | bool | true | Use asynchronous file logging |
| game_api.use_rest_api | bool | true | Use REST API |
| game_api.rest_api_url | string | "http://127.0.0.1:8211/v1/api" | REST API endpoint |
| game_api.rest_api_username | string | "admin" | REST API username |
| game_api.rest_api_password | string | "" | REST API password |
| update_notice.enabled | bool | true | Enable update notice check |
| update_notice.feed_url | string | GitHub raw UPDATE.md | Remote update feed URL |
| update_notice.timeout_ms | int | 3000 | Update request timeout in milliseconds |
| i18n.language | string | auto | Language code. First run resolves auto to system language |

## Automatic Config Synchronization

- New config keys added by newer versions are automatically inserted into the existing config.json.
- Deprecated config keys from older versions are automatically removed.
- User-modified values are preserved whenever types still match.
- Type mismatches are reset back to the default value.

---
*Generated automatically by PalServerBridge. Do not edit manually.*

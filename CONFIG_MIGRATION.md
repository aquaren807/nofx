# 配置文件迁移说明

## 迁移内容

已将 `trader/config.json` 的配置迁移到主配置文件 `config.json` 中，使用新字段 `custom_ai_model`。

## 变更详情

### 1. `config.json` 更新
在 `config.json` 中添加了新的顶级字段 `custom_ai_model`：

```json
{
  "admin_mode": true,
  "beta_mode": false,
  "leverage": { ... },
  "use_default_coins": true,
  "default_coins": [ ... ],
  "api_server_port": 8080,
  "max_daily_loss": 10.0,
  "max_drawdown": 20.0,
  "stop_trading_minutes": 60,
  "jwt_secret": "...",
  "custom_ai_model": {
    "id": "ark-trader-demo",
    "name": "Ark Doubao Trader",
    "ai_model": "custom",
    "custom_api_url": "https://ark.cn-beijing.volces.com/api/v3",
    "custom_api_key": "b1d26f00-3901-40ae-a4f3-c978f18f6c5d",
    "custom_model_name": "doubao-seed-1-6-thinking-250715"
  }
}
```

### 2. `main.go` 更新

#### 2.1 ConfigFile 结构体
添加了 `CustomAIModel` 字段：
```go
type ConfigFile struct {
  // ... 其他字段 ...
  CustomAIModel      TraderCustomModelFile `json:"custom_ai_model"`
}
```

#### 2.2 syncConfigToDatabase() 函数
- 在同步配置时，现在会读取并处理 `config.json` 中的 `custom_ai_model` 字段
- 如果该字段配置完整，会自动将自定义模型同步到数据库

#### 2.3 syncCustomModelToDefault() 函数
- 现在作为后备函数，仅在 `config.json` 中未配置 `custom_ai_model` 时才使用
- 保持与旧的 `trader/config.json` 兼容性

### 3. 文件状态

| 文件 | 状态 | 说明 |
|------|------|------|
| `config.json` | ✅ 已更新 | 包含所有配置，包括新的 `custom_ai_model` |
| `trader/config.json` | ⚠️ 可选 | 仍然支持作为后备，但不再必需 |

## 迁移优势

1. **单一配置源**：所有配置集中在一个 `config.json` 文件中
2. **简化管理**：不需要维护两个配置文件
3. **更清晰的结构**：配置文件层次更加清晰
4. **后向兼容**：保留对旧 `trader/config.json` 的支持

## 使用方式

### 新用户（推荐）
直接在 `config.json` 的 `custom_ai_model` 字段中配置自定义模型。

### 现有用户（兼容）
- 选项1：将 `trader/config.json` 的内容迁移到 `config.json` 的 `custom_ai_model` 字段
- 选项2：保留 `trader/config.json`，系统仍会读取它作为后备

## 重启生效

修改配置后，需要重启 Backend 服务使其生效：

```bash
# 使用 start.sh 脚本（Docker）
./start.sh restart

# 或手动重启
docker compose restart nofx
```

## 代码更新

| 文件 | 变更 |
|------|------|
| `config.json` | 添加 `custom_ai_model` 字段 |
| `main.go` | 更新 `ConfigFile` 结构体，修改 `syncConfigToDatabase()` 函数 |


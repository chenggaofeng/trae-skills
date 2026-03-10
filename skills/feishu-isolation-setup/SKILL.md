---
name: "feishu-isolation-setup"
description: "为飞书机器人配置多用户隔离（会话隔离、记忆隔离）。当用户需要为公司全员使用的飞书bot做消息隔离时调用此skill。"
language: "zh-CN"
---

# 飞书机器人多用户隔离配置

此skill用于为飞书机器人配置多用户隔离，实现会话隔离、记忆隔离和Cron隔离。

## 适用场景

- 飞书bot设置为全公司可用
- 需要每个用户的对话历史独立
- 需要每个用户存储的信息独立（用户A的信息不应被用户B看到）
- 需要定时任务在独立上下文中执行

## 核心原理

使用OpenClaw飞书插件的 **Dynamic Agent Creation** 功能，为每个私聊用户自动创建独立Agent。

每个Agent拥有：
- 独立的workspace目录（包含独立的USER.md、SOUL.md等）
- 独立的agentDir目录（存储auth profiles、model registry）
- 独立的sessions目录（存储会话历史）
- **完全隔离的记忆和数据**

## 配置步骤

### 1. 备份现有配置

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d)
```

### 2. 清理旧配置（如有）

移除无效的`dmScope`和`sandbox`配置：

```bash
jq 'del(.session.dmScope) | del(.agents.defaults.sandbox)' ~/.openclaw/openclaw.json > /tmp/openclaw_temp.json
mv /tmp/openclaw_temp.json ~/.openclaw/openclaw.json
```

### 3. 重置共享数据

清除之前混合的用户数据：

```bash
cat > ~/.openclaw/workspace/USER.md << 'EOF'
# User Profile

This is your personal AI assistant.
EOF
```

### 4. 应用新配置

```bash
# 启用 Dynamic Agent Creation
openclaw config set --json channels.feishu.dynamicAgentCreation '{"enabled": true, "maxAgents": 100}'

# 私聊开放访问（无需配对）
openclaw config set channels.feishu.dmPolicy open
openclaw config set channels.feishu.allowFrom '["*"]'

# 群聊开放访问
openclaw config set channels.feishu.groupPolicy open

# 群聊免@响应
openclaw config set channels.feishu.requireMention false

# 飞书流式响应
openclaw config set channels.feishu.streaming true
```

### 5. 重启Gateway

```bash
openclaw gateway restart
```

### 6. 验证配置

```bash
openclaw config get channels.feishu
```

预期输出：

```json
{
  "enabled": true,
  "dmPolicy": "open",
  "allowFrom": ["*"],
  "groupPolicy": "open",
  "requireMention": false,
  "streaming": true,
  "dynamicAgentCreation": {
    "enabled": true,
    "maxAgents": 100
  }
}
```

## 完整配置示例

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxx",
      "appSecret": "xxx",
      "domain": "feishu",
      "connectionMode": "websocket",
      "dmPolicy": "open",
      "allowFrom": ["*"],
      "groupPolicy": "open",
      "requireMention": false,
      "renderMode": "card",
      "streaming": true,
      "dynamicAgentCreation": {
        "enabled": true,
        "workspaceTemplate": "~/.openclaw/workspace-{agentId}",
        "agentDirTemplate": "~/.openclaw/agents/{agentId}/agent",
        "maxAgents": 100
      },
      "groups": {
        "*": {
          "requireMention": false,
          "activation": "always"
        }
      }
    }
  }
}
```

## 配置参数说明

| 参数 | 说明 |
|------|------|
| `dynamicAgentCreation.enabled` | 启用动态Agent创建 |
| `dynamicAgentCreation.workspaceTemplate` | workspace路径模板，支持`{agentId}`和`{userId}`占位符 |
| `dynamicAgentCreation.agentDirTemplate` | agentDir路径模板 |
| `dynamicAgentCreation.maxAgents` | 最大Agent数量限制，防止资源耗尽 |
| `dmPolicy: "open"` | 允许所有用户私聊（需配合`allowFrom: ["*"]`） |
| `groupPolicy: "open"` | 允许所有群组访问 |
| `requireMention: false` | 群聊无需@即可响应 |
| `streaming: true` | 启用飞书流式卡片输出 |

## 数据存储结构

```
~/.openclaw/
├── openclaw.json                    # 主配置文件
├── workspace/                       # 默认Agent的workspace（群聊使用）
│   ├── USER.md
│   ├── SOUL.md
│   └── memory/
├── agents/
│   ├── main/                        # 默认Agent
│   │   ├── agent/
│   │   └── sessions/
│   ├── feishu-ou_user_a/            # 用户A的独立Agent
│   │   ├── agent/
│   │   └── sessions/
│   └── feishu-ou_user_b/            # 用户B的独立Agent
│       ├── agent/
│       └── sessions/
├── workspace-feishu-ou_user_a/      # 用户A的独立workspace
│   ├── USER.md                      # 用户A的个人信息（隔离）
│   ├── SOUL.md
│   └── memory/                      # 用户A的记忆（隔离）
└── workspace-feishu-ou_user_b/      # 用户B的独立workspace
    ├── USER.md                      # 用户B的个人信息（隔离）
    ├── SOUL.md
    └── memory/                      # 用户B的记忆（隔离）
```

## 验证方法

### 私聊隔离测试

1. 用户A首次私聊机器人，发送消息
2. 用户A发送："记住我的代号是Alpha"
3. 用户B首次私聊机器人，发送消息
4. 用户B发送："我之前让你记住了什么？"
5. 验证用户B无法看到用户A的信息

### 服务器端验证

```bash
# 检查是否创建了独立的workspace目录
ls -la ~/.openclaw/ | grep workspace-feishu

# 检查是否创建了独立的Agent目录
ls -la ~/.openclaw/agents/ | grep feishu

# 查看已创建的Agent
openclaw agents list --bindings
```

## 常见问题

### Q: 为什么`dmScope: "per-channel-peer"`不能实现记忆隔离？

`dmScope`只控制session JSONL文件的隔离，不影响workspace文件。AI仍然会写入共享的`USER.md`文件。

### Q: 群聊是否也会自动创建独立Agent？

不会。群聊默认使用main Agent。如需为特定群组创建独立Agent，需要手动配置bindings。

### Q: 如何为特定群组创建独立Agent？

```bash
# 创建独立Agent
openclaw agents add --workspace ~/.openclaw/workspace-dept-a feishu-dept-a

# 绑定群组到独立Agent
openclaw config set --json bindings '[
  {
    "agentId": "feishu-dept-a",
    "match": {
      "channel": "feishu",
      "peer": {
        "kind": "group",
        "id": "oc_xxx"
      }
    }
  }
]'
```

### Q: 话题群如何实现独立上下文？

```bash
openclaw config set channels.feishu.groupSessionScope group_topic
```

可选值：
- `group` - 每个群一个会话（默认）
- `group_sender` - 每个群+发送者一个会话
- `group_topic` - 每个群话题一个会话
- `group_topic_sender` - 每个群+话题+发送者一个会话

## 参考资料

- [OpenClaw官方文档 - Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)
- [OpenClaw官方文档 - Feishu Channel](https://docs.openclaw.ai/channels/feishu)
- [飞书官方OpenClaw插件文档](https://www.feishu.cn/content/article/7613711414611463386)
- [OpenClaw GitHub - dynamic-agent.ts源码](https://github.com/openclaw/openclaw/blob/main/extensions/feishu/src/dynamic-agent.ts)

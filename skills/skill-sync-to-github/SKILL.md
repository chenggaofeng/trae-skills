---
name: "skill-sync-to-github"
description: "将本地 skill 同步到 GitHub 仓库和全局目录。当用户创建或更新 skill 后需要同步时调用此 skill。"
language: "zh-CN"
---

# Skill 同步到 GitHub 和全局

此 skill 用于将本地创建或更新的 skill 同步到：
1. GitHub 仓库 `trae-skills`
2. 全局 skill 目录 `~/.trae-cn/skills/`

## 适用场景

- 用户刚创建了一个新的 skill
- 用户更新了现有的 skill
- 用户需要将 skill 推送到 GitHub 进行版本管理
- 用户需要将 skill 同步到全局目录

## 路径说明

| 类型 | 路径 |
|------|------|
| **项目 Skills** | `/home/cgaof/2026/ws-n100/.trae/skills/` |
| **全局 Skills** | `~/.trae-cn/skills/` |
| **GitHub 仓库** | `/home/cgaof/2026/ws-n100/trae-skills/skills/` |

## 前置条件

1. 已创建 GitHub 仓库：`chenggaofeng/trae-skills`
2. 本地仓库路径：`/home/cgaof/2026/ws-n100/trae-skills`
3. Git 已配置好认证
4. 全局目录已创建：`~/.trae-cn/skills/`

## 同步步骤

### 1. 确定要同步的 skill

首先确认用户要同步哪个 skill：

```bash
# 查看本地所有 skill
ls -la /home/cgaof/2026/ws-n100/.trae/skills/
```

### 2. 同步到全局目录

```bash
# 确保全局目录存在
mkdir -p ~/.trae-cn/skills

# 复制 skill 到全局目录
SKILL_NAME="<skill-name>"
cp -r /home/cgaof/2026/ws-n100/.trae/skills/$SKILL_NAME ~/.trae-cn/skills/
echo "Synced $SKILL_NAME to global directory"
```

### 3. 复制 skill 到 Git 仓库

```bash
# 复制 skill 到 trae-skills 仓库
cp -r /home/cgaof/2026/ws-n100/.trae/skills/$SKILL_NAME /home/cgaof/2026/ws-n100/trae-skills/skills/
```

### 4. 更新 README.md（如有新 skill）

如果是新 skill，需要更新 README.md 的 Skills 列表：

```markdown
| Skill | 描述 |
|-------|------|
| [feishu-isolation-setup](./skills/feishu-isolation-setup/) | 飞书机器人多用户隔离配置 |
| [skill-sync-to-github](./skills/skill-sync-to-github/) | 将本地 skill 同步到 GitHub |
| [<new-skill>](./skills/<new-skill>/) | <描述> |
```

### 5. 提交并推送

```bash
cd /home/cgaof/2026/ws-n100/trae-skills

# 查看变更
git status

# 添加所有变更
git add .

# 提交
git commit -m "Add/Update <skill-name> skill"

# 推送到 GitHub
git push
```

## 完整脚本

```bash
#!/bin/bash
# sync-skill.sh - 同步 skill 到 GitHub 和全局目录

SKILL_NAME=$1

if [ -z "$SKILL_NAME" ]; then
    echo "Usage: sync-skill.sh <skill-name>"
    exit 1
fi

SKILL_SRC="/home/cgaof/2026/ws-n100/.trae/skills/$SKILL_NAME"
SKILL_DEST="/home/cgaof/2026/ws-n100/trae-skills/skills/$SKILL_NAME"
GLOBAL_DEST="$HOME/.trae-cn/skills/$SKILL_NAME"
REPO_DIR="/home/cgaof/2026/ws-n100/trae-skills"

if [ ! -d "$SKILL_SRC" ]; then
    echo "Error: Skill '$SKILL_NAME' not found at $SKILL_SRC"
    exit 1
fi

# 1. 同步到全局目录
mkdir -p "$HOME/.trae-cn/skills"
cp -r "$SKILL_SRC" "$GLOBAL_DEST"
echo "Synced $SKILL_NAME to global directory: $GLOBAL_DEST"

# 2. 复制 skill 到 Git 仓库
cp -r "$SKILL_SRC" "$SKILL_DEST"
echo "Copied $SKILL_NAME to trae-skills repo"

# 3. 进入仓库并推送
cd "$REPO_DIR"

# Git 操作
git add .
git commit -m "Update $SKILL_NAME skill"
git push

echo "Successfully synced $SKILL_NAME to GitHub and global directory!"
```

## 使用示例

### 示例 1：同步单个 skill

```bash
# 同步 feishu-isolation-setup skill
# 1. 同步到全局
cp -r /home/cgaof/2026/ws-n100/.trae/skills/feishu-isolation-setup ~/.trae-cn/skills/

# 2. 同步到 GitHub
cd /home/cgaof/2026/ws-n100/trae-skills
cp -r ../.trae/skills/feishu-isolation-setup ./skills/
git add .
git commit -m "Update feishu-isolation-setup skill"
git push
```

### 示例 2：同步所有 skill

```bash
# 同步所有 skill 到全局
cp -r /home/cgaof/2026/ws-n100/.trae/skills/* ~/.trae-cn/skills/

# 同步所有 skill 到 GitHub
cd /home/cgaof/2026/ws-n100/trae-skills
cp -r ../.trae/skills/* ./skills/
git add .
git commit -m "Sync all skills"
git push
```

## 验证同步结果

```bash
# 检查全局目录
ls -la ~/.trae-cn/skills/

# 检查 GitHub 仓库状态
gh repo view chenggaofeng/trae-skills

# 或访问网页
# https://github.com/chenggaofeng/trae-skills
```

## 注意事项

1. **确保 Git 认证**：首次推送需要配置 Git 认证
2. **检查冲突**：如果远程有更新，先 `git pull` 再推送
3. **更新 README**：新增 skill 时记得更新 README.md
4. **全局目录**：确保 `~/.trae-cn/skills/` 目录存在

## 相关链接

- GitHub 仓库：https://github.com/chenggaofeng/trae-skills
- 本地仓库：`/home/cgaof/2026/ws-n100/trae-skills`
- 全局 Skills：`~/.trae-cn/skills/`

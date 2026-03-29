# TDR 自动规划 - OpenClaw Skill 包装器

**功能：** 通过自然语言消息自动创建 TDR 任务

---

## 触发关键词

- `"创建 TDR 任务"`
- `"启动进化任务"`
- `"自动规划"`
- `"TDR: "`

---

## 使用示例

### 示例 1：创建 HTTP 服务器

**用户：**
```
创建一个 HTTP 服务器，监听 3000 端口，返回 Hello World
```

**Agent 响应：**
```
🎯 正在自动规划任务...
📡 调用 Sonnet 拆解任务...
✅ 任务已创建：task-002
   标题：创建 HTTP 服务器
   子任务：5 个
   复杂度：4/10

🚀 开始执行...
```

---

## 实现方式

### 方法 1：通过 sessions_spawn（推荐）

在 Agent 中集成：

```javascript
// 当检测到创建任务的意图时
if (message.includes('创建') && (message.includes('任务') || message.includes('功能'))) {
  // 调用 Sonnet 规划
  const plan = await sessions_spawn({
    task: `请规划以下需求：${message}`,
    model: 'advanced-model/sonnet',
    mode: 'run'
  });
  
  // 创建任务文件
  const taskId = generateTaskId();
  createTaskFile(plan, taskId);
  
  // 回复用户
  send(`✅ 任务已创建：${taskId}`);
}
```

### 方法 2：通过 CLI 调用

```javascript
const { execSync } = require('child_process');

execSync(`node scripts/auto-plan.js "${message}"`, {
  stdio: 'inherit'
});
```

---

## 完整流程

```
用户消息
   ↓
意图识别（创建任务）
   ↓
调用 auto-plan.js
   ↓
sessions_spawn → Sonnet 规划
   ↓
解析 JSON
   ↓
创建任务文件
   ↓
回复用户 + 开始执行
```

---

## 集成到现有 Agent

在 `SOUL.md` 或 `AGENTS.md` 中添加：

```markdown
## TDR 自动规划

当用户说"创建 XX"、"实现 XX 功能"时：

1. 调用 `node scripts/auto-plan.js "需求"`
2. 自动规划并创建任务
3. 自动开始执行

示例：
用户："创建一个 HTTP 服务器"
→ 自动规划 → 创建 task-XXX → 开始执行
```

---

## 配置 Cron 自动执行

```bash
# 监控新任务并自动开始执行
openclaw cron add \
  --name "tdr-auto-execute" \
  --schedule "*/2 * * * *" \
  --agent wilson \
  --message "cd <workspace>/workspace/skills/test-driven-revolution && node scripts/heartbeat-coordinator.js"
```

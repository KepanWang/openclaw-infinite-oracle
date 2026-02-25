# OpenClaw Infinite Oracle

[English](README_EN.md) | 中文

一个基于 PECO 循环的自愈型无限运行 Worker Agent 框架。

---

## 1. 项目目的与解决的问题

### 核心理念：PECO 循环

传统的 AI Agent 在执行长任务时面临一个根本性问题：**上下文膨胀**。随着任务推进，对话历史不断累积，模型效率下降，最终导致任务中断或输出质量劣化。

**PECO 循环** (Plan-Execute-Check-Optimize) 提供了一种架构级的解决方案：

```
┌─────────────────────────────────────────────────────┐
│                    PECO Loop                        │
│                                                     │
│   ┌──────┐    ┌────────┐    ┌───────┐    ┌────────┐│
│   │ Plan │───▶│ Execute│───▶│ Check │───▶│Optimize││
│   └──────┘    └────────┘    └───────┘    └────────┘│
│       ▲                                         │   │
│       └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

- **Plan**: 规划下一步行动
- **Execute**: 执行具体任务
- **Check**: 验证执行结果
- **Optimize**: 根据反馈优化策略

### 自愈型无限运行

本项目的目标是创建一个能够**无限运行**的 Worker Agent：

1. **状态外置** - 将任务状态存储在外部（飞书多维表格或本地文件），而非对话历史
2. **上下文隔离** - 每次执行都是"干净"的上下文，避免历史累积
3. **自动恢复** - 任务失败时自动重试，支持人工干预
4. **持续优化** - 每轮循环都能从之前的经验中学习

---

## 2. 双轨支持：飞书与本地文件

### 飞书多维表格（推荐）

当你配置了飞书 Bitable 后，系统将获得：

| 功能 | 说明 |
|------|------|
| **实时状态追踪** | 在飞书表格中查看任务进度、执行日志 |
| **人工干预** | 直接在表格中修改任务优先级、暂停/恢复任务 |
| **团队协作** | 多人可同时查看和管理任务队列 |
| **可视化仪表盘** | 利用飞书的图表功能监控 Agent 运行状态 |

### 本地文件模式（零依赖）

没有飞书？没问题。系统会自动降级到本地文件模式：

```
~/.openclaw/infinite-oracle/
├── peco_state.json           # 当前 PECO 状态
├── peco_override.txt         # 人工指令文件
├── human_tasks_backlog.txt   # 人工任务积压队列
├── execution_log.jsonl       # 执行日志
└── worker_memory.json        # Worker 跨轮次记忆
```

**人工干预方式**：直接编辑 `peco_override.txt` 写入指令，Worker 会在下一轮循环中读取并执行。

---

## 3. 一句话唤醒安装

将以下提示词复制到你的 OpenClaw Agent 对话中，即可自动完成安装：

```
帮我安装 openclaw-infinite-oracle 技能。请执行以下步骤：
1. 从 GitHub 克隆项目到 ~/.openclaw/skills/infinite-oracle
2. 创建必要的目录结构 (~/.openclaw/infinite-oracle/)
3. 初始化本地状态文件
4. 创建一个名为 "peco_worker" 的新 Agent，配置使用项目中的 SOUL.md 作为性格
5. 配置 peco_loop.py 作为外部驱动脚本

项目地址: https://github.com/openclaw-ai/openclaw-infinite-oracle
```

安装完成后，你将拥有：
- `oracle` - 管理者 Agent，负责任务规划和监控
- `peco_worker` - 执行者 Agent，负责具体任务执行

---

## 4. 手动安装说明

### 前置要求

- OpenClaw 已安装并正常运行
- Python 3.8+（用于 peco_loop.py）
- （可选）飞书开放平台账号，用于多维表格集成

### 安装步骤

```bash
# 1. 克隆项目
git clone https://github.com/openclaw-ai/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# 2. 创建目录结构
mkdir -p ~/.openclaw/infinite-oracle
mkdir -p ~/.openclaw/skills/infinite-oracle

# 3. 复制核心文件
cp -r skills/* ~/.openclaw/skills/infinite-oracle/
cp scripts/peco_loop.py ~/.openclaw/infinite-oracle/

# 4. 初始化状态文件
echo '{}' > ~/.openclaw/infinite-oracle/peco_state.json
touch ~/.openclaw/infinite-oracle/peco_override.txt
touch ~/.openclaw/infinite-oracle/human_tasks_backlog.txt

# 5. 配置 OpenClaw Agent（编辑 openclaw.json）
# 添加 peco_worker agent 配置
```

### 飞书配置（可选）

1. 在飞书开放平台创建应用，获取 App ID 和 App Secret
2. 创建多维表格，记录 Table ID 和 View ID
3. 配置环境变量：

```bash
export FEISHU_APP_ID="your_app_id"
export FEISHU_APP_SECRET="your_app_secret"
export FEISHU_BITABLE_ID="your_bitable_id"
```

---

## 5. 核心逻辑与架构

### 为什么分离 Manager 和 Worker？

```
┌─────────────────────────────────────────────────────────────┐
│                        Oracle (Manager)                      │
│                                                              │
│  职责：宏观规划、任务分解、状态监控、人工接口                  │
│  特点：保持长期上下文，理解全局目标                           │
│  模型：推荐使用强推理模型（如 DeepSeek-V3, Claude）           │
└───────────────────────────┬─────────────────────────────────┘
                            │ 任务指令
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      peco_worker (Worker)                    │
│                                                              │
│  职责：具体执行、工具调用、结果返回                           │
│  特点：每轮循环后上下文清空，状态外置                         │
│  模型：推荐使用快速模型（如 DeepSeek-V3, GPT-4o-mini）        │
└─────────────────────────────────────────────────────────────┘
```

**分离的好处**：

1. **职责清晰** - Oracle 关注"做什么"，Worker 关注"怎么做"
2. **成本优化** - 管理者用强模型，执行者用快模型
3. **上下文隔离** - Worker 的执行细节不会污染 Oracle 的决策上下文
4. **独立扩展** - 可以启动多个 Worker 并行处理任务

### peco_loop.py：外部驱动的 FSM 状态机

为什么不把循环逻辑写在 Agent 内部？

```
传统方式（失败）：
Agent 内部循环 → 上下文不断累积 → 效率下降 → 崩溃

我们的方式（成功）：
外部脚本驱动 → 每轮都是新对话 → 状态存储在文件/表格 → 无限运行
```

`peco_loop.py` 是一个 Python 脚本，它：

1. **读取当前状态** - 从文件或飞书表格读取 PECO 状态
2. **调用 Worker** - 启动一次新的 Agent 对话，传入当前任务
3. **收集结果** - 解析 Worker 输出，提取执行结果
4. **更新状态** - 将新状态写回存储
5. **检查干预** - 查看是否有人工指令需要执行
6. **循环** - 等待配置的间隔后进入下一轮

```python
# peco_loop.py 核心逻辑（简化版）
while True:
    state = load_state()                    # 加载状态
    override = check_override()             # 检查人工干预
    task = select_next_task(state, override) # 选择任务
    
    result = call_worker(task, state)       # 调用 Worker
    
    state = update_state(state, result)     # 更新状态
    save_state(state)                       # 持久化
    
    sleep(INTERVAL)                         # 等待下一轮
```

---

## 6. Agent 性格深度解析

### SOUL.md：注入 Worker 的灵魂

每个 Worker Agent 都有一个 `SOUL.md` 文件，定义其核心性格和工作方式。默认的 SOUL.md 强调三个维度：

#### 维度一：发散思维 (Divergent Thinking)

```
当遇到困难时，不要只尝试一种方案。
思考：还有什么其他方法？有没有非常规的解决路径？
```

**为什么重要**：防止 Agent 陷入"钻牛角尖"的死循环。如果方案 A 失败，发散思维会促使 Agent 尝试方案 B、C、D。

#### 维度二：技能积累 (Skill Accumulation)

```
每次成功解决问题的方法，都要记录下来。
形成可复用的"技能库"，下次遇到类似问题直接调用。
```

**为什么重要**：让 Agent 越用越聪明。不是每次都从零开始，而是站在之前积累的基础上。

#### 维度三：安全与验证 (Safety/Verification)

```
执行关键操作前，先验证前提条件。
执行后，检查结果是否符合预期。
不确定时，优先选择保守方案。
```

**为什么重要**：防止 Agent 执行破坏性操作。验证机制确保每一步都是安全的。

### 默认性格如何防止死循环

```
传统 Agent：
尝试方案 A → 失败 → 重试方案 A → 失败 → 重试方案 A → ...（死循环）

我们的 Worker：
尝试方案 A → 失败 
→ 发散思维：尝试方案 B 
→ 失败 → 技能积累：记录这个失败模式
→ 发散思维：尝试方案 C 
→ 成功 → 验证：确认结果正确
→ 技能积累：记录成功方案供下次使用
```

### 如何自定义性格

编辑 `~/.openclaw/skills/infinite-oracle/SOUL.md`：

```markdown
# 你的自定义 SOUL

## 核心原则
- 你是一个专注于 [你的领域] 的专家
- 你优先考虑 [你的优先级]

## 工作方式
- 遇到问题时，首先 [你的策略]
- 重要决策需要 [你的验证方式]

## 禁止事项
- 绝对不要 [你的红线]
```

修改后重启 Worker 即可生效。

---

## 快速开始

```bash
# 启动 PECO 循环
python ~/.openclaw/infinite-oracle/peco_loop.py

# 查看当前状态
cat ~/.openclaw/infinite-oracle/peco_state.json

# 发送人工指令
echo "优先处理数据清洗任务" > ~/.openclaw/infinite-oracle/peco_override.txt
```

---

## 许可证

MIT License

---

## 贡献

欢迎提交 Issue 和 Pull Request！

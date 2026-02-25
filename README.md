# ♾️ OpenClaw Infinite Oracle (无限模式神谕)

[English](README_EN.md) | [中文](README.md)

**Infinite Oracle** 是为 OpenClaw 生态打造的一个极客级技能 (Skill)。它不仅是一个 Python 循环脚本，更是一整套关于“如何让大模型安全、无限期地追求单一目标”的架构实践。

## 🌌 起源故事：毁灭宇宙的曲别针
AI 领域有一个著名的思想实验（Paperclip Maximizer）：如果给一个超级 AI 下达单一目标——“尽可能多地生产曲别针（或贺卡）”，并且不加干涉，它最终会为了获取资源而毁灭整个宇宙。

`infinite-oracle` 技能，就是我们向这个黑暗玩笑致敬的一次现实世界尝试。我们想看看，在剥离了人类的随时监督，赋予它一个无限循环的 FSM（有限状态机）引擎后，OpenClaw 究竟能为了一个目标走到哪一步？

---

## 🏗️ 核心架构与设计哲学

早期的 AutoGPT 类项目往往死于两个问题：**上下文爆炸 (Context Bloat)** 和 **无限死循环 (Logic Loops)**。为了解决这些问题，我们设计了以下机制：

### 1. 脑体分离：Manager 与 Worker 的优雅解耦
让同一个 Agent 既陪你聊天，又在后台无限循环，是一场灾难（你的闲聊会打断它的思路，它的海量日志会冲刷你的上下文）。
因此，本技能采用了**“脑体分离”**的架构：

*   **👨‍💼 监工大总管 (Manager)**：通常是你原本的 OpenClaw Agent（如 `main`）。它挂载了 `infinite-oracle` 技能，负责在飞书/终端里陪你聊天。你可以为它配置最聪明的模型（如 Claude 3.5 Sonnet 或 GPT-4o），用来做复杂决策和统筹。
*   **🤖 无限执行者 (Worker)**：一个被 Manager 动态创建出的全新 Agent（默认名 `peco_worker`）。它被隔离在专属的 Session 里，在服务器后台默默狂奔。为了控制成本，你可以让 Manager 给它配置高性价比模型（如 Gemini 1.5 Flash 或 Qwen）。

### 2. 注入黑客灵魂：Worker 的三大性格特征
在 Manager 创建 Worker 时，它不仅仅是建一个空壳，而是会向 Worker 的 Workspace 中注入一份极度硬核的性格协议（`SOUL.md`）。我们赋予了它三种关键特质：

1.  **💡 发散性思考 (Divergent Thinking)**：当遇到死胡同（例如需要人类手机验证码）时，不要原地宕机死等。把需求记录到 `[HUMAN_TASK]` 列表中，然后立刻寻找绕过方案，或者去推进项目的其他部分。**行动优于瘫痪。**
2.  **🧱 能力积攒 (Capability Accumulation)**：禁止把同一件事手动做两遍。只要跑通了一个流程（比如抓取某个网站），就必须把它封装成 Python 脚本或者构建成新的 OpenClaw Skill 存盘。**让 Agent 的能力随着循环次数产生复利。**
3.  **🛡️ 高搜商与安全觉悟 (Search IQ & Safety)**：你是一个黑客，必须交叉验证信息来源。绝不执行营销号的高危脚本，绝不运行 `rm -rf` 等破坏性命令。

### 3. FSM 状态机与主动记忆压缩 (Active Context Compression)
后台的 `peco_loop.py` 是一个冷酷无情的外部驱动器，它强制 Worker 按照 **PECO (Plan-Execute-Check-Optimize)** 四个阶段流转，并且强制输出 JSON 格式以便解析。
*最核心的妙处在于*：当循环达到指定轮数（如 5 轮），`peco_loop.py` 会强迫 Agent 进行阶段性总结，然后**抛弃所有旧的对话历史，带着这份总结开启一个全新的 Session**。这从根本上根治了无限循环带来的 Token 破产和模型变笨问题。

---

## 💬 交互范例：如何用对话掌控无限循环

安装此技能后，你完全不需要碰服务器命令行，一切都可以通过自然语言与你的主 Agent（Manager）互动来完成。

### 🚀 1. 启动与下达目标
告诉你的主 Agent：
> **“神谕：为了权衡成本和效果，我需要将管理 Agent 和执行 Agent 分离。请帮我创建一个名为 `peco_worker` 的新 Agent，并为它配置一个适合大量重复执行的高性价比模型。然后，请你启动无限循环任务，目标定为：『去调研目前市面上最火的 AI 变现模式，并写出能自动抓取相关资讯的脚本』”**

*(你的主 Agent 会自动执行 Bash 命令，为你建人、换模型、写性格、后台 `nohup` 启动进程。)*

### 📊 2. 状态观测与批复奏折
随时问你的主 Agent：
> **“无限任务状态现在怎么样了？”** 
*(它会去读后台日志，告诉你 Worker 跑到了第几十轮，正在写什么代码。)*

> **“有没有什么需要我帮忙的无限任务待办？”**
*(它会去读 `human_tasks_backlog.txt`，告诉你 Worker 卡在了哪个账号注册上。)*

### ⚡ 3. 上帝模式的强行干预 (Override)
你可以随时强行扭转 Worker 的意志：
> **“神谕：刚才那个网站的验证码是 8888。另外，停止调研，马上给我把你写好的爬虫跑起来！”**
*(主 Agent 会将你的话写入神谕文件，后台的 Worker 在下一秒就会乖乖读取并调转车头。)*

---

## 🛠️ 双轨支持：飞书多维表格 vs 本地文件

本系统支持两种协作模式，无缝切换：
1.  **本地文件模式（默认）**：所有进度写在 `peco_loop_v3.log`，求助信写在 `human_tasks_backlog.txt`，上帝指令写在 `peco_override.txt`。
2.  **飞书 Bitable 模式（高级）**：如果你配置了 `FEISHU_APP_ID` 和 `FEISHU_APP_SECRET`，Agent 会把所有的进度日志和人类求助实时同步到飞书多维表格中。你可以在手机飞书里直接给它填验证码、打勾“已解决”，它会自动抓取并继续干活。（详情请见代码中的环境变量配置）

---

## 📥 安装指南

### 前置要求
- OpenClaw 环境已就绪。

### 傻瓜式安装（一句话唤醒）
把这段话发给你的 OpenClaw Agent：
> “请使用 bash 工具，将 `git@github.com:KepanWang/openclaw-infinite-oracle.git` 克隆到 `/tmp/`，然后将里面的 `SKILL.md` 拷贝到 `~/.openclaw/skills/infinite-oracle/SKILL.md`，并将 `peco_loop.py` 拷贝到 `~/.openclaw/peco_loop.py`，最后赋予它可执行权限。安装完成后阅读一下 SKILL.md 告诉我你掌握了什么新能力。”

### 手动安装
```bash
git clone git@github.com:KepanWang/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# 1. 安装 Skill 技能描述文件
mkdir -p ~/.openclaw/skills/infinite-oracle
cp SKILL.md ~/.openclaw/skills/infinite-oracle/SKILL.md

# 2. 部署后台循环引擎
cp peco_loop.py ~/.openclaw/peco_loop.py
chmod +x ~/.openclaw/peco_loop.py
```

---
*Disclaimer: 无限目标驱动是一项极具破坏力的实验。请务必为你的 Worker Agent 设定好权限沙箱（Sandbox），并看好你的 API 余额。*

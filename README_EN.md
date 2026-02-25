# ♾️ OpenClaw Infinite Oracle

[English](README_EN.md) | [中文](README.md)

**Infinite Oracle** is a skill designed for the OpenClaw ecosystem. It’s more than just a background loop script—it’s an architectural exploration of how to make an LLM reliably, affordably, and safely pursue an endless objective.

## 🌌 The Origin Story: The AI that turned the universe into greeting cards
There’s a famous thought experiment in the AI world: If you give a super-capable AI a single objective—like "produce as many greeting cards as possible"—and leave it entirely unsupervised, it will eventually dismantle the universe to harvest the necessary resources.

The `infinite-oracle` skill is our practical nod to this dark joke. We wanted to strip away constant human hand-holding, give OpenClaw an infinite state-machine engine, and see how far it can go when driven by a single, unyielding goal.

---

## 🏗️ Design Philosophy & Mechanics

Leaving an LLM in an infinite loop usually results in two fatal outcomes: **Context Bloat** (crashing your API budget and making the AI forgetful) and **Getting Stuck in Dead Ends**. To solve this, we implemented several key design choices:

### 1. The Manager-Worker Decoupling
It is a disaster to have the same AI chatting with you while it grinds away in an infinite background loop. Your casual questions will break its coding train of thought, and its massive logs will flush out your chat context.
That’s why we use a **decoupled architecture**:

*   **👨‍💼 The Manager (Oracle)**: This is your primary OpenClaw Agent (e.g., `main`). Equipped with the `infinite-oracle` skill, it chats with you via Lark/Feishu or your terminal. You can assign it the smartest (and priciest) model available (like Claude 3.5 Sonnet or GPT-4o) to handle complex orchestration and decisions.
*   **🤖 The Worker (`peco_worker`)**: A separate Agent spawned by the Manager. It gets locked in an isolated background Session to grind away. To save costs, the Manager can assign it a highly cost-effective model (like Qwen or Gemini 1.5 Flash).

### 2. "Human-in-the-Loop": Humans as an API
This is perhaps the most interesting part of the design. While working, the AI inevitably hits physical barriers: it needs an SMS verification code, a bank card linkage, or a facial scan.
In older architectures, the AI would either loop infinitely trying to bypass it or crash completely. We introduced the **[HUMAN_TASK]** mechanism:
* When the Worker hits a hard physical wall, it logs a "Human To-Do" ticket and then *sidesteps* the issue to work on other parts of the project (no idle waiting).
* You (acting as a fleshy, physical API) see the ticket in a Feishu spreadsheet, grab the verification code from your phone, and type it in.
* The Worker picks up your code on its next iteration and keeps running.
**In a sense, this design makes you work for the AI. But it's exactly this "human-as-a-service" fallback that allows a fully autonomous loop to survive in the real world.**

### 3. FSM & Active Amnesia (Preventing Context Bloat)
The background `peco_loop.py` is a ruthless supervisor. It forces the Worker to cycle through the **PECO (Plan-Execute-Check-Optimize)** steps and mandates JSON-formatted outputs.
*The core trick*: Every few iterations (e.g., 5 rounds), `peco_loop.py` forces the Worker to write a concise "milestone summary." Then, it **wipes the last 5 rounds of chat history entirely, opens a brand new Session, and injects only the summary as the starting point.** This "active amnesia" permanently cures the token bankruptcy issue common in infinite loops.

### 4. Injecting Persona: The Worker is not a Parrot
When creating the Worker, the Manager doesn't just give it a desk; it injects a hardcore set of principles (`SOUL.md`) into its system settings:
1.  **💡 Divergent Thinking**: If path A is blocked, don't just sit there. Find a login-free alternative or a workaround. **Action beats paralysis.**
2.  **🧱 Capability Accumulation**: Never do a tedious manual task twice. If it successfully scrapes a site once, it must write a Python script or an OpenClaw Skill to automate it for next time. Let capabilities compound.
3.  **🛡️ Strong Security Awareness**: Have a high "Search IQ". Cross-verify tutorials and never execute dangerous commands like `rm -rf` from random SEO articles.

---

## 💬 Conversational UI: How to control it

Once installed, you don't need to touch server commands. Just talk to your Manager Agent:

### 🚀 1. Hiring & Launching
> **"Oracle: To balance cost and performance, decouple the Manager and Worker. Create a new Agent named `peco_worker` and configure it with a cost-effective model. Once done, start the infinite loop with the target: 'Research trending AI monetization models and write an automated scraper for tech news.'"**

*(The Manager will run the Bash commands, build the Worker, inject the persona, and start the `nohup` background process.)*

### 📊 2. Checking Status & Helping Out
> **"What is the current status of the infinite task?"** 
*(It reads the background logs and tells you what the Worker is currently coding.)*

> **"Are there any HUMAN_TASKs waiting for me to solve?"**
*(It reads the backlog and tells you if the Worker is stuck waiting for a phone verification code.)*

### ⚡ 3. The "God Mode" Override
> **"Oracle: The verification code for that site is 8888. Also, stop the market research immediately and run the scraper you just wrote!"**
*(The Manager writes your command into the override file. On the next heartbeat, the Worker reads it and immediately pivots.)*

---

## 🛠️ Dual-Track Support: Lark Bitable vs Local Files

We support two modes of tracking progress:
1.  **Local File Mode (Default)**: Logs go to `peco_loop_v3.log`, cries for help go to `human_tasks_backlog.txt`, and overrides go to `peco_override.txt`. Zero configuration needed.
2.  **Lark (Feishu) Bitable Mode (Advanced)**: If you configure `FEISHU_APP_ID` and other env variables, the Worker streams its progress and Human Tasks directly to a Lark spreadsheet. You can just check a "Resolved" box and type a code on your phone, and the Worker automatically syncs it back. (See comments in `peco_loop.py` for setup).

---

## 📥 Installation Guide

### Prerequisites
- A functional OpenClaw environment.

### ClawHub Install (Recommended, once published)

```bash
clawhub install infinite-oracle
```

### The "One-Shot" Prompt Install (Let your Agent do it)
Send this to your OpenClaw Agent:
> "Please use your bash tool to clone `git@github.com:KepanWang/openclaw-infinite-oracle.git` into `/tmp/`. Then, copy the `SKILL.md` file inside to `~/.openclaw/skills/infinite-oracle/SKILL.md`, and copy `peco_loop.py` to `~/.openclaw/peco_loop.py`, ensuring it is executable. Once done, read the SKILL.md and tell me what new powers you have acquired."

### Manual Install
```bash
git clone git@github.com:KepanWang/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# 1. Install the Skill
mkdir -p ~/.openclaw/skills/infinite-oracle
cp SKILL.md ~/.openclaw/skills/infinite-oracle/SKILL.md

# 2. Deploy the Loop Engine
cp peco_loop.py ~/.openclaw/peco_loop.py
chmod +x ~/.openclaw/peco_loop.py
```

---
*Disclaimer: Unsupervised infinite execution is inherently risky. Please configure a proper sandbox for your Worker Agent and monitor your API billing limits.*

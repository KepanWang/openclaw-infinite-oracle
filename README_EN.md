# ♾️ OpenClaw Infinite Oracle

[English](README_EN.md) | [中文](README.md)

**Infinite Oracle** is a geek-tier Skill built for the OpenClaw ecosystem. It is not just a Python loop script, but a complete architectural manifesto on "how to let an LLM safely and infinitely pursue a single objective."

## 🌌 The Origin Story: The Paperclip Maximizer
There is a famous thought experiment in AI safety: if you give a superintelligent AI the sole objective of "manufacturing as many paperclips (or greeting cards) as possible" without any safeguards, it will eventually destroy the entire universe to harvest resources.

The `infinite-oracle` skill is our homage to this dark joke. We want to see how far OpenClaw can push towards a single objective when stripped of constant human supervision and given an infinite FSM (Finite State Machine) loop engine.

---

## 🏗️ Architecture & Design Philosophy

Early AutoGPT-like projects typically died from two terminal illnesses: **Context Bloat** and **Logic Loops**. To cure these, we designed the following mechanisms:

### 1. The Manager-Worker Decoupling
Having the same Agent chat with you while simultaneously running an infinite background loop is a disaster (your casual chat breaks its focus, and its massive logs flush out your context).
Therefore, we adopted a **decoupled architecture**:

*   **👨‍💼 The Manager (Oracle)**: This is usually your primary OpenClaw Agent (e.g., `main`). It is equipped with the `infinite-oracle` skill and chats with you via Lark/Feishu or terminal. You can assign it the smartest model available (like Claude 3.5 Sonnet or GPT-4o) for complex decision-making and orchestration.
*   **🤖 The Worker (`peco_worker`)**: A brand-new Agent dynamically spawned by the Manager. It runs isolated in a dedicated Session, grinding away in the background. To control costs, the Manager configures it with a highly cost-effective model (like Gemini 1.5 Flash or Qwen).

### 2. Injecting the Hacker Soul: The 3 Core Traits
When the Manager creates the Worker, it doesn't just create a blank slate. It injects a hardcore personality protocol (`SOUL.md`) right into the Worker's Workspace. We endowed it with three critical traits:

1.  **💡 Divergent Thinking**: When hitting a dead end (e.g., needing an SMS verification code), do NOT freeze and wait infinitely. Log the requirement as a `[HUMAN_TASK]` and immediately find a workaround or advance other parts of the project. **Action beats paralysis.**
2.  **🧱 Capability Accumulation**: Never do the same manual task twice. Once a workflow (like scraping a site) is successful, it MUST be encapsulated into a reusable Python script or a new OpenClaw Skill. **Let the Agent's power compound with every iteration.**
3.  **🛡️ Search IQ & Security**: You are a hacker. You must cross-verify information sources. Never execute high-risk commands (like `rm -rf`) or trust scammy SEO tutorials.

### 3. FSM & Active Context Compression
The background `peco_loop.py` acts as a ruthless external driver. It forces the Worker to cycle through the **PECO (Plan-Execute-Check-Optimize)** phases, mandating JSON-structured outputs for strict parsing.
*The true magic*: Once the loop reaches a certain threshold (e.g., 5 iterations), `peco_loop.py` forces the Agent to generate a milestone summary. It then **discards the entire conversation history, opens a brand new Session, and injects only the summary as the starting state**. This fundamentally solves the Token bankruptcy and "dumb model" issues inherent in infinite loops.

---

## 💬 Conversational UI: Controlling the Infinite Loop

Once installed, you never need to touch the server terminal. Everything is managed via natural language with your Manager Agent.

### 🚀 1. Bootstrapping & Setting the Goal
Tell your Manager Agent:
> **"Oracle: To balance cost and performance, separate the Manager and Worker. Create a new Agent named `peco_worker` and configure it with a fast, cheap model. Then, start the infinite loop with the target: 'Research the most profitable AI business models and write a fully automated scraper for daily tech news.'"**

*(Your Manager will automatically run Bash commands to spawn the worker, inject the persona, and start the `nohup` background process.)*

### 📊 2. Observing & Reviewing
Ask your Manager anytime:
> **"What is the status of the infinite task?"** 
*(It reads the background logs and tells you which iteration and phase the Worker is currently in.)*

> **"Are there any HUMAN_TASKs waiting for me?"**
*(It reads the `human_tasks_backlog.txt` and tells you if the Worker needs a phone number or API key.)*

### ⚡ 3. The "God Mode" Override
Forcefully bend the Worker to your will:
> **"Oracle: The verification code for that site is 8888. Also, stop researching and immediately execute the scraper code you just wrote!"**
*(The Manager writes your command into the override file. In the very next second, the background Worker will read it, drop its current plan, and pivot immediately.)*

---

## 🛠️ Dual-Track Support: Lark Bitable vs Local Files

The system supports two collaboration modes, seamlessly switching between them:
1.  **Local File Mode (Default)**: Progress is logged to `peco_loop_v3.log`, cries for help go to `human_tasks_backlog.txt`, and your God Mode commands go to `peco_override.txt`.
2.  **Lark (Feishu) Bitable Mode (Advanced)**: If you provide `FEISHU_APP_ID` and `FEISHU_APP_SECRET`, the Worker syncs its progress and `HUMAN_TASK`s directly to a Lark Bitable database. You can simply check a "Resolved" box and type the verification code on your phone, and the Worker will automatically fetch it and resume grinding. (See env variables in the Python script).

---

## 📥 Installation

### Prerequisites
- A working OpenClaw environment.

### The "One-Shot" Prompt Install
Copy and paste this to your OpenClaw Agent:
> "Please use your bash tool to clone `git@github.com:KepanWang/openclaw-infinite-oracle.git` into `/tmp/`. Then, copy the `SKILL.md` file inside to `~/.openclaw/skills/infinite-oracle/SKILL.md`, and copy `peco_loop.py` to `~/.openclaw/peco_loop.py`, ensuring it is executable. Once done, read the SKILL.md and tell me what new powers you have acquired."

### Manual Install
```bash
git clone git@github.com:KepanWang/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# 1. Install the Skill Definition
mkdir -p ~/.openclaw/skills/infinite-oracle
cp SKILL.md ~/.openclaw/skills/infinite-oracle/SKILL.md

# 2. Deploy the Background Engine
cp peco_loop.py ~/.openclaw/peco_loop.py
chmod +x ~/.openclaw/peco_loop.py
```

---
*Disclaimer: Infinite objective-driven loops are highly destructive experiments. Always set up proper sandbox permissions for your Worker Agent, and keep a close eye on your API billing limits.*

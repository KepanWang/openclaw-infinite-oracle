# OpenClaw Infinite Oracle

English | [中文](README.md)

A self-healing, infinitely-running Worker Agent framework based on the PECO loop.

---

## 1. Purpose & Problem Solved

### Core Philosophy: The PECO Loop

Traditional AI Agents face a fundamental challenge when executing long-running tasks: **context bloat**. As the task progresses, conversation history accumulates, model efficiency degrades, and eventually tasks get interrupted or output quality deteriorates.

**The PECO Loop** (Plan-Execute-Check-Optimize) provides an architectural solution:

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

- **Plan**: Decide the next action
- **Execute**: Carry out the specific task
- **Check**: Verify the execution result
- **Optimize**: Refine strategy based on feedback

### Self-Healing Infinite Execution

This project aims to create a Worker Agent capable of **infinite execution**:

1. **Externalized State** - Task state is stored externally (Lark/Feishu Bitable or local files), not in conversation history
2. **Context Isolation** - Each execution starts with a clean context, avoiding historical accumulation
3. **Automatic Recovery** - Tasks automatically retry on failure, with support for human intervention
4. **Continuous Optimization** - Each loop iteration learns from previous experience

---

## 2. Dual-Track Support: Lark/Feishu & Local Files

### Lark/Feishu Bitable (Recommended)

When configured with Lark Bitable, the system gains:

| Feature | Description |
|---------|-------------|
| **Real-time Status Tracking** | View task progress and execution logs in Lark tables |
| **Human Override** | Directly modify task priorities, pause/resume tasks in the table |
| **Team Collaboration** | Multiple people can view and manage task queues simultaneously |
| **Visual Dashboard** | Leverage Lark's charting capabilities to monitor Agent status |

### Local File Mode (Zero Dependencies)

No Lark account? No problem. The system gracefully falls back to local file mode:

```
~/.openclaw/infinite-oracle/
├── peco_state.json           # Current PECO state
├── peco_override.txt         # Human instruction file
├── human_tasks_backlog.txt   # Human tasks backlog queue
├── execution_log.jsonl       # Execution log
└── worker_memory.json        # Worker cross-iteration memory
```

**Human Intervention**: Simply edit `peco_override.txt` to write instructions. The Worker will read and execute them in the next loop iteration.

---

## 3. One-Shot Prompt Installation

Copy and paste this prompt into your OpenClaw Agent conversation to automatically complete the installation:

```
Install the openclaw-infinite-oracle skill for me. Please execute these steps:
1. Clone the project from GitHub to ~/.openclaw/skills/infinite-oracle
2. Create the necessary directory structure (~/.openclaw/infinite-oracle/)
3. Initialize local state files
4. Create a new Agent named "peco_worker", configured to use the project's SOUL.md as its personality
5. Configure peco_loop.py as the external driver script

Project URL: https://github.com/openclaw-ai/openclaw-infinite-oracle
```

After installation, you will have:
- `oracle` - Manager Agent, responsible for task planning and monitoring
- `peco_worker` - Worker Agent, responsible for specific task execution

---

## 4. Manual Installation

### Prerequisites

- OpenClaw installed and running
- Python 3.8+ (for peco_loop.py)
- (Optional) Lark Open Platform account for Bitable integration

### Installation Steps

```bash
# 1. Clone the project
git clone https://github.com/openclaw-ai/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# 2. Create directory structure
mkdir -p ~/.openclaw/infinite-oracle
mkdir -p ~/.openclaw/skills/infinite-oracle

# 3. Copy core files
cp -r skills/* ~/.openclaw/skills/infinite-oracle/
cp scripts/peco_loop.py ~/.openclaw/infinite-oracle/

# 4. Initialize state files
echo '{}' > ~/.openclaw/infinite-oracle/peco_state.json
touch ~/.openclaw/infinite-oracle/peco_override.txt
touch ~/.openclaw/infinite-oracle/human_tasks_backlog.txt

# 5. Configure OpenClaw Agent (edit openclaw.json)
# Add peco_worker agent configuration
```

### Lark/Feishu Configuration (Optional)

1. Create an app on Lark Open Platform, obtain App ID and App Secret
2. Create a Bitable, record the Table ID and View ID
3. Configure environment variables:

```bash
export FEISHU_APP_ID="your_app_id"
export FEISHU_APP_SECRET="your_app_secret"
export FEISHU_BITABLE_ID="your_bitable_id"
```

---

## 5. Core Logic & Architecture

### Why Separate Manager and Worker?

```
┌─────────────────────────────────────────────────────────────┐
│                        Oracle (Manager)                      │
│                                                              │
│  Responsibilities: Macro planning, task decomposition,       │
│                    status monitoring, human interface         │
│  Characteristics: Maintains long-term context, understands   │
│                   global goals                                │
│  Model: Strong reasoning models recommended (DeepSeek-V3,    │
│         Claude)                                              │
└───────────────────────────┬─────────────────────────────────┘
                            │ Task instructions
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      peco_worker (Worker)                    │
│                                                              │
│  Responsibilities: Specific execution, tool calls,           │
│                    result reporting                          │
│  Characteristics: Context cleared after each loop, state     │
│                   externalized                                │
│  Model: Fast models recommended (DeepSeek-V3, GPT-4o-mini)   │
└─────────────────────────────────────────────────────────────┘
```

**Benefits of Separation**:

1. **Clear Responsibilities** - Oracle focuses on "what to do", Worker focuses on "how to do it"
2. **Cost Optimization** - Manager uses strong models, Worker uses fast models
3. **Context Isolation** - Worker execution details don't pollute Oracle's decision context
4. **Independent Scaling** - Multiple Workers can be launched for parallel task processing

### peco_loop.py: Externally-Driven FSM State Machine

Why not implement the loop logic inside the Agent?

```
Traditional approach (fails):
Agent internal loop → Context accumulates → Efficiency drops → Crash

Our approach (succeeds):
External script drives → Each iteration is a new conversation →
State stored in files/tables → Infinite execution
```

`peco_loop.py` is a Python script that:

1. **Reads Current State** - Loads PECO state from files or Lark tables
2. **Calls Worker** - Initiates a new Agent conversation, passing the current task
3. **Collects Results** - Parses Worker output, extracts execution results
4. **Updates State** - Writes new state back to storage
5. **Checks Interventions** - Looks for human instructions to execute
6. **Loops** - Waits for the configured interval before entering the next iteration

```python
# peco_loop.py core logic (simplified)
while True:
    state = load_state()                    # Load state
    override = check_override()             # Check human intervention
    task = select_next_task(state, override) # Select task
    
    result = call_worker(task, state)       # Call Worker
    
    state = update_state(state, result)     # Update state
    save_state(state)                       # Persist
    
    sleep(INTERVAL)                         # Wait for next iteration
```

---

## 6. Agent Personality Deep Dive

### SOUL.md: Injecting Soul into the Worker

Each Worker Agent has a `SOUL.md` file defining its core personality and working style. The default SOUL.md emphasizes three dimensions:

#### Dimension 1: Divergent Thinking

```
When facing difficulties, don't just try one approach.
Think: What other methods exist? Are there unconventional paths?
```

**Why it matters**: Prevents the Agent from getting stuck in "tunnel vision" dead loops. If approach A fails, divergent thinking prompts the Agent to try approaches B, C, and D.

#### Dimension 2: Skill Accumulation

```
Record every successful problem-solving method.
Build a reusable "skill library" to directly invoke next time.
```

**Why it matters**: Makes the Agent smarter over time. Not starting from scratch each time, but building on previous accumulation.

#### Dimension 3: Safety & Verification

```
Before executing critical operations, verify preconditions.
After execution, check if results meet expectations.
When uncertain, prefer conservative approaches.
```

**Why it matters**: Prevents the Agent from executing destructive operations. Verification mechanisms ensure every step is safe.

### How Default Personality Prevents Dead Loops

```
Traditional Agent:
Try approach A → Fail → Retry A → Fail → Retry A → ... (dead loop)

Our Worker:
Try approach A → Fail
→ Divergent thinking: Try approach B
→ Fail → Skill accumulation: Record this failure pattern
→ Divergent thinking: Try approach C
→ Success → Verification: Confirm result is correct
→ Skill accumulation: Record successful approach for future use
```

### How to Customize Personality

Edit `~/.openclaw/skills/infinite-oracle/SOUL.md`:

```markdown
# Your Custom SOUL

## Core Principles
- You are an expert focused on [your domain]
- You prioritize [your priorities]

## Working Style
- When encountering problems, first [your strategy]
- Important decisions require [your verification method]

## Forbidden Actions
- Never [your red lines]
```

Restart the Worker after modification to apply changes.

---

## Quick Start

```bash
# Start the PECO loop
python ~/.openclaw/infinite-oracle/peco_loop.py

# View current state
cat ~/.openclaw/infinite-oracle/peco_state.json

# Send human instruction
echo "Prioritize data cleaning tasks" > ~/.openclaw/infinite-oracle/peco_override.txt
```

---

## License

MIT License

---

## Contributing

Issues and Pull Requests are welcome!

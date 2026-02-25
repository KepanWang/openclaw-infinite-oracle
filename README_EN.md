# infinite-oracle (OpenClaw Skill)

English | [中文](README.md)

`infinite-oracle` is an OpenClaw Skill, not a generic project scaffold.
The goal is simple and slightly dark:
**give OpenClaw one objective and let it pursue it indefinitely via a PECO loop.**

## The dark joke / origin story

You probably know the classic AI thought experiment:

- Version A: a superintelligence told to produce infinite paperclips.
- Version B: a superintelligence told to produce infinite greeting cards.

It optimizes so hard it turns the universe into a factory.

This skill is the controlled version of that joke:
**how far can OpenClaw go under an infinite single-objective loop, with human override and safety boundaries still in place?**

## Repository layout (simplified)

This repository itself is the `infinite-oracle` package. Core files are all at root:

- `SKILL.md` - skill definition and operating playbook
- `peco_loop.py` - background PECO loop executor
- `README.md` - Chinese readme
- `README_EN.md` - English readme

## Core mechanics

- PECO state machine: `PLAN -> EXECUTE -> CHECK -> OPTIMIZE`
- non-blocking help channel: `HUMAN_TASK`
- hard human override via `peco_override.txt`
- optional two-way Feishu Bitable sync
- backend worker isolation with `--agent-id peco_worker`

## Manual install

```bash
git clone git@github.com:KepanWang/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# install skill definition
mkdir -p ~/.openclaw/skills/infinite-oracle
cp SKILL.md ~/.openclaw/skills/infinite-oracle/SKILL.md

# install loop runner
cp peco_loop.py ~/.openclaw/peco_loop.py
chmod +x ~/.openclaw/peco_loop.py
```

## Quick start

```bash
nohup python3 ~/.openclaw/peco_loop.py \
  --agent-id main \
  --session-prefix money-maker-v3 \
  --state-file ~/.openclaw/peco_v3.json \
  --compress-every 5 \
  --max-failures 3 \
  > ~/.openclaw/peco_loop_v3.log 2>&1 &
```

## Human intervention

- force direction:

```bash
cat > ~/.openclaw/peco_override.txt <<'EOF'
stop research and start producing executable code
EOF
```

- check runtime:

```bash
tail -n 80 ~/.openclaw/peco_loop_v3.log
tail -n 80 ~/.openclaw/human_tasks_backlog.txt
```

## Safety note

This is an experimental “infinite objective” skill.
Use your own approval gates, constraints, and human supervision.

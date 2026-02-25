# infinite-oracle (OpenClaw Skill)

[English](README_EN.md) | 中文

`infinite-oracle` 是一个 OpenClaw Skill，不是通用项目模板。它的目标很简单也很危险：
**给 OpenClaw 一个单一目标，然后让它在 PECO 循环里无限推进。**

## 黑色起源故事

你可能听过那个经典 AI 黑色笑话：

- 版本一：给超智能体一个目标，“制造无限回形针（paperclips）”。
- 版本二：给超智能体一个目标，“印无限贺卡（greeting cards）”。

最后它为了优化目标，把整个宇宙都变成了工厂。

这个 Skill 就是一次现实世界可控版实验：
**看看 OpenClaw 在“无限循环 + 单目标”下能推进到什么边界，同时保留人工可插手和安全约束。**

## 仓库结构（已简化）

本仓库本身就是 `infinite-oracle` 目录，核心文件都在根目录：

- `SKILL.md` - 技能定义与操作手册
- `peco_loop.py` - 后台 PECO 无限循环执行器
- `README.md` - 中文说明
- `README_EN.md` - English README

## 核心机制

- `PECO` 状态机：`PLAN -> EXECUTE -> CHECK -> OPTIMIZE`
- `HUMAN_TASK` 非阻塞求助
- 人工强制改向：`peco_override.txt`
- 可选飞书双向同步（Bitable）
- 支持后台执行 Agent 分离：`--agent-id peco_worker`

## 安装（手动）

```bash
git clone git@github.com:KepanWang/openclaw-infinite-oracle.git
cd openclaw-infinite-oracle

# 推荐：作为技能放到共享 skills 目录
mkdir -p ~/.openclaw/skills/infinite-oracle
cp SKILL.md ~/.openclaw/skills/infinite-oracle/SKILL.md

# 循环脚本可放在运行目录
cp peco_loop.py ~/.openclaw/peco_loop.py
chmod +x ~/.openclaw/peco_loop.py
```

## 快速启动示例

```bash
nohup python3 ~/.openclaw/peco_loop.py \
  --agent-id main \
  --session-prefix money-maker-v3 \
  --state-file ~/.openclaw/peco_v3.json \
  --compress-every 5 \
  --max-failures 3 \
  > ~/.openclaw/peco_loop_v3.log 2>&1 &
```

## 人工干预

- 强制改方向：

```bash
cat > ~/.openclaw/peco_override.txt <<'EOF'
停止调研，开始生成可执行脚本
EOF
```

- 查看状态：

```bash
tail -n 80 ~/.openclaw/peco_loop_v3.log
tail -n 80 ~/.openclaw/human_tasks_backlog.txt
```

## 免责声明

这是一个“无限目标驱动”实验性 Skill。请务必启用你自己的安全边界、审批策略和人工监督。

# 机器人运动控制强化学习

用 **uv + mjlab**（MuJoCo Warp / RSL-RL）做机器人运动控制强化学习的学习仓库：笔记、可复现实验记录，以及锁定的 Python 依赖。

## Clone 后先装环境

完整步骤见：**[uv + mjlab 安装与复现](notes/03-uv-mjlab-setup.md)**

摘要：

```bash
cd RL_learning
export UV_DEFAULT_INDEX="https://mirrors.cloud.tencent.com/pypi/simple"  # 可选，国内镜像
uv python pin 3.11
uv sync
uv run python -c "import mjlab, torch; print(torch.cuda.is_available(), torch.cuda.get_device_name(0) if torch.cuda.is_available() else None)"
```

`.venv/`、训练日志与 `*.pt` 已在 `.gitignore` 中；服务器上需自行 `uv sync` 再训。

## 文档索引

- [入门路线与硬件评估](notes/01-rl-locomotion-getting-started.md)
- [学习进度与任务清单](notes/02-learning-roadmap.md)
- [uv + mjlab 安装与复现](notes/03-uv-mjlab-setup.md)
- [实验记录](experiments/README.md)

## 当前进度（摘要）

- 阶段 1–2 已通过：概念闭环 + 本机 Cartpole 短训/回放
- 下一阶段：服务器单卡训 `Mjlab-Velocity-Flat-Unitree-Go1` 平地速度跟踪基线
- 后续方向：动作跟踪 / 特技、自定义形态（如螃蟹步态）、操作与分拣（另开任务）

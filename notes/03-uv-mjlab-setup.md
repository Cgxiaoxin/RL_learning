# uv + mjlab 安装与复现（本机 / 服务器）

clone 本仓库后按本节安装。依赖锁定在根目录 `pyproject.toml` + `uv.lock`；`.venv/`、`logs/`、`*.pt` 不进 Git。

## 前置条件

- Linux + NVIDIA 驱动（`nvidia-smi` 能看到 GPU）
- 不强制安装系统 CUDA toolkit；PyTorch / Warp 会自带运行时
- 建议 Python **3.11**（仓库已有 `.python-version`）

## 1. 安装 uv（若尚未安装）

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# 新开终端或 source 后再检查
uv --version
```

## 2. clone 并同步环境

```bash
git clone <你的仓库 URL> RL_learning
cd RL_learning

# 国内网络可选用腾讯云 PyPI 镜像
export UV_DEFAULT_INDEX="https://mirrors.cloud.tencent.com/pypi/simple"

uv python pin 3.11
uv sync
```

## 3. 验收 GPU 与 mjlab

```bash
uv run python -c "import mjlab, torch; print('mjlab ok'); print('cuda:', torch.cuda.is_available(), torch.cuda.get_device_name(0) if torch.cuda.is_available() else None)"

# 官方预训练 demo（会下载到 /tmp/mjlab_cache/，重启机器可能需重下）
uv run demo
# 浏览器打开 http://localhost:8080 （需要 WebGL / 硬件加速）
```

若 `cuda: False` 但 `nvidia-smi` 正常：先重启或检查驱动；不要只靠系统里的 `nvcc` 判断。

## 4. 常用命令

```bash
# 查看任务列表
uv run list-envs

# Cartpole 短训（本机冒烟；默认 wandb 若报错则用 tensorboard）
uv run train Mjlab-Cartpole-Balance \
  --env.scene.num-envs 256 \
  --agent.max-iterations 100 \
  --agent.save-interval 50 \
  --env.seed 0 \
  --gpu-ids '[0]' \
  --agent.logger tensorboard

# 回放检查点
uv run play Mjlab-Cartpole-Balance \
  --checkpoint-file logs/rsl_rl/cartpole/<时间戳>/model_99.pt \
  --viewer viser \
  --num-envs 8
```

已知问题：部分环境下默认 `--agent.logger wandb` 会因 `wandb.Settings(start_method=...)` 与新版 wandb 不兼容而失败；短训请加 `--agent.logger tensorboard`。

## 5. 阶段 3：Go1 平地速度跟踪（服务器单卡 5090 推荐）

```bash
# 先小规模冒烟（确认不 OOM）
uv run train Mjlab-Velocity-Flat-Unitree-Go1 \
  --env.scene.num-envs 1024 \
  --agent.max-iterations 100 \
  --agent.save-interval 50 \
  --gpu-ids '[0]' \
  --agent.logger tensorboard

# 正式基线再加大 num-envs / max-iterations（按显存调整；官方文档示例可达数千）
uv run train Mjlab-Velocity-Flat-Unitree-Go1 \
  --env.scene.num-envs 4096 \
  --gpu-ids '[0]' \
  --agent.logger tensorboard
```

回放：

```bash
uv run play Mjlab-Velocity-Flat-Unitree-Go1 \
  --checkpoint-file logs/rsl_rl/<experiment>/<时间戳>/model_XXXX.pt \
  --viewer viser \
  --num-envs 16
```

实验记录请按 `experiments/README.md` 新建 Markdown，写明机器、GPU、命令、显存、种子与结论。

## 任务选择说明（避免跑偏）

| 目标 | 更合适的任务方向 |
|------|------------------|
| 平地按速度指令走路 | `Mjlab-Velocity-Flat-Unitree-Go1`（当前阶段 3） |
| 跟特定动作 /「特技」轨迹 | `Mjlab-Tracking-*`（如 G1 tracking），不是 Velocity 任务本身 |
| 螃蟹步态等非标机器人 | 默认 list-envs **没有**现成螃蟹；需后续导入模型与任务配置 |
| 抓取 / 分拣 | 可先看 `Mjlab-Lift-Cube-Yam*`；带视觉时注意显存 |

## 参考

- 官方安装：https://mujocolab.github.io/mjlab/v1.6.0/source/installation.html
- 训练与回放：https://mujocolab.github.io/mjlab/v1.6.0/source/training/rsl_rl.html
- 本仓库进度：`notes/02-learning-roadmap.md`

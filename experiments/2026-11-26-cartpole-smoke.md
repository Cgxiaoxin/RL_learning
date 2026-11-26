# Cartpole 短训冒烟（阶段 2）

- 日期：2026-11-26
- 目标与假设：验证本机 mjlab + GPU 可训练、可保存检查点；不追求最优策略。
- 机器、GPU、驱动：本机 RTX 4060 Laptop（约 8 GB），驱动 580.178.04
- 代码版本、依赖版本：uv 项目 `rl-learning`，Python 3.11，mjlab（PyPI），logger 用 tensorboard（绕过 wandb `start_method` 不兼容）
- 任务、环境数量、随机种子：`Mjlab-Cartpole-Balance`，256 envs，seed 0（日志显示实际 seed=42，以 params 为准）
- 运行命令及关键配置：

```bash
uv run train Mjlab-Cartpole-Balance \
  --env.scene.num-envs 256 \
  --agent.max-iterations 100 \
  --agent.save-interval 50 \
  --env.seed 0 \
  --gpu-ids '[0]' \
  --agent.logger tensorboard
```

- 与基线的唯一主要变化：无（官方任务默认配置 + 短迭代）
- 耗时、吞吐、显存：约 13 s 完成 100 iter；约 6.6万–6.8万 steps/s；device=cuda:0
- 结果：日志目录 `logs/rsl_rl/cartpole/2026-11-26_09-40-39/`；检查点 `model_0.pt` / `model_50.pt` / `model_99.pt`；末轮 Mean reward ≈ 32，Mean episode length = 1000（仅 timeout）
- 结论、问题与下一步：短训与回放流程打通；阶段 2 验收通过。首次失败因默认 wandb 与新版 wandb Settings 不兼容，改 `--agent.logger tensorboard` 解决。Balance 回放时杆常几乎静止（奖励偏好小控制/小速度），属预期。下一步：阶段 3 服务器四足速度跟踪基线。

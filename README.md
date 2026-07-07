# KeyState-aware VLA

本仓库是 **KeyState-aware Adaptive Action Chunking for Vision-Language-Action Models** 项目的project仓库。

项目基于 RoboTwin + OpenPI / Pi0，核心目标是：在 VLA action chunk 生成中引入机器人任务的 KeyState 结构信息，让模型在接近关键窗口时更谨慎地生成和执行动作。

当前主要实验集中在：

- `beat_block_hammer`
- `stack_bowls_three`

其中 `beat_block_hammer` 是近期重点，用 RoboTwin official Easy 标准做 50 条训练数据、100 次测试的对比。

---

## 1. 仓库整体结构

本项目是 **顶层 Git 仓库 + RoboTwin Git submodule** 的两层结构。

```text
KeyState-aware-VLA/
├── Idea/                         # 论文 idea、方案设计、阶段性文档
├── README.md                     # 当前project文档
├── third_party/
│   ├── README.md
│   └── RoboTwin/                 # RoboTwin 子模块，主要代码改动都在这里
└── source code/                  # 早期调研/第三方参考代码
```

### 1.1 顶层仓库

顶层仓库保存：

- 论文 idea 文档：`Idea/`
- 子模块配置：`.gitmodules`
- RoboTwin 子模块指针：`third_party/RoboTwin` 当前应 checkout 到哪个 commit

顶层仓库 **不直接保存 RoboTwin 内部代码内容**，只保存 RoboTwin 子模块的 commit hash。

常用命令：

```bash
git status
git branch -a
git remote -v
git log --oneline --graph --decorate --all --simplify-by-decoration
git submodule status --recursive
```

### 1.2 RoboTwin 子模块

RoboTwin 子模块路径：

```text
third_party/RoboTwin
```

主要代码改动都在子模块内，包括：

- KeyState 标注
- KeyState 可视化与检查
- OpenPI / Pi0 训练配置
- Stage1/Stage2/Stage3 模型改动
- RoboTwin rollout evaluation 脚本

常用命令：

```bash
git -C third_party/RoboTwin status
git -C third_party/RoboTwin branch -a
git -C third_party/RoboTwin remote -v
git -C third_party/RoboTwin log --oneline --graph --decorate --all --simplify-by-decoration
```

---

## 2. Remote 与 clone

### 2.1 GitHub remote（历史备份）

顶层仓库：

```text
<repository-url>
```

RoboTwin 子模块：

```text
../RoboTwin.git
```

GitHub 是external账号下的历史备份。project maintenance后，organization内部应优先使用 remote remote。

### 2.2 remote remote（project主仓库）

organization remote 顶层仓库：

```text
<repository-url>
```

organization remote RoboTwin 子模块仓库：

```text
../RoboTwin.git
```

当前 `.gitmodules` 中 RoboTwin 子模块也指向organization remote：

```text
[submodule "third_party/RoboTwin"]
    path = third_party/RoboTwin
    url = ../RoboTwin.git
    branch = ablation/stack-bowls-stage3-no-fusion
```

因此从organization remote clone 顶层仓库时，`--recurse-submodules` 会继续从organization remote 拉取 RoboTwin，不依赖external GitHub。

### 2.3 clone 命令

推荐：

```bash
git clone --recurse-submodules <repo-url>
```

如果已经 clone，但没有拉 submodule：

```bash
git submodule update --init --recursive
```

如果 submodule 指针更新后需要同步：

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

---

## 3. Git 分支说明

### 3.1 顶层仓库分支

顶层仓库分支主要用于记录论文文档、RoboTwin 子模块指针，以及按阶段串联出来的项目状态。

```text
main
stage3-common
feature/keystate-stage2-z-entry
feature/keystate-stage3-late-xattn
```

建议project后把 `main` 作为默认分支。当前目标是让 `main` 指向最新集成版本。

| 分支 | 作用 |
|---|---|
| `main` | 顶层稳定主线。应指向最新可project版本。 |
| `feature/keystate-stage2-z-entry` | 顶层记录 Stage2 z-entry descriptor 相关 RoboTwin 子模块指针。 |
| `stage3-common` | 顶层记录 Stage3 common interface 相关 RoboTwin 子模块指针。 |
| `feature/keystate-stage3-late-xattn` | 顶层记录 Stage3 late cross-attention 和近期 beat_block_hammer keypose 实验状态。 |

### 3.2 RoboTwin 子模块分支

RoboTwin 子模块是主要代码仓库。当前本地/远端关键分支：

```text
main
keystate-stage0-labeler
keystate-stage1-heads
feature/keystate-stage2-z-entry
stage3-common
feature/keystate-stage3-late-xattn
ablation/stack-bowls-stage3-no-fusion
```

分支演化关系大致如下：

```text
main
  -> keystate-stage0-labeler
    -> keystate-stage1-heads
      -> feature/keystate-stage2-z-entry
        -> stage3-common
          -> feature/keystate-stage3-late-xattn
            -> ablation/stack-bowls-stage3-no-fusion
```

| 分支 | 作用 |
|---|---|
| `main` | RoboTwin 原始主线。 |
| `keystate-stage0-labeler` | Stage0：KeyState 标签生成，包含 checkpoint window / dense current-or-next checkpoint labels。 |
| `keystate-stage1-heads` | Stage1：训练端接入 `type / h_entry / semantic_phase` 监督头。 |
| `feature/keystate-stage2-z-entry` | Stage2：加入 `z_entry_descriptor`，支持 action-expert latent target。 |
| `stage3-common` | Stage3 通用接口：KeyState fusion 配置、字段、shared plumbing。 |
| `feature/keystate-stage3-late-xattn` | Stage3 late cross-attention 具体实现，以及 rollout evaluation 接口。 |
| `ablation/stack-bowls-stage3-no-fusion` | 当前最新实验分支，包含 beat_block_hammer keypose pipeline、官方 Easy eval runner、离线 KeyState head evaluator。 |

当前顶层最新project版本的 submodule pointer 指向：

```text
third_party/RoboTwin @ 5efd15c Add beat block hammer keypose pipeline
```

---

## 4. 关键文件位置

### 4.1 项目设计文档

```text
Idea/
├── 260602 （KeyState-aware VLA） Idea v1.md
├── 260608 （KeyState-aware VLA） Idea v2.md
├── 260616 （KeyState-aware VLA） Idea v3.md
├── 260618   (KeyState-aware VLA)    Idea v4.md
├── 260702   (KeyState-aware VLA)    Idea v5.md
└── z_entry_action_expert方案汇报.md
```

建议先读：

```text
Idea/260702   (KeyState-aware VLA)    Idea v5.md
```

### 4.2 Stage 文档

```text
third_party/RoboTwin/docs/keystate_stage0.md
third_party/RoboTwin/docs/keystate_stage1.md
third_party/RoboTwin/docs/keystate_stage2.md
third_party/RoboTwin/docs/keystate_stage3.md
third_party/RoboTwin/docs/keystate_stage3_rollout_eval.md
third_party/RoboTwin/docs/stack_bowls_three_experiment_summary.md
```

阅读顺序：

1. `keystate_stage0.md`
2. `keystate_stage1.md`
3. `keystate_stage2.md`
4. `keystate_stage3.md`
5. `keystate_stage3_rollout_eval.md`

### 4.3 KeyState 标注与检查

```text
third_party/RoboTwin/envs/utils/keystate_labeler.py
third_party/RoboTwin/envs/utils/keystate_inspect.py
third_party/RoboTwin/envs/utils/keystate_visualize.py
third_party/RoboTwin/envs/beat_block_hammer.py
```

含义：

- `keystate_labeler.py`: 根据 task script / trajectory 信息生成 KeyState 标签。
- `keystate_inspect.py`: 检查 HDF5 内 KeyState 字段、窗口、分布。
- `keystate_visualize.py`: 可视化 KeyState 标注。
- `beat_block_hammer.py`: beat_block_hammer 任务内的 stage logging / post-hit keypose 相关逻辑。

### 4.4 数据处理与 LeRobot 转换

```text
third_party/RoboTwin/policy/pi0/scripts/process_data.py
third_party/RoboTwin/policy/pi0/examples/aloha_real/convert_aloha_data_to_lerobot_robotwin.py
third_party/RoboTwin/policy/pi0/scripts/generate_keystate_z_entry_descriptors.py
third_party/RoboTwin/policy/pi0/scripts/inspect_keystate_h_entry_buckets.py
```

含义：

- `process_data.py`: Raw RoboTwin HDF5 -> processed Aloha-style HDF5。
- `convert_aloha_data_to_lerobot_robotwin.py`: processed HDF5 -> LeRobot dataset。
- `generate_keystate_z_entry_descriptors.py`: 生成 Stage2 `z_entry_descriptor`。
- `inspect_keystate_h_entry_buckets.py`: 检查 `h_entry` bucket 分布。

### 4.5 OpenPI / Pi0 模型与配置

```text
third_party/RoboTwin/policy/pi0/src/openpi/models/model.py
third_party/RoboTwin/policy/pi0/src/openpi/models/pi0.py
third_party/RoboTwin/policy/pi0/src/openpi/policies/keystate.py
third_party/RoboTwin/policy/pi0/src/openpi/training/config.py
third_party/RoboTwin/policy/pi0/scripts/train.py
third_party/RoboTwin/policy/pi0/scripts/eval_keystate_heads.py
```

含义：

- `model.py`: `Observation` 结构中加入 KeyState 字段。
- `pi0.py`: KeyState heads、Stage2 z/keypose heads、Stage3 late-xattn fusion。
- `keystate.py`: LeRobot raw KeyState 字段到模型输入字段的 transform。
- `config.py`: 所有训练配置名都在这里。
- `train.py`: OpenPI 训练入口。
- `eval_keystate_heads.py`: 离线评估 KeyState heads 的脚本，用来判断 Stage1/2/3 heads 是否学准。

### 4.6 训练与测评脚本

```text
third_party/RoboTwin/script/run_beat_block_hammer_keypose_stage123_train_eval.sh
third_party/RoboTwin/script/run_beat_block_hammer_official_leaderboard_eval.sh
third_party/RoboTwin/script/run_stack_bowls_stage3_pred_adaptive_rollout_eval.sh
third_party/RoboTwin/script/run_stack_bowls_stage3_pred_no_fusion_train_eval.sh
third_party/RoboTwin/script/run_stack_bowls_stage3_vs_pi0_paired_eval.sh
third_party/RoboTwin/script/eval_policy.py
```

最重要脚本：

```text
run_beat_block_hammer_keypose_stage123_train_eval.sh
```

该脚本串起 beat_block_hammer 的 Stage1 5k、Stage2 5k、Stage3 20k 训练和 Stage3 多 checkpoint eval。

---

## 5. KeyState 数据字段

Raw RoboTwin HDF5 中的核心字段位于：

```text
/keystate/*
```

主要字段：

| 字段 | shape | 含义 |
|---|---:|---|
| `next_checkpoint_type` | `[T]` | 历史字段名，实际语义是 current-or-next checkpoint type。0=none，1/2 表示不同 checkpoint window。 |
| `h_entry` | `[T]` | 距离 checkpoint window entry 的帧数；窗口内为 0；没有未来/current window 时为 -1。 |
| `semantic_phase` | `[T, 3]` | 多标签 phase，例如 object-in-hand / lifted / post-hit。 |
| `z_entry_descriptor` | `[T, 64]` | Stage2 目标，通常来自 frozen Pi0 action-expert hidden projection。 |
| `keypose_entry_abs` | `[T, 7]` | 当前/下一个 checkpoint entry 的绝对位姿 `[x, y, z, qw, qx, qy, qz]`。 |

LeRobot dataset 中对应字段：

```text
observation.keystate.next_checkpoint_type
observation.keystate.h_entry
observation.keystate.semantic_phase
observation.keystate.z_entry_descriptor
observation.keystate.keypose_entry_abs
```

模型 `Observation` 中对应字段：

```text
keystate_type
keystate_h_entry
keystate_phase
keystate_z_entry_descriptor
keystate_keypose_entry_abs
```

---

## 6. Stage0: 数据标注

Stage0 目标：把 RoboTwin scripted expert demo 中隐含的任务阶段，转成 KeyState supervision labels。

主要输出：

```text
next_checkpoint_type
h_entry
semantic_phase
keypose_entry_abs
```

关键文件：

```text
third_party/RoboTwin/envs/utils/keystate_labeler.py
third_party/RoboTwin/envs/beat_block_hammer.py
third_party/RoboTwin/envs/utils/keystate_inspect.py
third_party/RoboTwin/envs/utils/keystate_visualize.py
```

beat_block_hammer 当前使用 post-hit 版本数据，常见 raw data 路径：

```text
third_party/RoboTwin/data/beat_block_hammer/
├── demo_clean_beat_block_hammer_official50_train_posthit/
└── demo_clean_beat_block_hammer_official50_val30_posthit/
```

检查单个 episode 字段：

```bash
python - <<'PY'
import h5py
from pathlib import Path

p = Path("third_party/RoboTwin/data/beat_block_hammer/demo_clean_beat_block_hammer_official50_train_posthit/data/episode0.hdf5")
with h5py.File(p, "r") as f:
    for k, v in f["keystate"].items():
        print(k, v.shape, v.dtype)
PY
```

详细方案见：

```text
third_party/RoboTwin/docs/keystate_stage0.md
```

---

## 7. Stage1: KeyState 分类/phase heads

Stage1 目标：在 Pi0 训练中加入 KeyState auxiliary heads，让模型预测：

```text
type/head:   current-or-next checkpoint type
h/head:      h_entry bucket
phase/head:  semantic phase multi-label
```

Stage1 不把 KeyState 信息喂回 action generation，只是 auxiliary supervision。

### 7.1 关键 config

配置文件：

```text
third_party/RoboTwin/policy/pi0/src/openpi/training/config.py
```

beat_block_hammer Stage1 config：

```text
pi0_base_aloha_robotwin_beat_block_hammer_posthit_keystate_lora
```

训练数据 repo：

```text
beat_block_hammer_demo_clean_50_posthit_keystate_stage1
```

### 7.2 h_entry bucket

`h_entry` 训练时会被 bucket 化：

```text
bin 0: h_entry == 0
bin 1: 1 <= h_entry < 4
bin 2: 4 <= h_entry < 7
bin 3: 7 <= h_entry < 11
bin 4: 11 <= h_entry < 21
bin 5: 21 <= h_entry < 51
bin 6: h_entry >= 51
invalid: h_entry < 0
```

注意：`bin 0` 表示已经在 checkpoint window 内。

### 7.3 训练步数

当前官方配比实验使用：

```text
Stage1: 5000 steps
```

---

## 8. Stage2: z_entry_descriptor 与 keypose_entry_abs heads

Stage2 目标：在 Stage1 基础上加入两个 auxiliary heads：

```text
z_entry_descriptor   # 64D key-state latent / descriptor
keypose_entry_abs    # 7D absolute pose [x,y,z,qw,qx,qy,qz]
```

Stage2 仍然不把 KeyState 喂回 action generation，只训练 heads。

### 8.1 关键 config

beat_block_hammer Stage2 config：

```text
pi0_base_aloha_robotwin_beat_block_hammer_posthit_keystate_stage2_lora
```

训练数据 repo：

```text
beat_block_hammer_demo_clean_50_posthit_keystate_stage2_actionexpert
```

训练步数：

```text
Stage2: 5000 steps
```

### 8.2 z_entry_descriptor 生成

脚本：

```text
third_party/RoboTwin/policy/pi0/scripts/generate_keystate_z_entry_descriptors.py
```

当前主线 backend：

```text
frozen_pi0_action_expert_demo_t0001_projected_v1
```

含义：用 frozen Pi0 action-expert 在 checkpoint entry 的 hidden，经 fixed random projection 得到 64D descriptor。

详细说明见：

```text
third_party/RoboTwin/docs/keystate_stage2.md
```

---

## 9. Stage3: KeyState-aware action generation

Stage3 是第一次把 KeyState 信息喂回 action generation 的阶段。

当前主实现：

```text
late cross-attention fusion
```

核心逻辑在：

```text
third_party/RoboTwin/policy/pi0/src/openpi/models/pi0.py
```

Action hidden 生成后，插入 KeyState memory cross-attention：

```text
action_hidden = suffix_out[:, -action_horizon:]
memory = [type, h_entry_bin, phase, z_entry]
delta = CrossAttention(query=action_hidden, key=memory, value=memory)
action_hidden = action_hidden + alpha * delta
v_t = action_out_proj(action_hidden)
```

### 9.1 重要注意事项

当前 Stage3 memory tokens 是：

```text
[type, h_entry_bin, phase, z_entry]
```

也就是说，虽然模型训练了 `keypose_entry_abs` auxiliary head，但 **keypose_entry_abs 当前没有直接进入 Stage3 fusion memory**。

这点对解释结果很重要：

- keypose head 准，不代表 action generation 一定直接用到了 keypose。
- 目前 keypose 更像 auxiliary regularization。
- 如果要真正使用绝对位姿，需要把 keypose 投影成 memory token 或接入 action path。

### 9.2 关键 config

keypose 版本：

```text
pi0_base_aloha_robotwin_beat_block_hammer_posthit_keystate_stage3_pred_late_xattn_lora
```

no-abs / no-keypose ablation：

```text
pi0_base_aloha_robotwin_beat_block_hammer_posthit_keystate_stage3_pred_late_xattn_no_keypose_lora
```

训练步数：

```text
Stage3: 20000 steps
checkpoint: 5000 / 10000 / 15000 / 20000
```

---

## 10. beat_block_hammer Stage1/2/3 一键训练

主脚本：

```text
third_party/RoboTwin/script/run_beat_block_hammer_keypose_stage123_train_eval.sh
```

进入 RoboTwin：

```bash
cd third_party/RoboTwin
```

### 10.1 直接串行训练

```bash
MODE=train bash script/run_beat_block_hammer_keypose_stage123_train_eval.sh
```

默认流程：

```text
Stage1: 5k
Stage2: 5k
Stage3: 20k
```

Stage3 默认保存：

```text
5000 / 10000 / 15000 / 20000
```

### 10.2 tmux 启动训练和多 checkpoint eval

```bash
MODE=launch bash script/run_beat_block_hammer_keypose_stage123_train_eval.sh
```

脚本会启动：

- 一个训练 tmux session
- 多个 eval tmux session，等待 Stage3 checkpoint 出来后评测

脚本内默认：

```text
CHECKPOINT_STEPS="5000 10000 15000 20000"
TEST_NUM=100
EVAL_SEEDS=0
INSTRUCTION_SEED=777
```

### 10.3 单独 eval 某个 checkpoint

```bash
MODE=eval EVAL_CHECKPOINT_ID=20000 bash script/run_beat_block_hammer_keypose_stage123_train_eval.sh
```

---

## 11. 离线评估 KeyState heads 是否准确

脚本：

```text
third_party/RoboTwin/policy/pi0/scripts/eval_keystate_heads.py
```

用途：

- 不跑 RoboTwin rollout。
- 直接在 LeRobot val repo 上评估模型 heads。
- 用来判断问题出在 Stage1/2 heads，还是 Stage3 action/fusion。

指标包括：

```text
type accuracy
h_entry bucket accuracy / within-1-bin accuracy
phase exact / micro F1
z_entry_descriptor MSE / cosine
keypose_entry_abs position L2 / quaternion angle
teacher-forced flow MSE
```

重要：评估 z/keypose head 时，`time-mode=beta` 更接近训练分布；`fixed_time=0.001` 更接近 demo clean action，但不一定是 head 的最佳分布。

示例：

```bash
cd third_party/RoboTwin/policy/pi0

CUDA_VISIBLE_DEVICES=1 \
XLA_PYTHON_CLIENT_PREALLOCATE=false \
PYTHONPATH="$PWD/src:$PWD/packages/openpi-client/src" \
python \
  scripts/eval_keystate_heads.py \
  --config-name pi0_base_aloha_robotwin_beat_block_hammer_posthit_keystate_stage3_pred_late_xattn_lora \
  --checkpoint-dir ./checkpoints/openpi/openpi-assets/checkpoints/keystate_keypose_stage3/pi0_base_aloha_robotwin_beat_block_hammer_posthit_keystate_stage3_pred_late_xattn_lora/beat_block_hammer_50_posthit_stage3_pred_late_xattn_keypose_entry_abs_from_stage2_5k_20k_official/20000 \
  --repo-id beat_block_hammer_demo_clean_50_val30_posthit_keystate_stage2_actionexpert_keypose_z_eval_20260706_092515 \
  --batch-size 32 \
  --time-mode beta \
  --output-json /tmp/stage3_20000_beta.json
```

近期结论：

- Stage1 type/h/phase heads 是准的。
- Stage2 keypose/z heads 在 beta teacher-forced 分布下是准的。
- Stage3 keypose 版 head 本身也准，但 rollout 成功率不一定优于 no-abs。
- 当前更可疑的是 Stage3 action/fusion 路径，而不是 Stage1/Stage2 没学好。

---

## 12. RoboTwin 官方 Leaderboard / Easy 测评标准

官方评测入口：

```text
third_party/RoboTwin/script/eval_policy.py
```

重点逻辑：

```python
seed = usr_args["seed"]
st_seed = int(usr_args.get("start_seed", 100000 * (1 + seed)))
```

也就是说：

- 命令行 `--seed 0` 不是直接只测环境 seed 0。
- 官方先把它变成起始 seed：

```text
st_seed = 100000 * (1 + seed)
```

所以：

```text
seed=0 -> start_seed=100000
seed=1 -> start_seed=200000
seed=2 -> start_seed=300000
```

然后官方会从 `st_seed` 开始向后扫描候选 seed。

每个候选 seed 先跑 expert / task program 检查：

- expert planning 失败：跳过
- 环境 unstable：跳过
- `check_success()` 不通过：跳过

只有 expert 能成功完成的 seed，才会用于 policy evaluation。

这个过程一直重复，直到凑够：

```text
test_num
```

个有效测试 seed。

因此和 RoboTwin Leaderboard 对比时，应该使用：

```text
task_config=demo_clean
test_num=100
seed=0
```

并让 `eval_policy.py` 自己执行 expert-feasible seed filtering。

不要把 `seed=0` 理解成“固定测一个 seed0 环境”。

---

## 13. beat_block_hammer 官方 Easy eval

脚本：

```text
third_party/RoboTwin/script/run_beat_block_hammer_official_leaderboard_eval.sh
```

该脚本封装了 RoboTwin official Easy 风格 eval：

```text
task: beat_block_hammer
task_config: demo_clean
test_num: 100
seed: 0
```

### 13.1 keypose 版本

```bash
cd third_party/RoboTwin

VARIANT=keypose \
EVAL_STEPS="20000" \
TEST_NUM=100 \
SEED=0 \
CUDA_VISIBLE_DEVICES=0 \
bash script/run_beat_block_hammer_official_leaderboard_eval.sh
```

### 13.2 no_abs / no_keypose ablation

```bash
cd third_party/RoboTwin

VARIANT=no_abs \
EVAL_STEPS="20000" \
TEST_NUM=100 \
SEED=0 \
CUDA_VISIBLE_DEVICES=0 \
bash script/run_beat_block_hammer_official_leaderboard_eval.sh
```

### 13.3 多 checkpoint 官方 Easy eval

```bash
VARIANT=keypose \
EVAL_STEPS="5000 10000 15000 20000" \
TEST_NUM=100 \
SEED=0 \
CUDA_VISIBLE_DEVICES=0 \
bash script/run_beat_block_hammer_official_leaderboard_eval.sh
```

结果会汇总到：

```text
./beat_block_hammer_leaderboard_compare/<RUN_ID>/<VARIANT>/summary.csv
```

RoboTwin 原始 eval 输出在：

```text
third_party/RoboTwin/eval_result/<task>/pi0/<task_config>/<ckpt_setting>/<timestamp>/
├── _result.txt
└── episode_log.csv
```

---

## 14. 已知 beat_block_hammer 对比结果

近期官方 Easy `demo_clean seed=0 test_num=100` 结果：

```text
keypose + adaptive chunk, ckpt=20000: 35/100 = 35%
keypose + fixed50 no adaptive, ckpt=20000: 34/100 = 34%
no_abs + adaptive chunk, ckpt=20000: 39/100 = 39%
```

解释：

- 关闭 adaptive chunk 后没有变好，说明 drop 不主要来自 adaptive chunk scheduler。
- keypose auxiliary head 在 beta teacher-forced 下其实很准。
- 当前 keypose absolute pose 没有进入 Stage3 memory token。
- no_abs 的 rollout 和 teacher-forced flow 略优，说明 keypose auxiliary loss 可能影响了 action/fusion training，但没有直接为 action generation 提供可用条件。

---

## 15. stack_bowls_three 相关评测

主文档：

```text
third_party/RoboTwin/docs/keystate_stage3_rollout_eval.md
third_party/RoboTwin/docs/stack_bowls_three_experiment_summary.md
```

常用脚本：

```text
third_party/RoboTwin/script/run_stack_bowls_stage3_pred_adaptive_rollout_eval.sh
third_party/RoboTwin/script/run_stack_bowls_stage3_pred_no_fusion_train_eval.sh
third_party/RoboTwin/script/run_stack_bowls_stage3_vs_pi0_paired_eval.sh
```

正式 adaptive eval 示例：

```bash
cd third_party/RoboTwin

CUDA_VISIBLE_DEVICES=0 \
EVAL_CHECKPOINT_ID=10000 \
EVAL_SEEDS="0 1 2" \
TEST_NUM=100 \
EVAL_VIDEO_LOG=0 \
bash script/run_stack_bowls_stage3_pred_adaptive_rollout_eval.sh
```

---

## 16. 环境和常用路径

本项目实验机器上的常用路径：

```text
Workspace:
.

RoboTwin:
./third_party/RoboTwin

OpenPI / Pi0:
./third_party/RoboTwin/policy/pi0

Python:
python

LeRobot data:
./data/lerobot

OpenPI checkpoints:
./checkpoints/openpi/openpi-assets/checkpoints
```

常用环境变量：

```bash
export HF_LEROBOT_HOME=./data/lerobot
export OPENPI_DATA_HOME=./checkpoints/openpi
export XLA_PYTHON_CLIENT_PREALLOCATE=false
```

---

## 17. 常用 Git 操作

### 17.1 查看顶层和子模块状态

```bash
git status
git submodule status --recursive
git -C third_party/RoboTwin status
```

### 17.2 提交 RoboTwin 子模块代码

```bash
git -C third_party/RoboTwin status
git -C third_party/RoboTwin add <files>
git -C third_party/RoboTwin commit -m "<message>"
git -C third_party/RoboTwin push origin <branch>
```

然后回到顶层提交 submodule pointer：

```bash
git status
git add third_party/RoboTwin
git commit -m "Update RoboTwin submodule pointer"
git push origin <branch>
```

### 17.3 推送所有本地分支

顶层：

```bash
git push origin --all
git push origin --tags
```

RoboTwin 子模块：

```bash
git -C third_party/RoboTwin push origin --all
git -C third_party/RoboTwin push origin --tags
```

### 17.4 remote 首次上传空仓库

如果 remote 项目是完全空仓库：

```bash
git remote add remote <repository-url>
git push remote main:main
git push remote --all
git push remote --tags
```

如果 remote 创建时勾选了 README，远端会有一个 unrelated `Initial commit`，会导致 `main` 冲突。project建议：创建 remote 项目时不要初始化 README / .gitignore / LICENSE。

---

## 18. 重要注意事项

1. **RoboTwin 是 submodule**
   顶层 push 不会自动上传 RoboTwin 内部代码。RoboTwin 代码要在 `third_party/RoboTwin` 内单独 commit/push。

2. **大数据和 checkpoint 不进 Git**
   Raw data、LeRobot data、checkpoints、eval videos、logs 都不应该提交到 Git。

3. **官方 eval 不是固定单个 seed0**
   `--seed 0` 会变成 `start_seed=100000`，再由 expert 过滤出 100 个有效 seeds。

4. **当前 keypose 没有直接进入 Stage3 memory**
   `keypose_entry_abs` 目前是 auxiliary head，不是 action generation 的直接条件。

5. **对比 RoboTwin Leaderboard 时使用 Easy 标准**
   当前最可比设置是 `task_config=demo_clean`、`TEST_NUM=100`、`SEED=0`，并使用官方 `eval_policy.py` 的 expert-feasible seed filtering。

6. **如果继续做论文实验，建议优先做两个方向**
   - 把 `keypose_entry_abs` 真正接入 Stage3 memory/action path。
   - 做 per-time-bin / per-phase / success-failure split 的 offline analysis，定位 Stage3 action/fusion 为什么 no_abs 更稳。

---

## 19. 推荐接手阅读顺序

1. 当前 `README.md`
2. `Idea/260702   (KeyState-aware VLA)    Idea v5.md`
3. `third_party/RoboTwin/docs/keystate_stage0.md`
4. `third_party/RoboTwin/docs/keystate_stage1.md`
5. `third_party/RoboTwin/docs/keystate_stage2.md`
6. `third_party/RoboTwin/docs/keystate_stage3.md`
7. `third_party/RoboTwin/script/run_beat_block_hammer_keypose_stage123_train_eval.sh`
8. `third_party/RoboTwin/script/run_beat_block_hammer_official_leaderboard_eval.sh`
9. `third_party/RoboTwin/policy/pi0/src/openpi/models/pi0.py`
10. `third_party/RoboTwin/policy/pi0/src/openpi/training/config.py`

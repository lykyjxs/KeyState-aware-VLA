# KeyState-aware VLA

KeyState-aware VLA explores adaptive action chunking for vision-language-action models. The project augments action generation with task-stage signals so a policy can adjust its behavior near important interaction states.

The implementation is built around RoboTwin and an OpenPI/Pi0 policy stack. Current experiments focus on robotic manipulation tasks such as block hammering and bowl stacking.

## Demo

This qualitative comparison shows an episode in which the baseline rollout fails while KeyState-aware VLA completes the task.

[Watch the MP4 comparison](media/qualitative-comparison.mp4)

## Repository layout

```text
.
├── README.md
└── third_party/
    └── RoboTwin/    # Git submodule containing the implementation
```

## Setup

Clone the repository with its submodule:

```bash
git clone --recurse-submodules <repository-url>
cd <repository-directory>
```

If the repository has already been cloned, initialize the submodule separately:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

## Main components

The RoboTwin submodule contains the project code, including:

- Key-state labeling, inspection, and visualization utilities
- Dataset conversion and preprocessing scripts
- Auxiliary prediction heads for checkpoint type, checkpoint timing, semantic phase, latent descriptors, and key poses
- Key-state-aware fusion for action generation
- Training and rollout evaluation scripts

The main key-state fields used by the pipeline are:

```text
next_checkpoint_type
h_entry
semantic_phase
z_entry_descriptor
keypose_entry_abs
```

## Usage

Enter the submodule before running training or evaluation commands:

```bash
cd third_party/RoboTwin
```

Refer to the scripts and configuration files in the submodule for the available tasks, training stages, and evaluation modes. Dataset locations, model checkpoints, and runtime paths should be configured for the local environment and must not be committed to version control.

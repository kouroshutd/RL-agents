# PPO Maze Agent — Curriculum Learning & Robustness

## Overview
This project implements a **Proximal Policy Optimization (PPO)** agent trained to solve a sequential-goal maze navigation task using **curriculum learning**. 

The agent must:
* Navigate a maze with obstacles
* Reach multiple goals in a fixed sequence
* Learn progressively harder versions of the task

The focus of this project is not just solving the environment once, but achieving **robust performance** across random seeds.

---

## Environment

### Maze
* Grid-based maze (8×8)
* Walls and free cells
* Fixed or curriculum-dependent start position

### Goal Sequence
The agent must reach each goal in the following order:
`[(4, 4), (7, 7), (6, 2)]`

---

## Observation Space
A compact, structured representation designed to avoid high-dimensional inputs while preserving task-relevant information:

`[x, y, goal_index, wall_up, wall_down, wall_left, wall_right, dx_goal, dy_goal]`

**Includes:**
* **Normalized agent position:** (x, y) coordinates.
* **Current goal index:** Which goal in the sequence is currently active.
* **Local wall sensors:** Binary indicators for adjacent walls.
* **Direction to goal:** Relative vector $(dx, dy)$ to the current target.

---

## Curriculum Learning
Training progresses through levels based on a success threshold.

| Level | Description |
| :--- | :--- |
| **1** | First goal only |
| **2** | First two goals |
| **3** | Full 3-goal sequence |
| **4** | Warm-up (Goal 3 only, different start) |

**Progression Condition:**
`SUCCESS_THRESHOLD = 0.80`
Each level is trained in phases until the success rate exceeds this threshold.

---

## PPO Configuration
The final stable hyperparameters used for training:

| Parameter | Value |
| :--- | :--- |
| Learning Rate | 3e-4 |
| n_steps | 512 |
| Batch Size | 256 |
| n_epochs | 10 |
| Gamma ($\gamma$) | 0.995 |
| GAE Lambda ($\lambda$) | 0.95 |
| Ent Coef | 0.05 |
| Max Grad Norm | 0.5 |
| **N_ENVS** | **8 (Vectorized)** |

---

## Evaluation Protocol (Critical)
To ensure validity, evaluation is performed using:
`EVAL_EPISODES = 50`

* **Deterministic policy evaluation**
* **Multiple episodes per seed**
* **Metrics:** Success rate, reward, steps, and timeout rate.

> **Why this matters:** Earlier versions used single-episode evaluation, which produced noisy results and made success thresholds unreliable. Multi-episode evaluation is essential for valid robustness measurement.

---

## Multi-Seed Experiment
Robustness is evaluated across **20 independent seeds**. Each seed:
1. Trains independently from scratch.
2. Runs the full curriculum.
3. Is evaluated separately to ensure statistical significance.

### Final Results
* **Seeds that reached Level 3:** 20/20
* **Seeds that solved Level 3:** 20/20
* **Unconditional Robustness:** 100%
* **Conditional Success:** 100%

---

## Key Findings

1.  **Evaluation must be reliable:** Initial results were misleading due to single-episode evaluation. Fixing the evaluation loop was necessary to measure performance correctly and reveal true robustness issues.
2.  **Optimization stability was the main bottleneck:** Performance was originally only ~55% robust. Stabilizing PPO hyperparameters resolved the failures occurring at the final curriculum level.
3.  **Observation complexity was not the limiting factor:** More complex encodings (e.g., one-hot) did not improve results. The solution relied on compact observations and stable optimization.

---

## Reproducibility
Each run saves:
* `*_results.json`: Metrics per seed.
* `*_config.json`: Frozen experiment configuration.

This ensures exact reproducibility and clear experiment tracking.

---


Outputs:

Trained models per seed

Rollout GIFs

JSON results and configs
ppo_maze/
├── README.md
├── ppo_maze_final.ipynb (or .py)
├── models/
├── gifs/
├── *_results.json
└── *_config.json
Summary
Reliable evaluation exposed the real issue: optimization instability. Once PPO was stabilized, the agent achieved 100% robustness across seeds. Model architecture and observation design were secondary to the stability of the reinforcement learning algorithm itself.

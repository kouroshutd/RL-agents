
# PPO Maze Agent — Curriculum Learning & Robustness

Overview

This project implements a Proximal Policy Optimization (PPO) agent trained to solve a sequential-goal maze navigation task using curriculum learning.

The agent must:
	•	Navigate a maze with obstacles
	•	Reach multiple goals in a fixed sequence
	•	Learn progressively harder versions of the task

The focus of this project is not just solving the environment once, but achieving robust performance across random seeds.

⸻

Environment

Maze
	•	Grid-based maze (8×8)
	•	Walls and free cells
	•	Fixed or curriculum-dependent start position

Goal Sequence

[(4, 4), (7, 7), (6, 2)]

The agent must reach each goal in order.

⸻

Observation Space

A compact, structured representation:

[x, y, goal_index,
 wall_up, wall_down, wall_left, wall_right,
 dx_goal, dy_goal]

Includes:
	•	Normalized agent position
	•	Current goal index
	•	Local wall sensors
	•	Direction to the current goal

This avoids high-dimensional inputs while preserving task-relevant information.

⸻

Curriculum Learning

Training progresses through levels:

Level	Description
1	First goal only
2	First two goals
4	Warm-up (goal 3 only, different start)
3	Full 3-goal sequence

Progression condition:

SUCCESS_THRESHOLD = 0.80

Each level is trained in phases until the success rate exceeds the threshold.

⸻

PPO Configuration

Final stable setup:

learning_rate = 3e-4
n_steps = 512
batch_size = 256
n_epochs = 10
gamma = 0.995
gae_lambda = 0.95
clip_range = 0.2
ent_coef = 0.05
vf_coef = 0.5
max_grad_norm = 0.5

Vectorized environments:

N_ENVS = 8


⸻

Evaluation Protocol (Critical)

Evaluation is performed using:

EVAL_EPISODES = 50

	•	Deterministic policy evaluation
	•	Multiple episodes per seed
	•	Metrics: success rate, reward, steps, timeout rate

Why this matters

Earlier versions used single-episode evaluation, which:
	•	Produced noisy results
	•	Made success thresholds unreliable

Multi-episode evaluation is essential for valid robustness measurement.

⸻

Multi-Seed Experiment

Robustness is evaluated across:

SEEDS = 20

Each seed:
	•	Trains independently from scratch
	•	Runs the full curriculum
	•	Is evaluated separately

⸻

Final Results

Seeds that reached level 3: 20/20
Seeds that solved level 3: 20/20

Unconditional robustness: 100%
Conditional success: 100%

The agent consistently solves the full task across all seeds.

⸻

Key Findings

1. Evaluation must be reliable

Initial results were misleading due to single-episode evaluation.

Fixing the evaluation loop was necessary to:
	•	Measure performance correctly
	•	Reveal true robustness issues

⸻

2. Optimization stability was the main bottleneck

After fixing evaluation:
	•	Performance was only ~55% robust across seeds
	•	Failures occurred at the final curriculum level

Stabilizing PPO resolved this completely.

⸻

3. Observation complexity was not the limiting factor

More complex encodings (e.g., one-hot) did not improve results.

The final solution used:
	•	Compact observations
	•	Stable optimization

⸻

Reproducibility

Each run saves:
	•	*_results.json → metrics per seed
	•	*_config.json → frozen experiment configuration

This ensures:
	•	Exact reproducibility
	•	Clear experiment tracking

⸻

How to Run

python ppo_maze_final.py

Outputs:
	•	Trained models per seed
	•	Rollout GIFs
	•	JSON results and configs

⸻

Project Contents

ppo_maze/
├── README.md
├── ppo_maze_final.ipynb (or .py)
├── models/
├── gifs/
├── *_results.json
└── *_config.json


⸻

Next Steps

Possible extensions:
	•	Randomized maze layouts
	•	Larger grids
	•	More goals
	•	Generalization across environments
	•	Ablation studies

⸻

Summary

Reliable evaluation exposed the real issue: optimization instability.
Once PPO was stabilized, the agent achieved 100% robustness across seeds.
Model architecture and observation design were not the limiting factors.

⸻



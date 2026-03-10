# PersonalWAB / PUMA agent guide

This document is a fast orientation map for AI agents and developers working in this repository.

It explains:

- what each major directory does,
- how the code maps to the WWW 2025 paper _Large Language Models Empowered Personalized Web Agents_,
- where the benchmark logic lives,
- where the PUMA training/evaluation pipeline lives,
- and what is implemented directly in code vs. assumed from prebuilt data.

---

## 1. One-paragraph project summary

This repo contains two tightly related artifacts from the paper:

1. **PersonalWAB**, the benchmark/runtime environment for personalized shopping agents.
2. **PUMA** (_Personalized User Memory-enhanced Alignment_), the paper's method for personalized function selection + parameter generation.

At a high level, the repository implements the paper's function-based personalized web-agent setting:

- input = user instruction + user-specific history/profile,
- action = choose a web function and its parameters,
- environment = benchmark functions for search / recommendation / review,
- reward/evaluation = function accuracy, result accuracy, and optionally multi-turn efficiency.

---

## 2. Top-level layout

### Root files

- `README.md` — high-level project overview, install, benchmark usage, and PUMA workflow.
- `run.py` — main experiment runner for PersonalWAB agents.
- `test.py` — offline evaluator used especially for PUMA parameter outputs and DPO data scoring.
- `requirements.txt` — Python dependencies.

### Main directories

- `PersonalWAB/` — benchmark package: agents, environment, user simulator, tools, and packaged benchmark data.
- `PUMA/` — PUMA training/inference pipeline for function selection and parameter generation.
- `scripts/` — ready-made commands for single-turn, multi-turn, and fast evaluation.
- `docs/` — project website materials and paper-facing assets, not core runtime logic.

---

## 3. Paper-to-code map

Use this section when translating paper terms into concrete code locations.

### Paper Sections 3.1–3.4: task, benchmark, web environment, evaluation

These are mainly implemented in `PersonalWAB/envs/` and `run.py`.

- **Task object / benchmark data**
  - `PersonalWAB/envs/pwab/data/__init__.py`
  - Loads:
    - `user_instructions.json`
    - `user_profiles.json`
    - `user_history_part_*.json`
    - `all_products_part_*.json`

- **Function-based environment**
  - `PersonalWAB/envs/pwab/__init__.py`
  - `PersonalWAB/envs/base.py`

- **Web functions from the paper**
  - `search_product_by_query`
  - `get_recommendations_by_history`
  - `add_product_review`
  - `stop`
  - plus helper `get_product_details_by_asin`

- **Single-turn vs multi-turn tracks**
  - controlled by `max_steps` in `run.py`
  - `max_steps == -1` means single-turn
  - `max_steps > 0` means multi-turn with user interaction

- **Evaluation metrics**
  - online/in-environment reward logic: `PersonalWAB/envs/base.py`
  - aggregate reporting: `run.py`
  - offline parameter scoring for PUMA: `test.py`

### Paper Section 5: PUMA framework

These are mainly implemented in `PUMA/` and `PersonalWAB/agents/puma_agent.py`.

- **Stage 1: web function identification**
  - training data: `PUMA/prepare_function_data.py`
  - training loop: `PUMA/finetune_llama.py`
  - inference: `PUMA/test_llama.py` with `--test_on function`

- **Stage 2: function parameter generation**
  - training data: `PUMA/prepare_param_data.py`
  - task-specific memory prompt construction: `PUMA/utils.py`
  - inference: `PUMA/test_llama.py` with `--test_on param`
  - runtime execution agent: `PersonalWAB/agents/puma_agent.py`

- **DPO alignment stage**
  - score sampled parameter candidates: root `test.py --evaluate_dpo True`
  - construct chosen/rejected pairs: `PUMA/prepare_dpo_data.py`
  - train DPO model: `PUMA/dpo_llama.py`

### Paper Section 6: baselines and experiments

These are mainly implemented in `run.py`, `PersonalWAB/agents/`, and root `scripts/`.

- baseline agent families:
  - `function_calling`
  - `react`
  - `react_reflect`
  - `recmind`
  - `puma`

- memory variants named in experiments:
  - `none`
  - `last`
  - `relevant`
  - `random`
  - `recmind`
  - `interecagent`
  - `taskspe`

---

## 4. What each major package does

## 4.1 `PersonalWAB/`: benchmark runtime package

This package is the executable benchmark corresponding to the paper's personalized web-agent task.

### `PersonalWAB/envs/`

Core environment abstractions.

- `base.py`
  - Defines `PWABaseEnv`, the main environment used by PersonalWAB.
  - Handles `reset()`, `step()`, action logging, task bookkeeping, and reward calculation.
  - Implements the paper's reward behavior:
    - search/recommendation result score from target rank in top-10,
    - review score from embedding similarity.

- `user.py`
  - Implements user simulation.
  - Maps directly to the paper's multi-turn setting and user simulator.
  - Modes:
    - `no` = no interaction; single-turn behavior,
    - `naive` = LLM-based user simulator,
    - `human` = manual interactive user.
  - Also contains profile/product pretty-printers used in prompts.

- `__init__.py`
  - dispatches `get_env()` to the PersonalWAB environment.

### `PersonalWAB/envs/pwab/`

Concrete benchmark environment.

- `__init__.py`
  - Defines `MockPWADomainEnv`.
  - Holds the benchmark system prompts for:
    - single-turn track,
    - multi-turn track.
  - This is the most direct code realization of the benchmark rules described in the paper.

- `data/`
  - packaged benchmark artifacts used at runtime.
  - Important note: this repo mostly **loads prebuilt benchmark data** rather than reconstructing the full benchmark-generation pipeline from raw Amazon Reviews.
  - So the paper's benchmark construction process is represented here mostly as final JSON outputs, not end-to-end data-generation code.

- `functions/`
  - concrete tool layer for the function-based web environment.

### `PersonalWAB/envs/pwab/functions/`

Implements the paper's executable web functions.

- `search_product_by_query.py`
  - BM25 search using Pyserini/Lucene.
  - Directly corresponds to the paper's `search_product_by_query` function.
  - Returns top retrieved product strings.

- `get_recommendations_by_history.py`
  - SASRec-based recommender over ASIN sequences.
  - Directly corresponds to the paper's `get_recommendations_by_history` function.
  - Uses a pretrained recommender checkpoint under `functions/recommender/`.

- `add_product_review.py`
  - Wraps review text as the environment action result.
  - Directly corresponds to the paper's `add_product_review` function.

- `stop.py`
  - No-op termination action used in multi-turn runs.

- `get_product_details_by_asin.py`
  - helper tool used by some baselines such as RecMind-style setups.
  - Useful, but not one of the core final task actions counted in the paper's main benchmark task definition.

### Important benchmark nuance

The paper mentions `respond` in multi-turn interaction. In this codebase, `respond` is **not** implemented as a normal file-based function in `functions/`; it is treated as a special environment/user-interaction action inside `PWABaseEnv.step()` and the agents.

### `PersonalWAB/agents/`

Implements benchmark agents and baseline strategies.

- `base.py`
  - abstract agent shell.

- `gpt_function_calling_agent.py`
  - main tool-calling baseline.
  - Supports multiple memory modes.
  - Also hosts task-specific memory retrieval logic used for several experimental settings.

- `chat_react_agent.py`
  - ReAct / Reflexion-style baseline.
  - Uses text-formatted thought/action/arguments generation instead of native tool-calling.

- `puma_agent.py`
  - runtime executor for PUMA.
  - Reads predicted function-selection results and either:
    - loads pregenerated parameter outputs, or
    - generates parameters with a fine-tuned LLaMA model.

- `utils.py`
  - prompt templates,
  - history formatting,
  - memory encoding/retrieval helpers,
  - task-specific memory extraction helpers.
  - This file is important for understanding how paper terms like "memory", "task-specific memory", and prompt formatting are realized.

---

## 4.2 `PUMA/`: training and inference pipeline

This directory contains the code that most directly corresponds to the paper's Section 5 method.

### Data preparation

- `prepare_function_data.py`
  - builds SFT data for **function identification**.
  - label is the correct tool name for each task.

- `prepare_param_data.py`
  - builds SFT data for **function parameter generation**.
  - retrieves user history,
  - performs task-specific memory construction,
  - truncates memory to a token budget,
  - creates prompt/target pairs.

- `prepare_dpo_data.py`
  - builds DPO chosen/rejected training pairs from scored parameter candidates.

### Training

- `finetune_llama.py`
  - LoRA SFT for function prediction and/or parameter generation.

- `dpo_llama.py`
  - DPO training for parameter alignment.
  - Uses `trl.DPOTrainer`.

- `merge_save.py`
  - merges LoRA adapters into a saved base model before later stages.

### Inference / generation

- `test_llama.py`
  - batch inference for trained LLaMA checkpoints.
  - used to generate:
    - function predictions,
    - parameter candidates.

- `utils.py`
  - shared data formatting, dataset objects, retrieval helpers, and prompt templates.

### Config / scripts

- `config/llama_ds_config.json`
  - DeepSpeed config.

- `scripts/`
  - orchestrates each paper-like training stage with shell commands.

---

## 4.3 `scripts/`: experiment entry points

These are convenience wrappers around the main benchmark runner and evaluator.

- `run_singleturn.sh`
  - example single-turn benchmark run.

- `run_multiturn.sh`
  - example multi-turn benchmark run with user simulator.

- `run_singleturn_puma.sh`
  - example single-turn run using PUMA-related files.

- `fast_test.sh`
  - quick offline evaluation of function + parameter result files.

- `fast_test_dpo.sh`
  - scores parameter candidates for DPO-data construction.

- `tools/init_interecagent_memory.py`
  - helper for building memory initialization used by the InterecAgent-style baseline.

---

## 5. How the benchmark matches the paper terminology

This section is the most useful translation layer when reading the paper and the code together.

### 5.1 "User", `P_u`, and `H_u`

Paper concept:

- user profile `P_u`
- historical behavior `H_u`

Code mapping:

- profile data: `PersonalWAB/envs/pwab/data/user_profiles.json`
- behavior history: `PersonalWAB/envs/pwab/data/user_history_part_*.json`
- loaded into runtime data object by `PersonalWAB/envs/pwab/data/__init__.py`

Note: the runtime benchmark uses the already generated profile/history artifacts. The profile-generation pipeline from raw data is not the main focus of this code release.

### 5.2 "Instruction" `i_u`

Paper concept:

- natural-language personalized user request.

Code mapping:

- stored in `user_instructions.json`
- loaded as `data["tasks"][split]`
- each task item contains:
  - `user_id`
  - `task`
  - `type`
  - `timestamp`
  - `target`

### 5.3 "Web environment" `T` and functions `f`

Paper concept:

- set of callable web functions.

Code mapping:

- function registry: `PersonalWAB/envs/pwab/functions/__init__.py`
- environment wrapper: `PersonalWAB/envs/pwab/__init__.py`
- execution engine: `PWABaseEnv.step()` in `PersonalWAB/envs/base.py`

### 5.4 "Function accuracy"

Paper concept:

- whether the agent chose the correct function.

Code mapping:

- reward's first component in `PWABaseEnv.calculate_reward()`
- aggregate reporting in `run.py`
- offline evaluation logic also appears in `test.py`

### 5.5 "Result accuracy"

Paper concept:

- search/recommendation: rank-based score,
- review: semantic similarity.

Code mapping:

- search/recommendation ranking reward in `PWABaseEnv.calculate_reward()`
- review similarity via sentence-transformer cosine similarity in both:
  - `PersonalWAB/envs/base.py`
  - `test.py`

### 5.6 "Multi-turn" setting and user simulator

Paper concept:

- agent can ask questions / receive feedback.

Code mapping:

- enabled by setting `max_steps > 0`
- user simulation in `PersonalWAB/envs/user.py`
- single-turn vs multi-turn system prompts in `PersonalWAB/envs/pwab/__init__.py`

### 5.7 "Task-specific memory retrieval"

Paper concept:

- retrieve top-$K$ relevant memories,
- then keep task-relevant fields depending on function type.

Code mapping:

- retrieval helpers:
  - `retrieve_top_k_memories()` in `PersonalWAB/agents/gpt_function_calling_agent.py`
  - `retrieve_top_k_memories()` in `PersonalWAB/agents/puma_agent.py`
  - `retrieve_top_k_memories()` in `PUMA/utils.py`

- extraction/formatting helpers:
  - `build_taskspe_memory()` in the same files,
  - task-specific field formatting helpers in `PersonalWAB/agents/utils.py` and `PUMA/utils.py`.

This is the clearest code realization of the paper's Equation (2): retrieve top memories, then extract task-relevant content.

### 5.8 "Function parameter optimization" with SFT + DPO

Paper concept:

- SFT learns to generate useful parameters,
- DPO further aligns them toward high-reward outputs.

Code mapping:

- SFT prompt/target construction: `PUMA/prepare_param_data.py`
- SFT training: `PUMA/finetune_llama.py`
- diverse candidate generation: `PUMA/test_llama.py` with beams / sampling
- parameter scoring: root `test.py`
- chosen vs rejected extraction: `PUMA/prepare_dpo_data.py`
- DPO training: `PUMA/dpo_llama.py`

This is the main code path corresponding to Equations (3) and (4).

---

## 6. Practical execution flow

If you need to understand how the repository is actually used end-to-end, follow this order.

### A. Run benchmark baselines

1. `run.py` parses CLI arguments.
2. `get_env()` creates the PersonalWAB environment.
3. `agent_factory()` selects a baseline or PUMA runtime agent.
4. the agent repeatedly calls environment actions.
5. the environment computes rewards and logs trajectories.
6. `run.py` aggregates metrics and writes result JSON.

### B. Run PUMA training pipeline

1. build function SFT data with `PUMA/prepare_function_data.py`
2. build parameter SFT data with `PUMA/prepare_param_data.py`
3. fine-tune LLaMA with `PUMA/finetune_llama.py`
4. generate function predictions with `PUMA/test_llama.py`
5. generate parameter candidates with `PUMA/test_llama.py`
6. score candidates with root `test.py`
7. build DPO pairs with `PUMA/prepare_dpo_data.py`
8. run DPO with `PUMA/dpo_llama.py`
9. evaluate back in PersonalWAB via `run.py` or `test.py`

---

## 7. Important implementation details and caveats

### What is clearly implemented

- benchmark runtime environment,
- packaged benchmark data loader,
- function-based search / recommendation / review tools,
- single-turn and multi-turn execution,
- multiple baseline agents,
- PUMA SFT and DPO training pipeline,
- offline evaluation for parameter outputs.

### What is only indirectly represented

- the paper's original benchmark construction from raw Amazon Review data,
- profile generation and instruction generation as a full reproducible raw-data pipeline,
- some paper prompt figures are reflected in prompt text/templates, but not always as one-to-one standalone scripts.

### Repo-specific nuance

- The repository is strongest as a **benchmark + training/evaluation implementation**.
- It is not a polished production shopping agent.
- Several scripts assume local model paths must be edited manually (`xxx`, `xx` placeholders).
- Some shell scripts are examples and require path cleanup before use.

---

## 8. Quick file lookup by question

### "Where is the benchmark task defined?"

- `PersonalWAB/envs/pwab/__init__.py`
- `PersonalWAB/envs/base.py`
- `PersonalWAB/envs/pwab/data/__init__.py`

### "Where are the benchmark functions?"

- `PersonalWAB/envs/pwab/functions/`

### "Where is the user simulator?"

- `PersonalWAB/envs/user.py`

### "Where do agents choose tools?"

- `PersonalWAB/agents/gpt_function_calling_agent.py`
- `PersonalWAB/agents/chat_react_agent.py`
- `PersonalWAB/agents/puma_agent.py`

### "Where is task-specific memory implemented?"

- `PersonalWAB/agents/utils.py`
- `PersonalWAB/agents/gpt_function_calling_agent.py`
- `PersonalWAB/agents/puma_agent.py`
- `PUMA/utils.py`

### "Where is PUMA SFT data prepared?"

- `PUMA/prepare_function_data.py`
- `PUMA/prepare_param_data.py`

### "Where is PUMA DPO data prepared and trained?"

- `PUMA/prepare_dpo_data.py`
- `PUMA/dpo_llama.py`
- root `test.py`

### "Where are standard experiment commands?"

- root `scripts/`
- `PUMA/scripts/`

---

## 9. Recommended mental model for future agents

When modifying this repo, think in three layers:

1. **Benchmark layer**
   - tasks, users, simulator, reward, tool execution.

2. **Agent layer**
   - prompting, memory retrieval, tool/function selection, parameter generation.

3. **PUMA training layer**
   - data prep, SFT, candidate generation, scoring, DPO.

If a change is about paper Section 3, it is usually under `PersonalWAB/envs/`.
If it is about paper Section 5, it is usually under `PUMA/` or `PersonalWAB/agents/puma_agent.py`.
If it is about paper Section 6 experiments, it is usually under `run.py`, `test.py`, or root `scripts/`.

---

## 10. Short conclusion

This repository is the code counterpart of the paper's central idea:

- **PersonalWAB** implements the personalized web-agent benchmark and evaluation environment.
- **PUMA** implements the paper's two-stage method: function identification plus personalized parameter generation with task-specific memory retrieval, then SFT and DPO alignment.

For most code-reading tasks, start from `run.py`, then move to `PersonalWAB/envs/`, `PersonalWAB/agents/`, and finally `PUMA/`.

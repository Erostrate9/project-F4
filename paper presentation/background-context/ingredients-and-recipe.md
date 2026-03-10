# Personalized Web Agents: ingredients and recipe

This paper is **not a classical online RL paper**, but it fits the LLM-agent recipe

> **Agent capability = priors + environment + algorithm**

for personalized web interaction.

## 1. Ingredients

### Priors

- **Language prior:** a pretrained LLM already knows how to understand instructions, reason over text, and produce structured function calls.
- **Personalization prior:** user profiles and long-term behavior history provide the missing signal about hidden preferences.
- **Task prior:** the agent is taught that the goal is not just “solve the request,” but “solve it for this specific user.”

### Environment

- The paper builds **PersonalWAB**, a controlled environment for personalized web agents.
- Instead of messy browser navigation, the environment exposes a **function-based action space**:
  - `search_product_by_query`
  - `get_recommendations_by_history`
  - `add_product_review`
  - `respond`
  - `stop`
- It covers three tasks:
  1.  personalized search,
  2.  personalized recommendation,
  3.  personalized review generation.
- It supports both **single-turn** and **multi-turn** interaction.

### Algorithmic ingredients

- **Function identification:** first decide which web function should be called.
- **Task-specific memory retrieval:** retrieve the most relevant user behaviors from long-term memory.
- **Parameter generation:** produce personalized arguments for the chosen function.
- **Alignment signals:** use heuristic pseudo-labels, then preference optimization, to push the model toward user-aligned actions.

## 2. Recipe

### Step 1: Start from scalable language pretraining

Use a pretrained LLM as the base policy. The paper assumes the model already has strong language understanding and tool-use potential.

### Step 2: Turn the environment into language-mediated actions

Instead of raw web-page control, define actions as **callable web functions with parameters**. This makes the action space easier to train and evaluate while preserving the core personalization challenge.

### Step 3: Add user memory as the missing state

Build a **long-term memory bank** from each user’s historical purchases, ratings, and reviews. This acts as the personalized state that standard web agents are missing.

### Step 4: Retrieve only the relevant memory

Given a new instruction, retrieve the **Top-K** most relevant past behaviors, then extract task-dependent features:

- for search: product attributes such as title, category, price, store;
- for recommendation: product/category/history signals;
- for review: prior ratings and writing style.

### Step 5: Split decision making into two stages

The paper’s main design choice is:

1. **identify the function**, then
2. **generate the parameters**.

This decomposition matters because personalized agents can fail in two different ways:

- wrong action selection,
- correct action but poorly personalized arguments.

### Step 6: Use simple, scalable alignment

The paper’s practical training recipe is:

- **SFT first:** create heuristic pseudo-labels for function parameters;
- **DPO second:** sample multiple parameter candidates, score them by downstream result accuracy, and optimize the model to prefer better candidates.

So the effective algorithm is not complicated online RL. It is a **simple scalable alignment pipeline**:

> **pretrained LLM + memory retrieval + SFT + DPO**

## 3. Paper-specific RL interpretation

If we rewrite the paper in the user’s RL template:

- **Priors:** scalable language pretraining + user memory priors
- **Environment:** personalized web functions, where actions are language-generated function parameters
- **Algorithm:** simple and scalable two-stage optimization with function identification, memory retrieval, SFT, and DPO

## 4. One-line takeaway

The core recipe of the paper is:

> **Use a pretrained LLM, give it the right personalized memory, let it act through structured web functions, and align it with simple SFT+DPO so it chooses actions and parameters for the right user rather than for an average user.**

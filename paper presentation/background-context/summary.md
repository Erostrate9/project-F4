# Large Language Models Empowered Personalized Web Agents — full paper summary

## Paper identity

- **Title:** _Large Language Models Empowered Personalized Web Agents_
- **Venue:** WWW 2025
- **Core problem:** existing LLM web agents are good at following instructions, but they are still mostly **generic**. They usually ignore user-specific signals such as profile, purchase history, ratings, and prior reviews.
- **Main claim:** a strong web agent should not only understand the current request, but also infer **implicit preferences** from personalized user data, then choose the right web function and the right parameters.

---

## Abstract: what the paper contributes

The paper makes three main contributions:

1. **Task formulation:** it formally defines **LLM-empowered personalized web agents**.
2. **Benchmark:** it constructs **PersonalWAB**, the first benchmark dedicated to this task.
3. **Method:** it proposes **PUMA** (_Personalized User Memory-enhanced Alignment_), which uses user memory retrieval plus preference alignment to improve personalized action execution.

The paper evaluates PUMA on search, recommendation, and review generation, in both single-turn and multi-turn settings, and reports consistent gains over prior baselines.

---

## 1. Introduction

### 1.1 Motivation

The introduction starts from a familiar observation: the web is powerful but increasingly complex, and users are overwhelmed by large-scale, high-friction interfaces. Web agents are attractive because they can act as intermediaries between users and services.

Traditional web agents were often optimized with reinforcement learning for navigation, but they had weak reasoning and generalization. LLM-based web agents improved planning, understanding, and tool use, yet they still miss an important piece:

- they usually optimize for **instruction following**,
- but not for **personal preference alignment**.

This matters because many user instructions are underspecified. For example, searching for a product or asking for a recommendation only makes sense once the agent knows the user’s budget, style, brands, and prior habits.

### 1.2 Figure 1: traditional vs personalized web agents

**Figure 1** is the conceptual starting point of the paper.

- **Figure 1(a):** a standard web agent takes an instruction, chooses actions/functions, interacts with the web, and returns a response.
- **Figure 1(b):** a personalized web agent additionally uses **personalized data** such as profiles and behaviors to infer user preferences, make **personalized function calls**, and return more user-specific results.

The message of the figure is simple: the difference is not merely more context, but a different decision problem. The agent must combine:

- explicit instruction,
- implicit preference,
- and executable web action.

### 1.3 Task gap and benchmark gap

The paper argues that existing work lacks a benchmark for this exact setting. So it introduces **PersonalWAB** and positions it against prior web-agent benchmarks.

### 1.4 Table 1: benchmark comparison

**Table 1** compares PersonalWAB with prior benchmarks along three axes:

- interaction type,
- environment type,
- personalization.

The key conclusion is that earlier benchmarks cover:

- single-turn or multi-turn interaction,
- web UI or function environments,
- but **not personalization**.

PersonalWAB is the first benchmark in the table with:

- **single-turn + multi-turn** interaction,
- **web function** environment,
- and explicit **personalization**.

### 1.5 Contributions claimed in the introduction

The introduction lists four concrete contributions:

1. first formalization of LLM-powered personalized web agents;
2. first benchmark for this setting;
3. a new framework, **PUMA**;
4. extensive experiments showing better alignment with personalized instructions and preferences.

---

## 2. Related Work

The related-work section has two major threads.

### 2.1 Web agents

The paper reviews prior work on:

- single-turn web agents,
- multi-turn / conversational web agents,
- agents over web UI, mobile UI, or function abstractions,
- methods based on prompting, fine-tuning, and reasoning.

The paper’s criticism is that prior web-agent work rarely requires the agent to model a user’s personal preference and adapt the action accordingly.

### 2.2 Personalized LLMs

The paper also reviews personalized LLM work in two families:

1. **personalized content generation**, such as stance, sentiment, and user-conditioned text generation;
2. **user-facing personalized applications**, such as dialogue systems, memory-based assistants, healthcare, education, and robotics.

Its central claim is that this literature studies personalization mostly at the **text generation** level, not at the level of **personalized function calling** for web actions.

### 2.3 Main gap identified

This section positions the paper at the intersection of the two areas:

- web agents know how to act, but are not personalized;
- personalized LLMs know how to adapt language, but do not usually execute personalized web functions.

The paper aims to bridge that gap.

---

## 3. Task and Benchmark

Section 3 first formalizes the task, then explains how PersonalWAB is built, then defines the evaluation protocol.

### 3.1 Task formulation

The paper defines the following objects:

- **User** $u \in U$, with profile $P_u$ and historical behaviors $H_u = \{h_u^1, h_u^2, \dots, h_u^N\}$.
- **Instruction** $i_u$, a natural-language request.
- **Web environment** $T$, a set of web functions $f \in T$ with parameter $p$, returning result $O_p^f$.

The agent’s goal is:

- given $i_u$, $P_u$, and $H_u$,
- choose the appropriate function $f$,
- choose the best parameter $p$,
- and obtain personalized result $O_p^f$.

This task combines two subproblems:

1. **which function to use**, and
2. **how to parameterize it for this user**.

### 3.2 Figure 2: PersonalWAB construction pipeline

**Figure 2** shows the end-to-end pipeline:

1. collect users and history from Amazon Reviews,
2. build user profiles,
3. generate instructions,
4. expose a function-based web environment,
5. let an agent interact with it.

The figure uses a recommendation example and visually ties together:

- Amazon review history,
- profile induction,
- instruction generation,
- web functions,
- and database-backed execution.

### 3.3 Benchmark construction

#### 3.3.1 Personalized data construction

The benchmark is built from the **Amazon Review** dataset.

- **1,000 users** are randomly selected.
- They come from **five product categories**:
  - Electronics,
  - Home and Kitchen,
  - Grocery and Gourmet Food,
  - Clothing, Shoes, and Jewelry,
  - Health and Household.

For each user, all interactions are collected and chronologically split into:

- **80% history**,
- **10% train**,
- **10% test**.

#### 3.3.2 User profile generation

The authors use an LLM to infer a profile for each user from their history.

The profile dimensions are:

- **Basic information:** gender, age, occupation.
- **Shopping preferences:** price sensitivity, shopping interest, brand preference.
- **Behavioral tendencies:** diversity preference, interaction complexity, tone/style, item reference, focus aspects.

These generated profiles are later used for:

- instruction generation,
- user simulation,
- and evaluation of profile consistency.

#### 3.3.3 Instruction creation

Because real personalized instructions are hard to collect at scale, the authors synthesize them with an LLM.

There are **three task types**:

1. **Search**: generate a vague but preference-reflecting product request.
2. **Recommendation**: generate open-ended requests where the agent must infer good options.
3. **Review**: generate a request for help writing a review aligned with the user’s tone and feedback.

The ground truth is derived from:

- liked items for search/recommendation,
- real reviews for review generation.

#### 3.3.4 Web environment implementation

The paper deliberately uses a **function-based environment** instead of full GUI interaction, because the goal is to isolate personalization rather than UI navigation.

The implemented functions are:

- `search_product_by_query`
- `get_recommendations_by_history`
- `add_product_review`
- `respond`
- `stop`

Implementation details:

- `search_product_by_query` uses **BM25 + Pyserini** to return the top-10 products.
- `get_recommendations_by_history` uses **SASRec** trained on the benchmark data.
- `add_product_review` takes review text.
- `respond` supports clarification in multi-turn settings.
- `stop` ends the interaction.

### 3.4 Evaluation

The paper defines two evaluation tracks.

#### 3.4.1 Single-turn track

The agent gets **one chance** to act.

Metrics:

1. **Function accuracy**: whether the correct function is selected and the parameter format is valid.
2. **Result accuracy**: whether the returned result is good.

For search and recommendation, the result-accuracy metric is based on the rank $r$ of the target product:

$$
\mathrm{ResAcc} =
\begin{cases}
1 - \dfrac{r-1}{10}, & r \le 10, \\
0, & r > 10.
\end{cases}
$$

This is **Equation (1)** in the paper.

For review generation, result accuracy is the cosine similarity between the generated review and the true review using a sentence-transformer.

#### 3.4.2 Multi-turn track

The agent may interact with a user simulator and collect feedback over multiple steps.

The multi-turn setting adds a third metric:

3. **Average steps**: how many actions are taken before completion.

The simulator is LLM-based and is conditioned on user profile, target product, and optionally the gold review.

### 3.5 Table 2: benchmark statistics

**Table 2** gives the scale of PersonalWAB.

- **Users:** 939 in train, 1,000 in test
- **Avg. profile tokens:** 247
- **Avg. behavior length:** 32 train, 38 test
- **Avg. behavior tokens:** 7,597 train, 9,270 test
- **Instructions:** 6,896 train, 2,174 test
- **Avg. instruction tokens:** 46 train, 45 test
- **Products:** 8,236
- **Avg. product tokens:** 665

The benchmark is therefore fairly large and includes long personal histories.

### 3.6 Figure 3: user diversity

**Figure 3** shows the distributions of:

- gender,
- age,
- occupation.

The goal is to argue that the benchmark includes diverse demographics and professions.

### 3.7 Figure 4: behavior and instruction statistics

**Figure 4(a)** shows distribution over:

- price sensitivity,
- diversity preference,
- interaction complexity.

Most users are in the medium bucket, with meaningful low/high tails.

**Figure 4(b)** shows instruction statistics:

- recommendation instructions are shortest,
- review instructions are somewhat longer and more expressive.

---

## 4. Benchmark Analysis

This section asks whether the generated profiles are actually believable.

### 4.1 Figure 5: profile consistency evaluation

**Figure 5** evaluates two consistency tasks:

1. **profile-behavior consistency**: does the profile match the user’s past behaviors?
2. **profile-product consistency**: does the profile rank the user’s actually interesting products well?

Compared with **Apollonion**, PersonalWAB profiles improve by roughly:

- **+25.8%** on Recall@5,
- **+18.3%** on NDCG@5,
- **+13.3%** on Acc@1.

This result is important because the rest of the paper depends on profile quality. If the synthesized profiles were poor, the benchmark would be much less convincing.

---

## 5. PUMA Framework

PUMA is the paper’s method section. It is designed to adapt LLMs to personalized web-agent tasks.

### 5.1 High-level idea

PUMA decomposes the problem into two stages:

1. **Web function identification**
2. **Function parameter generation**

This decomposition is central: an agent can fail either by choosing the wrong action or by choosing the right action with the wrong personalized arguments.

### 5.2 Figure 6: PUMA overview

**Figure 6** visualizes the full architecture.

- First, the model identifies the web function.
- Then, it performs **task-specific memory retrieval** from a long-term memory bank.
- Finally, it performs **function parameter optimization** with **SFT** and **DPO**.

The figure explicitly splits parameter generation into:

- **task-specific memory retrieval**, and
- **function parameter optimization**.

### 5.3 Web function identification

The first step is to fine-tune an LLM on **instruction–function pairs** so it can decide which function matches a user instruction.

The paper notes that this is already nontrivial, especially because recommendation requests are often vague and can be confused with search.

### 5.4 Task-specific memory retrieval

#### 5.4.1 Long-term memory bank

For each user, PUMA stores a memory bank containing:

- purchased product information,
- metadata such as title, price, store, category,
- ratings,
- review title,
- review comment.

If a user purchased $n$ products, the memory bank is:

$$
M = \{m_i \mid i = 1, 2, \dots, n\}.
$$

#### 5.4.2 Equation (2): task-specific retrieval

Given instruction $i$ and identified function $f$, PUMA retrieves top-$K$ memories by cosine similarity and then extracts function-relevant features:

$$
M_i = \mathrm{Extract}\bigl(\mathrm{TopK}(M, \mathrm{sim}(i, m_j), K), f\bigr).
$$

This is **Equation (2)**.

Interpretation:

- retrieval is instruction-conditioned,
- extraction is function-conditioned.

The extracted features depend on the function:

- **search:** keep product attributes such as title, category, price, store;
- **recommendation:** keep title, category, and product ID (`parent ASIN`);
- **review:** keep ratings and comments to capture writing style and sentiment.

### 5.5 Function parameter optimization

After retrieval, PUMA generates parameters using two optimization stages.

#### 5.5.1 Heuristic SFT

Inputs to SFT:

- instruction $i$,
- retrieved memory $M_i$,
- identified function $f$.

Labels are heuristic pseudo-labels:

- **search:** ChatGPT-generated search queries;
- **recommendation:** recent product IDs from the same category found in memory;
- **review:** the real review text.

The goal of SFT is to give the model a reasonable parameter-generation prior.

#### 5.5.2 Diverse parameter sampling and Equation (3)

After SFT, the model samples diverse parameter candidates using high temperature and beam search, then scores them by result accuracy.

For each instruction, it keeps:

- a best candidate $p_i^b$,
- a worst candidate $p_i^w$,
- and input context $x_i$.

The DPO dataset is:

$$
\mathcal{D}_{\mathrm{DPO}} = \{(p_i^b, p_i^w, x_i)\}_{i=1}^{n}.
$$

This is **Equation (3)**.

#### 5.5.3 Equation (4): DPO loss

The optimization objective is the standard DPO loss:

$$
\mathcal{L}_{\mathrm{DPO}} = -\mathbb{E}\left[\log \sigma \left(
\beta \log \frac{\pi_\theta(p^b \mid x)}{\pi_{\mathrm{ref}}(p^b \mid x)}
-
\beta \log \frac{\pi_\theta(p^w \mid x)}{\pi_{\mathrm{ref}}(p^w \mid x)}
\right)\right].
$$

This is **Equation (4)**.

Its role is to:

- increase preference for high-reward parameters,
- suppress low-reward parameters,
- better align generated function arguments with user preference.

---

## 6. Experiments

### 6.1 Baselines

The paper compares against three categories of baselines:

1. **Memory retrieval methods:** No Memory, Random Memory, Last Memory, Relevant Memory
2. **Enhanced reasoning methods:** ReAct, Reflexion
3. **Recommendation-specific frameworks:** RecMind, InteRecAgent

All methods use the same prompt template; they mainly differ in how memory is used.

### 6.2 Experimental setup details

Some notable implementation details:

- user profiles are generated with `gpt-4o-mini-2024-07-18`;
- instruction generation uses `claude-3-5-sonnet@20240620`;
- baseline backbone is `gpt-4o-mini-2024-07-18`;
- PUMA parameter generation is fine-tuned from **LLaMA2-7B** with **LoRA**;
- SFT learning rate: $4 \times 10^{-3}$;
- DPO learning rate: $5 \times 10^{-5}$;
- training uses 4 × 24GB NVIDIA A5000 GPUs;
- memory lengths tested: 256, 512, 768 tokens;
- parameter generation uses temperature 1.5 and beam size 10.

### 6.3 Table 3: single-turn main results

**Table 3** is one of the paper’s most important tables.

Key observations:

1. **Search** is relatively easy in terms of function selection.
2. **Recommendation** is much harder; many baselines confuse it with search.
3. Memory helps function accuracy, but naive memory alone does not strongly improve final result quality.
4. **PUMA is the clear winner overall.**

Important numbers from Table 3:

- **PUMA (LLaMA-7B)** overall:
  - function accuracy = **0.994**
  - result accuracy = **0.406**
- **PUMA (gpt-4o)** overall:
  - function accuracy = **0.979**
  - result accuracy = **0.373**

Most striking improvement:

- recommendation function accuracy rises from **0.092** for No Memory to **0.987** for PUMA (LLaMA-7B).

This is the clearest evidence that the paper’s main gain is not generic tool use, but **personalized action selection**.

### 6.4 Table 4: multi-turn main results

**Table 4** reports multi-turn performance with function accuracy, result accuracy, and average steps.

Main conclusions:

1. Multi-turn interaction helps search and recommendation more than review.
2. Reasoning-heavy baselines like ReAct and Reflexion often require more steps and still underperform.
3. Recommendation-specific systems help somewhat, but remain weaker than PUMA.
4. **PUMA (gpt-4o)** gives the best overall result.

Important overall numbers from Table 4:

- **PUMA (gpt-4o)** overall:
  - function accuracy = **0.994**
  - result accuracy = **0.399**
  - average steps = **3.608**

Recommendation is again the main challenge, and again PUMA leads:

- recommendation function accuracy = **0.984**
- recommendation result accuracy = **0.052**

### 6.5 Figure 7: efficiency

**Figure 7** measures average single-turn completion time.

- GPT-based baselines take about **6.5–6.9 seconds**.
- **PUMA** takes about **2.8 seconds**.

The claimed reason is that PUMA uses:

- a smaller model,
- more compact memory,
- and therefore lower inference overhead.

So PUMA is not only more accurate, but also more efficient.

---

## 7. In-depth analysis and appendix results

The appendix is important because it explains _why_ PUMA works and how robust it is.

### 7.1 Table 5: ablation study

**Table 5** removes key components of PUMA.

Main findings:

- removing **task-specific memory** notably hurts result accuracy, especially recommendation;
- removing **SFT** causes a dramatic collapse in result accuracy;
- removing **DPO** causes a smaller but still real drop.

Interpretation:

- memory provides relevant user evidence,
- SFT is essential to learn to produce usable parameters,
- DPO adds preference alignment on top.

### 7.2 Table 6: memory length

**Table 6** compares memory token lengths 256, 512, and 768.

The main pattern is:

- function accuracy is fairly stable,
- result accuracy, especially for recommendation, improves with longer memory.

The best overall result is at **768 tokens**.

### 7.3 Figure 8: action transitions in multi-turn interaction

**Figure 8** visualizes which functions PUMA calls across steps.

- For **search**, the agent often alternates between `search_product_by_query` and `respond`, showing iterative refinement using user feedback.
- For **recommendation**, the action flow is more entangled, indicating a harder task with more dynamic adaptation.

The figure supports the claim that recommendation is structurally more difficult than search.

### 7.4 Figure 9: performance over multiple attempts

**Figure 9** shows how result accuracy changes with more attempts in multi-turn settings.

Findings:

- most tasks are solved early;
- review tasks usually finish within the first two attempts;
- later attempts tend to have lower result accuracy because harder tasks remain;
- the model is still not very effective at leveraging later-round feedback.

This suggests multi-turn interaction helps, but is not fully optimized yet.

### 7.5 Table 7: outcome accuracy

The paper introduces **Outcome Accuracy (O. Acc.)**, motivated by a practical point: users care more about getting relevant results than about whether the agent used the “correct” internal function.

Table 7 shows that even when function accuracy and result accuracy vary strongly, **PUMA still has the highest outcome accuracy** across search and recommendation.

This is a useful contribution because it reframes evaluation around user-facing success.

### 7.6 Table 8: BM25 vs dense retrieval

**Table 8** compares BM25 and dense retrieval for the search function.

Finding:

- dense retrieval slightly underperforms BM25 across methods,
- but the overall ranking of methods remains the same,
- and PUMA stays best.

This supports the paper’s claim that the main contribution is the **agent’s personalized use of tools**, not a specialized retrieval backend.

### 7.7 Table 9: zero-shot and few-shot users

The paper evaluates users with fewer than 10 historical records.

Findings:

- search remains stable or improves slightly,
- recommendation also improves somewhat because less irrelevant memory must be filtered,
- review quality drops because sparse history makes style personalization harder,
- PUMA still remains best overall.

This is important because it shows the framework still works under sparse-memory conditions.

---

## 8. Prompting details and supplementary diagrams

The paper also includes a set of prompt templates and support figures.

### 8.1 Figure 10 and Figure 11: profile generation

- **Figure 10** gives the user-profile-generation prompt.
- **Figure 11** specifies the allowed choices for profile fields such as gender, age, occupation, price sensitivity, diversity, interaction style, and focus aspect.

These figures matter because they define exactly how the synthetic user profiles are standardized.

### 8.2 Figures 12–14: instruction generation prompts

- **Figure 12:** search instruction generation prompt
- **Figure 13:** recommendation instruction generation prompt
- **Figure 14:** review instruction generation prompt

These prompts are designed to generate requests that are natural, grounded, somewhat vague, and consistent with user profile and style.

### 8.3 Figure 15: user simulator prompt

**Figure 15** defines the prompt for the multi-turn user simulator.

It constrains the simulator to:

- respond one line at a time,
- avoid hallucination,
- avoid over-specifying products,
- reveal clues only through natural feedback.

This is crucial because the quality of multi-turn evaluation depends directly on simulator quality.

### 8.4 Figures 16 and 17: task-execution prompts

- **Figure 16** gives the single-turn prompt template.
- **Figure 17** gives the multi-turn prompt template.

These figures show that all methods are prompted under a common format, which helps make the baseline comparison fair.

---

## 9. Discussion, ethics, privacy, and scope

The paper explicitly acknowledges limitations.

### 9.1 Ethical and privacy concerns

Using personalized data introduces two major concerns:

1. **Fairness:** personalization may amplify bias, popularity bias, or unequal treatment across user groups.
2. **Privacy:** user histories may contain sensitive behavioral and purchase information.

The paper suggests future work on:

- fairness-aware personalization,
- diversity-aware recommendations,
- privacy-preserving personalization.

### 9.2 Scope of the benchmark

The benchmark focuses on **shopping**, which is a strong but limited domain.

The authors argue this domain is a good starting point because it has:

- structured user histories,
- diverse actions,
- clear personalization value.

They also claim the framework should generalize to other domains such as:

- news recommendation,
- social media content curation,
- and broader user-facing web applications.

---

## 10. Conclusion and final takeaway

The paper’s overall contribution is to move web-agent research from:

- **generic instruction-following agents**

to:

- **personalized agents that act for a specific user**.

Its main technical insight is that personalization is both:

1. a **memory retrieval** problem, and
2. an **alignment** problem.

The benchmark contribution is **PersonalWAB**.
The method contribution is **PUMA**.

The strongest empirical story is:

- personalization especially helps **recommendation**,
- task-specific memory matters,
- SFT is essential,
- DPO further improves alignment,
- and the resulting system is both **more accurate** and **faster**.

## Final condensed summary

If reduced to one paragraph: this paper defines the task of **personalized web agents**, builds the first benchmark (**PersonalWAB**) for evaluating them on personalized search, recommendation, and review generation, and proposes **PUMA**, a two-stage framework that first identifies the correct web function and then generates personalized parameters using task-specific memory retrieval plus SFT and DPO. Across both single-turn and multi-turn evaluation, PUMA substantially improves function selection and final results, especially on recommendation tasks, while also being more efficient than strong LLM baselines. The paper’s broader message is that future web agents should not merely follow instructions; they should infer who the user is and act accordingly.

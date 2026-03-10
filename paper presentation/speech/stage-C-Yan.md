# Stage C Speech Script — Yan

## Opening handoff from Eric to Yan

“Thanks, Eric. So far, we have seen the problem setting and the benchmark environment. In Stage C, I’ll explain the method, the experimental results, and the main conclusion of the paper.”

---

## Slide 20 — PUMA: the main method idea

“The core method in this paper is called **PUMA**, which stands for **Personalized User Memory-enhanced Alignment**.

The high-level idea is very simple. Personalization is turned into a two-stage process. First, the agent identifies the right web function. Second, it generates the most user-aligned parameters for that function.

This is a useful decomposition because agents can fail in two different ways. They can choose the wrong action, or they can choose the correct action but fill in poor parameters.

The figure on this slide gives the intuition. PUMA tries to retrieve the right part of the user’s long-term memory, and then use that memory to guide parameter generation. So the method combines memory retrieval with alignment, instead of relying only on generic instruction following.”

---

## Slide 21 — Task-specific memory retrieval

“This slide focuses on the memory part of PUMA.

For each user, the system builds a long-term memory bank containing historical purchases, product metadata, ratings, and reviews. So the memory is rich enough to capture both what the user bought and how the user usually reacts to products.

Then, when a new instruction comes in, PUMA does not use the full history directly. Instead, it retrieves the **Top-K** most relevant behaviors and extracts the features that match the chosen function.

That is what Equation 2 says. Starting from the full memory bank $M$, PUMA retrieves relevant memories using similarity with the current instruction $i$, and then extracts the information needed for the identified function $f$.

So the important point is that memory retrieval is **task-specific**. The method is not just storing user history. It is selecting the right slice of memory for the current personalized task.”

---

## Slide 22 — Function parameter optimization: SFT + DPO

“After retrieving relevant memory, the next step is to generate good function parameters.

This slide shows **Stage 1**, which is supervised fine-tuning, or **SFT**.

Because there is no perfect labeled dataset for all personalized parameters, the authors build pseudo-labels heuristically. For search, the label is a textual query. For recommendation, the label is recent product IDs from the relevant category. For review, the label is the actual review text.

On the right, the paper then constructs a preference-pair dataset for DPO. For each input, it keeps a better parameter candidate and a worse parameter candidate, based on downstream result accuracy.

The key message here is that SFT gives the model a reasonable starting point. It teaches the model how to generate usable personalized parameters before stronger preference-based optimization is applied.”

---

## Slide 23 — Function parameter optimization: Stage 2 with DPO

“This slide shows **Stage 2**, which is **Direct Preference Optimization**, or **DPO**.

You do not need to read every symbol in the equation. The main idea is simple. DPO pushes the model to prefer parameter choices that lead to better personalized outcomes, and avoid parameter choices that lead to worse outcomes.

So in practice, this stage rewards better parameters, suppresses worse ones, and aligns the output with user preference.

Why is this important? Because even if the agent chooses the correct function, the final quality still depends heavily on the parameter values. DPO gives the model a direct way to learn from this difference without requiring a separate reward model during deployment.

So if Slide 22 gives the model a good initialization, Slide 23 is what sharpens it into a more personalized decision maker.”

---

## Slide 24 — Single-turn results: PUMA leads on overall personalization

“Now let’s look at the experimental results.

This table reports the **single-turn** setting. That means the agent only has one chance to act.

The main result is that **PUMA performs best overall**. In particular, the strongest improvement appears in **recommendation function accuracy**. That matters because recommendation is exactly where hidden user preference is most important.

For example, if we compare with the No Memory baseline, recommendation function accuracy is extremely low there, but rises sharply with PUMA. This tells us that generic tool use is not enough. The agent needs memory and alignment to know what recommendation action really fits the user.

Another nice result is that **PUMA with LLaMA-7B** achieves the best overall scores, showing that the method is effective even with a smaller specialized model, not only with a large closed model.”

---

## Slide 25 — Multi-turn results: stronger accuracy with controlled interaction

“This table moves to the **multi-turn** setting, where the agent can interact with the user simulator.

Here, most methods improve somewhat because they can ask for more information. But again, **PUMA still achieves the best overall function accuracy and result accuracy**.

This suggests two things. First, memory retrieval is even more useful when interaction is allowed. Second, simply adding more reasoning steps is not enough. Some baselines take more steps, but they still do not reach the same overall quality.

So the main lesson from this table is that multi-turn interaction helps, but what really matters is whether the agent can use user memory in a focused and personalized way. That is exactly what PUMA is designed to do.”

---

## Slide 26 — Efficiency: better personalization does not have to be slower

“This slide looks at efficiency.

In the single-turn track, the GPT-based baselines take roughly **6.5 to 6.9 seconds** on average, while **PUMA takes about 2.8 seconds**.

This is an important practical result. The method is not only more accurate, it is also faster. So personalization does not necessarily mean higher latency.

That matters for deployment, because real user-facing systems need both quality and speed. A method that is accurate but too slow is hard to use in practice. This figure suggests that PUMA is strong on both dimensions.”

---

## Slide 27 — Conclusion: what should we remember?

“This slide gives the big-picture takeaway.

The paper formalizes a new setting: **personalized web agents**. It introduces **PersonalWAB** as the first benchmark dedicated to this setting. And it shows that **PUMA** improves performance by combining task-specific memory retrieval, function identification, and preference-aligned parameter optimization.

The gains are strongest when the instruction is ambiguous or underspecified. That makes sense, because in those cases the agent must rely more on user memory and less on the literal wording of the prompt.

So if I had to summarize the main message in one sentence, it would be this: if web agents are supposed to act for users, then they need to remember who the users are.”

---

## Slide 28 — Limitations and future work

“Finally, the paper is also clear about its limitations.

The current setting is still restricted to **e-commerce**. The environment is simplified into function calls rather than messy real websites. And the quality of personalization still depends on how informative the stored behavior history is. The paper also notes open questions about privacy and stale memory.

For future work, the authors suggest broader domains, more realistic web environments, dynamic preference updating over time, real human interaction, and fairness, safety, and privacy-aware personalization.

So overall, this paper is a strong first step. It gives the field a clear problem definition, a benchmark, and a method. But it also leaves plenty of room for more realistic and broader personalized agent systems in the future.”

---

## Closing

“That concludes our presentation. Thank you for listening, and we’d be happy to take your questions.”

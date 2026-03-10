**PUMA uses fine-tuning because the paper is not only testing whether memory helps, but whether a model can learn a stable mapping from `instruction + user memory + function type` to good personalized function parameters.**  
That is different from “does a strong API model know Amazon-like text?”

A few clarifications:

## 1. How did they fine-tune the model?

The paper fine-tunes **`LLaMA-2-7B` with LoRA-style training**.

---

## 2. Why pretraining alone is not enough

A pretrained LLM may have seen:

- Amazon-style product text,
- review language,
- generic shopping knowledge.

But that does **not** mean it has learned the specific benchmark mapping:

$$
(\text{instruction}, \text{user profile}, \text{user history}, \text{function}) \rightarrow \text{personalized parameter}
$$

That mapping is the real task.

Examples:

- for `search`, it must compress user preference into a **better query**
- for `recommend`, it must infer **which past behaviors matter**
- for `review`, it must imitate **this user’s tone and focus**

So the issue is not raw world knowledge.  
The issue is **task-specific personalization behavior**.

---

## 3. Why use `SFT` instead of only prompting / few-shot / RAG?

Because prompting and RAG mainly **supply information**.  
They do not guarantee the model learns to **use that information correctly and consistently**.

The paper’s logic is:

- **retrieval** gives relevant memory
- **`SFT`** teaches the model how to turn that memory into usable parameters
- **`DPO`** further pushes outputs toward higher-reward parameters

So `SFT` is doing something different from RAG:

- RAG = “here is more context”
- `SFT` = “here is the transformation behavior to imitate”

This is especially important for recommendation, where the request is vague and the model can easily default to generic search-like behavior.

---

## 4. Why not just use a strong API model?

That criticism is valid in practice.

In fact, the paper partially agrees with it:

- they report **`PUMA (gpt-4o)`**
- and also **`PUMA (LLaMA-7B)`**

So the framework is **not only about fine-tuned local models**.  
The paper shows two things:

1. retrieval + decomposition helps even with a strong proprietary model
2. additional task-specific training helps even more

So the claim is not “fine-tuning is always necessary.”  
The claim is closer to: **prompting alone leaves performance on the table for personalized function execution.**

---

## 5. Why did the authors still choose fine-tuning?

Because they want to show that personalization is not just a prompting trick.  
They treat it as a learnable alignment problem.

Their argument is:

- general LLMs are good at language
- but not reliably good at **personalized function parameter generation**
- therefore the model should be adapted to the task distribution

This is why PUMA has:

- function identification training
- parameter-generation `SFT`
- preference optimization with `DPO`

---

## 6. Is `SFT` redundant?

Not according to the paper’s design.

Why not redundant:

- the labels teach the model what a **good parameter** looks like
- the task has structured outputs, not just free-form chat
- recommendation is especially hard because the model must infer latent preference from long histories

Also, the paper’s ablation message is essentially:

- removing task-specific memory hurts
- removing `SFT` hurts a lot
- removing `DPO` also hurts, but less than removing `SFT`

So within the paper’s own evidence, `SFT` is a core component, not an unnecessary extra.

---

## 7. Is the paper less realistic because of this?

Yes, that is a fair limitation.

A real company often prefers:

- prompting
- RAG
- tool calling
- small adapters if needed
- and only rarely full fine-tuning

So a fair critique is:

**PUMA is strong as a research demonstration, but not necessarily the default production recipe.**

A more production-oriented version of the idea would be:

- keep the decomposition
- keep task-specific retrieval
- use a frontier API model
- try prompt engineering / structured output / reranking first
- add lightweight tuning only if needed

---

## 8. Why few-shot relational context may still be insufficient

Because the model must learn patterns like:

- which history items are relevant for this task
- which fields matter for this function
- how to convert them into a parameter format
- how to avoid generic but plausible wrong outputs

Few-shot examples can help, but they are:

- less stable,
- more context-expensive,
- more sensitive to prompt wording,
- and harder to optimize systematically.

`SFT` gives a more compact and repeatable way to encode this behavior.

---

## 9. Best way to phrase this in discussion

A balanced answer would be:

> The criticism is reasonable: in production, companies often prefer API-based prompting and RAG over fine-tuning. However, PUMA uses `SFT` because the paper treats personalization as a task-specific alignment problem, not just an information-retrieval problem. Pretraining may expose the model to product and review text, but it does not teach the exact mapping from user memory to personalized function parameters. So in the paper, retrieval supplies relevant evidence, while `SFT` teaches the model how to use that evidence consistently. This makes the method stronger experimentally, even if it is less deployment-friendly than a pure API-based setup.

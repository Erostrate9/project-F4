# Stage A Speech Script — Sabrina

## Opening

“Good evening everyone. Today our group is presenting the paper _Large Language Models Empowered Personalized Web Agents_. I’ll cover Stage A, which includes the motivation, the related work gap, the task formulation, and the main contributions.”

---

## Slide 4 — Why do we need personalized web agents?

“Let’s start with the motivation.

Modern web services are very powerful, but they are also difficult to use efficiently. There are too many products, too many pages, and too many options. So web agents are attractive because they can act on behalf of users.

But the problem is that most existing web agents are still optimized mainly for **instruction following**. They can follow a command, but they are much weaker at understanding **who the user is**.

This becomes a serious issue when the request is underspecified. For example, if I say, ‘Find me a backpack,’ that is not enough information. The right answer depends on my budget, my preferred brands, my past purchases, and even what kind of trips I usually take.

The same is true for recommendation and review writing. A good personalized web agent should combine three things: the current instruction, personalized user data, and the right web action. So the key shift in this paper is from **generic automation** to **user-aware automation**.”

---

## Slide 5 — From traditional agents to personalized agents

“This figure makes that shift very clear.

In figure (a), we have a traditional web agent. It takes the current instruction, interacts with the web, and returns a result. But it mostly treats all users in the same way.

In figure (b), we have a personalized web agent. Now the agent also has access to user profile information and behavior history. That means it can infer hidden preferences and make a more personalized action.

So the difference is not just adding more data. The difference is changing the role of the agent. A traditional agent asks, ‘What did the user request?’ A personalized agent asks, ‘What did the user request, and what would be best for this specific user?’

That is why the paper argues that personalization is central. If agents are supposed to act for users, then they need some memory of user preferences and behavior.”

---

## Slide 6 — Related work and the missing bridge

“Next, the paper positions itself between two existing research directions.

The first direction is **web-agent benchmarks**. These benchmarks study things like browsing, navigation, shopping, or multi-step web interaction. They are useful, but they usually do not include personalization.

The second direction is **personalized LLM research**. This work studies personalized dialogue, content generation, and recommendation. So it cares about the user, but usually not about actual web action execution.

The missing bridge is the sentence in bold: **personalized function execution**. In other words, can an agent choose the right web action and fill in the right parameters for this particular user?

The table on the right highlights this gap. Earlier benchmarks may support single-turn or multi-turn interaction, and some use web functions instead of full web pages, but they still do not model personalization. PersonalWAB is the first benchmark here that combines both interaction and personalization.”

---

## Slide 7 — Task formulation: what exactly is the agent solving?

“This slide gives the formal problem definition.

The input includes four parts: a user instruction, a user profile, the user’s behavior memory, and a set of callable web functions.

The output is also two-part. The agent has to choose the correct function, and then generate the personalized parameters for that function.

So the task is not only about selecting a valid action. It is also about inferring hidden preferences that are often not explicitly written in the prompt.

That is why the problem is harder than a standard web-agent task. The agent has to solve two things together. First, what action should I take? Second, how should I customize that action so that it fits the user?

So from this point on, the paper treats personalized web interaction as a joint problem of **action selection** and **preference inference**.”

---

## Slide 8 — Main contributions of the paper

“This slide summarizes the paper’s main contributions.

First, it defines a **new task**: personalized web agents as a formal research problem.

Second, it introduces a **new benchmark**, called **PersonalWAB**, that covers personalized search, recommendation, and review generation.

Third, it proposes a **new method**, called **PUMA**, which stands for Personalized User Memory-enhanced Alignment.

And fourth, the main empirical result is that retrieving user history improves both action selection and final result quality, especially when the instruction is ambiguous or underspecified.

So the big takeaway from Stage A is this: personalization is not just a small feature. In this paper, it is treated as both a **memory retrieval problem** and an **alignment problem**.

With that setup in place, Eric will now explain how the authors build the benchmark and why that benchmark is such an important part of the paper.”

---

## Closing handoff to Eric

“So far, we have seen why personalized web agents matter, what gap the paper addresses, and how the task is defined. Next, Eric will move to Stage B and walk through the benchmark construction, the evaluation protocol, and the benchmark analysis.”

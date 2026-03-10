# Stage B Speech Script — Eric

## Opening handoff from Sabrina to Eric

“So far, we have seen why personalized web agents matter, and what problem the paper is trying to solve. Now I’ll move to Stage B, which is the benchmark construction, the evaluation setup, and the benchmark analysis.”

---

## Slide 10 — RL-style view: priors, environment, and algorithm

“Before I get into the benchmark details, I want to give one high-level way to read this paper.

In modern agent research, especially from an RL-style view, performance depends on three things: **priors, environment, and algorithm**.

For **priors**, this paper starts from strong pretrained language models. That gives the agent general language understanding, reasoning, and tool-use ability.

For the **environment**, the key contribution is **PersonalWAB**. It defines the tasks, the action interface, and the evaluation setting for personalized web agents.

And for the **algorithm**, the paper proposes **PUMA**, which uses function identification, memory retrieval, and parameter optimization.

So this slide matters because the benchmark is not just background material. In this paper, it is a major part of the **environment design**, and that is what makes the later algorithm meaningful to evaluate.”

---

## Slide 11 — PersonalWAB: benchmark construction pipeline

“Now let’s move to the actual benchmark pipeline.

This slide shows the overall pipeline for building **PersonalWAB**. The authors do not run expensive live user studies. Instead, they build a controlled benchmark from **Amazon Review** data. The pipeline has three main stages. First, they construct personalized user data. Second, they create personalized instructions. Third, they build a web environment with callable functions.

The key goal is to isolate the real challenge they care about, which is **personalization**. They want to test whether an agent can understand a user’s hidden preferences and then take the right action. So this pipeline is designed to remove unnecessary noise and keep the benchmark focused.”

---

## Slide 12 — How the benchmark data is built

“Now let’s look at how the data is actually built.

The benchmark starts with **1,000 users** sampled from Amazon Reviews. These users come from five product categories: Electronics, Home and Kitchen, Grocery and Gourmet Food, Clothing, Shoes and Jewelry, and Health and Household. So even though the domain is e-commerce, there is still a decent range of products and user behavior.

For each user, the authors sort the interactions by time, and then split them into **80 percent history, 10 percent training, and 10 percent testing**. This is important because the agent should only use the past to help with future tasks. That makes the setup more realistic.

They also generate a structured user profile. The profile includes basic information, shopping preferences, behavioral tendencies, tone and style, item references, and focus aspects in reviews. In simple words, the benchmark does not just store what the user bought. It also tries to model **what kind of person this user is** and **how this user usually behaves**.”

---

## Slide 13 — Three tasks and a function-based web environment

“After building the users and profiles, the paper creates three task families.

The first is **search**, where the agent needs to produce a personalized query. The second is **recommendation**, where the agent needs to choose relevant products based on the user’s history. The third is **review generation**, where the agent needs to write a review in the user’s own style.

The other important design choice is on the right side of this slide. Instead of using a full browser interface, the authors use a **function-based web environment**. The available functions are `search_product_by_query`, `get_recommendations_by_history`, `add_product_review`, `respond`, and `stop`.

Why is this useful? Because the paper is not mainly testing browser navigation. It is testing whether the agent can choose the **right personalized action** and fill in the **right personalized parameters**. So again, this goes back to the environment idea: the benchmark is carefully designed to test the exact capability the authors care about, not unrelated UI skills.”

---

## Slide 14 — Evaluation protocol

“This slide explains how the benchmark is evaluated.

There are **two tracks**. In the **single-turn** track, the agent gets only one chance to act. In the **multi-turn** track, the agent can ask questions or respond, and it receives feedback from a simulator.

There are also three main metrics. First is **function accuracy**. This checks whether the agent picked the correct function and used the correct parameter format. Second is **result accuracy**. For search and recommendation, this depends on the rank of the target item in the returned list. The closer the target item is to the top, the higher the score. If it is outside the top ten, the score becomes zero. For review generation, the paper instead uses sentence-level similarity between the generated review and the reference review. Third, in the multi-turn setting, they also report **average steps**, which measures efficiency.

I think this evaluation design is quite clean. It separates three questions: did the agent choose the right tool, did it produce a good result, and did it do so efficiently?”

---

## Slide 15 — What does PersonalWAB contain?

“This table gives us a better sense of the scale of the benchmark.

On the user side, there are **939 training users** and **1,000 test users**. The average profile length is **247 tokens**, which means the profiles are not just tiny labels. More importantly, the behavior history is long. The average behavior length is **32** in training and **38** in testing, and the average behavior token count is around **7,597** for training and **9,270** for testing.

On the instruction side, there are **6,896 training instructions** and **2,174 test instructions**, with about **45 to 46 tokens** each on average. On the product side, there are **8,236 products**, and each product description is fairly detailed.

So the main takeaway is that PersonalWAB is not a toy dataset. Each test case combines a user profile, a long behavior history, a task instruction, a product space, and available tools. That is why this benchmark is useful for serious agent evaluation.”

---

## Slide 16 — Benchmark analysis: diversity of users and instructions

“After building a benchmark, the next question is: is it diverse enough?

This slide answers that question in two ways. On the left, Figure 3 shows the distribution of users by **gender, age, and occupation**. The point is not that the dataset is perfectly balanced, but that it covers a fairly wide range of user backgrounds and jobs.

On the right, Figure 4 looks at behavior and instruction diversity. The behavioral attributes include **price sensitivity, diversity preference, and interaction complexity**. Most users fall into the medium range, which is expected, but there are still enough low and high cases to make the benchmark challenging.

The figure also shows differences between instruction types. In particular, **recommendation instructions are usually shorter**, because they are more open-ended. **Review instructions are a bit more expressive and detailed**, because users tend to say more when they talk about how they feel about a product.

So this slide supports the claim that the benchmark has both user diversity and task diversity.”

---

## Slide 17 — Benchmark analysis: are the generated profiles believable?

“Diversity alone is not enough. The generated profiles also need to be believable. So this slide asks a very important question: do these LLM-generated profiles actually match real user behavior?

The paper checks this with **profile consistency evaluation**. In simple terms, it tests whether a generated profile matches the user’s past behaviors and also matches the products that the user is likely to care about.
In the paper, generated profiles are applied to **two evaluation tasks**.

- Profile-behavior consistency (given a generated profile, identify **which candidate user behavior sequence** belongs to that profile.)
- Profile-product consistency: given a generated profile, rank **candidate products/items**.

The result is shown in Figure 5. Compared with a previous method called **Apollonion** **/əˌpɑːloʊˈniːən/**, PersonalWAB improves by about **25.8 percent on Recall@5**, **18.3 percent on NDCG@5**, and **13.3 percent on Acc@1**.
the `@k` notation means: “measure performance within the first k returned items.”

Why does this matter? Because if the generated profiles were weak, then the whole benchmark would be questionable. But these results suggest that the profiles are meaningfully aligned with real user history. So the benchmark is not only diverse, but also reasonably consistent and believable.”

Q&A:
Recall@5
Meaning: how often the relevant item(s) appear in the top 5 results.
Intuition: If the system ranks products using the generated user profile, Recall@5 checks whether products the user truly likes are successfully retrieved among the first 5.
Indicates in this paper: Higher Recall@5 means the generated profile captures user preference well enough to bring relevant products into the top few results.

2. NDCG@5
   Full name: Normalized Discounted Cumulative Gain at 5
   Meaning: measures the quality of the ranking in the top 5, while giving more credit when relevant items appear higher in the list.
   Intuition: A relevant item at rank 1 is better than the same item at rank 5. NDCG@5 reflects that.

Indicates in this paper: Higher NDCG@5 means the generated profile not only retrieves relevant products, but ranks them in a better order, with the most relevant ones nearer the top.

Acc@1
Meaning: whether the top 1 result is correct/relevant.
Intuition: This is a very strict metric: only the first ranked item matters.
Indicates in this paper: Higher Acc@1 means the profile is strong enough that the system’s single best prediction is often the right one.

---

## Slide 18 — Benchmark strengths and limitations

“Finally, this slide summarizes why PersonalWAB matters, and also where it is limited.

On the strength side, this is the **first benchmark focused on personalized web-agent behavior**. It supports both **single-turn and multi-turn** interaction. And it separates **function choice** from **parameter quality**, which is very useful because an agent can fail in both places.

On the limitation side, the benchmark is still restricted to **e-commerce**. It uses **function abstraction** instead of messy real websites. And the users are still **simulated**, because the profiles and multi-turn user feedback come from models rather than real deployment.

So my overall takeaway is this: PersonalWAB is a very clean and useful **testbed** for personalization, even though it is not the full real web. And that is exactly why it is valuable as an environment for developing and evaluating personalized agents.

With that benchmark in place, the next question becomes: how do we design a method that can actually use this personalized environment well? That is what Yan will cover next with **PUMA**.”

---

## Closing handoff to Yan

“So to summarize my part, Stage B showed how the paper builds the environment: the users, the profiles, the tasks, the functions, and the evaluation protocol. Now Yan will move to Stage C and explain the method and the experimental results.”

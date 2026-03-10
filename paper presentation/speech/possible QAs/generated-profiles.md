## 1. What task are the generated profiles applied on?

In the paper, generated profiles are applied to **two auxiliary evaluation tasks**:

### A. Profile-behavior consistency

Goal: given a generated profile, identify **which candidate user behavior sequence** belongs to that profile.

- input: one profile
- candidates: the true user + several negative users
- output: rank the candidate users
- metric: `Acc@1`

This asks: “Does this generated profile uniquely match the correct user’s past behavior?”

### B. Profile-product consistency

Goal: given a generated profile, rank **candidate products/items**.

- input: one profile
- candidates: positive items the user actually interacted with + randomly sampled negative items
- output: rank the candidate items
- metrics: `Recall@5`, `NDCG@5`

This asks: “Does this generated profile prefer the kinds of products the real user actually liked?”

So the generated profiles are used here for **matching and ranking**, not directly for the main tool-calling benchmark.

They are also used elsewhere in the paper for:

- instruction generation,
- user simulation,
- and as personalization context for agents.

---

## 2. How were `Recall`, `NDCG`, and `Acc` calculated?

## `Acc@1`

This is **top-1 accuracy**.

For one profile:

- if the correct user is ranked 1st among candidate users, score = 1
- otherwise, score = 0

Then average over all evaluation examples.

### Formula

$$
\mathrm{Acc@1} = \frac{1}{N}\sum_{i=1}^{N}\mathbf{1}[\text{correct candidate ranked at position 1}]
$$

### What it indicates here

Whether the generated profile is **distinctive enough** to identify the correct user from behavior sequences.

---

## `Recall@5`

This is the fraction of relevant items found in the **top 5 ranked products**.

If a user has $R$ positive items in the candidate set, and $r_5$ of them appear in the top 5, then:

$$
\mathrm{Recall@5} = \frac{r_5}{R}
$$

### What `@5` means

Only the first 5 ranked products are considered.

### What it indicates here

Whether the profile can bring the user’s true preferred items into the top few results.

---

## `NDCG@5`

This is **Normalized Discounted Cumulative Gain at 5**.

It measures not just whether relevant items appear in the top 5, but **how high** they appear.

For binary relevance:

- relevant item: `rel_i = 1`
- irrelevant item: `rel_i = 0`

### DCG@5

$$
\mathrm{DCG@5}=\sum_{i=1}^{5}\frac{2^{rel_i}-1}{\log_2(i+1)}
$$

### NDCG@5

$$
\mathrm{NDCG@5}=\frac{\mathrm{DCG@5}}{\mathrm{IDCG@5}}
$$

where `IDCG@5` is the best possible DCG if all relevant items were ranked as high as possible.

### What it indicates here

Whether the profile not only retrieves relevant items, but ranks them **near the top**.

---

## 3. What does `@5` and `@1` mean?

- `@1` = look only at the top-ranked result
- `@5` = look only at the first 5 ranked results

So:

- `Acc@1` asks: “Is the very top prediction correct?”
- `Recall@5` asks: “How many relevant items are inside the top 5?”
- `NDCG@5` asks: “How well are relevant items ordered inside the top 5?”

---

## 4. What retrieval/ranking method is used according to the profile?

This part is important:

### For the profile-evaluation experiment

The paper does **not** describe a classical retrieval backend like BM25 or SASRec for this profile check.

Instead, it says:

- they follow the same setting as Apollonion,
- they use `gpt-4o-mini-2024-07-18`,
- and the task is to **rank candidate users/items based on the profile**.

So for this experiment, it is best understood as an **LLM-based ranking task over sampled candidates**, not open-corpus retrieval.

### Candidate construction

The appendix says they use:

- **profile-behavior/user prediction:** 1 positive + 4 negative users
- **profile-product/recommendation:** 3 positive + 7 negative items

So the model is not searching the whole product catalog here. It is ranking a **small candidate set**.

### How the ranked result is evaluated

- user-ranking task: `Acc@1`
- product-ranking task: `Recall@5` and `NDCG@5`

---

## 5. Real end-to-end example: `Acc@1`

Use a real profile from the dataset:

### Real generated profile

User `AGMQSZEQFFKH33FJQZLN7MF5QX2Q` has profile cues like:

- shopping interest: health and wellness, outdoor gear, home essentials, tech gadgets
- brands: Spoonk, adidas, TP-Link, THE NORTH FACE, Apple, etc.
- focus: quality, functionality, brand reputation

### Candidate users

Suppose the evaluation gives this profile and 5 candidate behavior sequences:

1. **User A**
   - Spoonk acupressure mat
   - adidas hiking boot
   - Mastic gum

2. **User B**
   - men’s titanium ring
   - Dickies carpenter jean
   - Wrangler thermal jean

3. **User C**
   - drawing glove
   - remote holder
   - Glad trash bags

4. **User D**
   - other unrelated sequence

5. **User E**
   - other unrelated sequence

The ranking model sees the profile and decides the order:

- Rank 1: User A
- Rank 2: User C
- Rank 3: User B
- Rank 4: User D
- Rank 5: User E

Since User A is the true user, for this example:

$$
\mathrm{Acc@1}=1
$$

If the model ranked User C first and User A second, then:

$$
\mathrm{Acc@1}=0
$$

Over many examples, they average these 0/1 values.

---

## 6. Real end-to-end example: `Recall@5` and `NDCG@5`

Use the same real profile.

### Positive items for this user

Three real products from this user’s history:

- `P1`: Spoonk Acupressure Eco Mat
- `P2`: adidas Outdoor Men’s AX2 Mid Gore-Tex Hiking Boot
- `P3`: Mastic Gum – 0.6oz

These are the **positive** items.

### Negative items

Seven sampled negatives, for example:

- `N1`: Ergonomic Food Mill Stainless Steel
- `N2`: Toddler Baby Boy Dinosaur Outfit
- `N3`: Amish Country Popcorn Baby White
- `N4`: Filippo Berio Olive Oil
- `N5`: Stainless Steel Potato Ricer
- `N6`: Pet Allergy Relief Tablets
- `N7`: Air Wick Essential Mist Starter Kit

Now the model ranks all 10 items from most profile-matching to least.

### Suppose the model outputs this ranking

1. `P1` Spoonk Acupressure Mat ✅
2. `N2` Toddler Baby Clothes ❌
3. `P2` adidas Hiking Boot ✅
4. `N4` Olive Oil ❌
5. `N7` Air Wick Mist Kit ❌
6. `P3` Mastic Gum ✅
7. `N1` Food Mill ❌
8. `N5` Potato Ricer ❌
9. `N6` Pet Allergy Relief ❌
10. `N3` Amish Country Popcorn ❌

There are 3 relevant items total: `P1`, `P2`, `P3`.

### Step A: `Recall@5`

Top 5 contains:

- `P1`
- `P2`

So 2 relevant items are inside top 5.

$$
\mathrm{Recall@5}=\frac{2}{3}=0.667
$$

---

### Step B: `NDCG@5`

Binary relevance in top 5:

- rank 1: 1
- rank 2: 0
- rank 3: 1
- rank 4: 0
- rank 5: 0

So:

$$
\mathrm{DCG@5}
= \frac{1}{\log_2(2)}
+ \frac{0}{\log_2(3)}
+ \frac{1}{\log_2(4)}
+ \frac{0}{\log_2(5)}
+ \frac{0}{\log_2(6)}
$$

$$
= 1 + 0 + 0.5 + 0 + 0 = 1.5
$$

The ideal ranking would place all 3 positives in the top 3:

1. `P1`
2. `P2`
3. `P3`

Then:

$$
\mathrm{IDCG@5}
= \frac{1}{\log_2(2)}+\frac{1}{\log_2(3)}+\frac{1}{\log_2(4)}
$$

$$
= 1 + 0.6309 + 0.5 = 2.1309
$$

Therefore:

$$
\mathrm{NDCG@5}=\frac{1.5}{2.1309}\approx 0.704
$$

So for this one example:

- `Recall@5 = 0.667`
- `NDCG@5 ≈ 0.704`

---

## 7. How to interpret the three metrics together

- `Acc@1`  
  checks whether the profile can identify the **correct user behavior sequence**.

- `Recall@5`  
  checks whether the profile can recover the user’s **true preferred items** in the top 5.

- `NDCG@5`  
  checks whether those true preferred items are ranked **high enough**, not just included somewhere.

So in this paper, higher values mean the generated profile is:

- more **distinctive**,
- more **behaviorally faithful**,
- and more **useful for preference ranking**.

---

## 8. One clean summary

The paper uses generated profiles in a **profile consistency evaluation** with two ranking tasks:

1. **profile → correct user behavior sequence**  
   measured by `Acc@1`

2. **profile → preferred products among sampled candidates**  
   measured by `Recall@5` and `NDCG@5`

For this evaluation, the paper uses an **LLM-based candidate ranking setup** with `gpt-4o-mini-2024-07-18`, not the benchmark’s BM25 or SASRec backends.

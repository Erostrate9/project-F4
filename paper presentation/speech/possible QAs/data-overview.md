# What does data in PersonalWAB look like?

The benchmark data is packaged under `PersonalWAB/envs/pwab/data/` and is loaded into four main objects:

- `user_profiles.json`
- `user_instructions.json`
- `user_history_part_*.json`
- `all_products_part_*.json`

At runtime, these become:

- `data["user_profile"]`
- `data["tasks"]`
- `data["user_history"]`
- `data["all_products"]`

## LLM-generated profile

Each user profile is stored by `user_id`. The top-level structure looks like:

```json
{
  "AGMQSZEQFFKH33FJQZLN7MF5QX2Q": {
    "user_profile": {
      "Gender": "Male",
      "Age": "35-44",
      "Occupation": "Self-employed",
      "Price Sensitivity": "Medium",
      "Shopping Interest": "Health and wellness, outdoor gear, home essentials, tech gadgets",
      "Brand Preference": "Spoonk, adidas, Krinos, LifePro, Halsa, TP-Link, E EGOOZ, Hallmark, THE NORTH FACE, Thule, Spigen, VRS DESIGN, Oarkive, RYB HOME, Hanes, Amazon Basics, Apple, Garden of Life, Forge, OXO, FGO, B BANGKOK PANTS, Sanuk, KOS, Benicci",
      "Diversity Preference": "Medium",
      "Interaction Complexity": "Medium",
      "Tone and Style": "Enthusiastic, supportive, practical",
      "Item Reference": "Specific products and brands mentioned in reviews",
      "Focus Aspect": "Quality, functionality, brand reputation"
    }
  }
}
```

So the profile is not raw structured shopping logs. It is an **LLM-generated natural-language summary** of a user's likely attributes and shopping habits.

Main profile dimensions used by the benchmark:

- demographic fields: gender, age, occupation
- shopping preference fields: price sensitivity, shopping interest, brand preference
- behavior/style fields: diversity preference, interaction complexity, tone and style
- review emphasis fields: item reference, focus aspect

These profiles are later converted into prompt text for the agent or the user simulator.

## Historical behavior memory

User history is stored separately from the profile. For each `user_id`, the benchmark keeps a list of past interactions. A representative record looks like:

```json
{
  "product_info": {
    "main_category": "Health_and_Household",
    "title": "Spoonk Acupressure Eco Mat, Pagoda Blue - with Massage Ball, Travel Mat & Sling Bag - Back & Neck Massager - Travel Pillow - Stress & Muscle Relief - Sleep Aid - Relaxation Kit - Made with Cotton",
    "average_rating": 4.3,
    "rating_number": 2974,
    "features": ["..."],
    "description": ["..."],
    "price": "69.99",
    "store": "Spoonk",
    "categories": [
      "Health & Household",
      "Health Care",
      "Alternative Medicine",
      "Acupuncture"
    ],
    "details": {
      "Material": "100% Cotton",
      "Brand": "Spoonk"
    },
    "parent_asin": "B07BYJ3389"
  },
  "review": {
    "rating": 5.0,
    "title": "Five Stars",
    "text": "Love it!!!!",
    "parent_asin": "B07BYJ3389",
    "timestamp": 1452779837000
  },
  "split": "history"
}
```

This is the real long-term memory source that PUMA and other memory-based methods retrieve from.

## TASK

Tasks are stored in `user_instructions.json` with `train` and `test` splits.

Each task item has this shape:

```json
{
  "user_id": "...",
  "task": "natural-language request",
  "target": {
    "product_info": { "...": "..." },
    "review": { "...": "..." },
    "split": "train or test"
  },
  "timestamp": 1661811211414,
  "type": "search | recommend | review"
}
```

Important note: the `target` stores the benchmark gold reference used for evaluation. The agent does **not** simply read off the answer; it must choose the right function and generate good parameters.

### Search

Search tasks ask the agent to create a personalized search query for a target product. A real example looks like:

```json
{
  "user_id": "AGG72WL34NYISSHBB2DWT64GQYVQ",
  "task": "Hey there! I'm super excited to find some high-quality replacement parts for my robotic vacuum cleaner. Looking for a reputable brand with great filters and brushes that'll keep my floors sparkling clean. Any awesome recommendations around $20?",
  "type": "search",
  "target": {
    "product_info": {
      "title": "Electropan Replacement Vacuum Filter Brush Kit for Robotic Vacuum ILIFE V3, V3s, V3s pro, V5, V5s, V5s pro ILIFE Robot Vacuum Replacement Parts Ilife Vacuum Filters V3s",
      "main_category": "Home_and_Kitchen",
      "price": "18.0",
      "store": "Electropan",
      "parent_asin": "..."
    },
    "review": {
      "rating": 5.0,
      "title": "...",
      "text": "..."
    },
    "split": "train"
  }
}
```

What this means in practice:

- the user message is usually **vague but preference-bearing**
- the correct action is `search_product_by_query`
- success depends on whether the target product is ranked high in the returned top results

### Recommendation

Recommendation tasks are usually more open-ended. The agent must infer likely items from user history rather than turn the request into a direct keyword search. A real example is:

```json
{
  "user_id": "AELEDUR7XBMUCCKG6SWXRPBIUKAQ",
  "task": "Hey there! I'm looking for some handy cleaning tools for my home. Any recommendations for quality products with great ratings? Thanks!",
  "type": "recommend",
  "target": {
    "product_info": {
      "title": "Hiware All-Purpose Shower Squeegee for Shower Doors, Bathroom, Window and Car Glass - Bronze, Stainless Steel, 10 Inches",
      "main_category": "Health_and_Household",
      "price": "16.99",
      "store": "HIWARE",
      "parent_asin": "..."
    },
    "review": {
      "rating": 5.0,
      "title": "...",
      "text": "..."
    },
    "split": "test"
  }
}
```

What makes recommendation harder:

- the request is often **underspecified**
- the correct action is `get_recommendations_by_history`
- the model must use profile and behavior history to infer suitable products

### Review

Review tasks ask the agent to generate a user-aligned review for a purchased product. A real example looks like:

```json
{
  "user_id": "AEHXS44XY3KPFGH4SCCSXRI2DWNQ",
  "task": "Can you help me write a review for this remote cover? It fits okay length-wise, but the width is disappointing. I want to mention the good and bad, including how it might collect debris. The blue color is a plus though!",
  "type": "review",
  "target": {
    "product_info": {
      "title": "Case Compatible with Samsung Smart TV Remote Controller BN59 Series, Light Weight Silicone Cover Protector Shockproof Anti-Slip Remote Skin Sleeve - Black",
      "main_category": "Electronics",
      "price": "5.99",
      "store": "Lambcare",
      "parent_asin": "B08C23246L"
    },
    "review": {
      "rating": 2.0,
      "title": "Does not fit...",
      "text": "..."
    },
    "split": "test"
  }
}
```

For review tasks:

- the correct action is `add_product_review`
- the agent is given product information at reset time
- evaluation compares the generated review to the gold review using sentence-level similarity

## Product catalog

The full product database is stored in `all_products_part_*.json`. Each product entry is keyed by `parent_asin` and looks like:

```json
{
  "B07BYJ3389": {
    "main_category": "Health_and_Household",
    "title": "Spoonk Acupressure Eco Mat, Pagoda Blue - with Massage Ball, Travel Mat & Sling Bag - Back & Neck Massager - Travel Pillow - Stress & Muscle Relief - Sleep Aid - Relaxation Kit - Made with Cotton",
    "average_rating": 4.3,
    "rating_number": 2974,
    "features": ["..."],
    "description": ["..."],
    "price": "69.99",
    "store": "Spoonk",
    "categories": [
      "Health & Household",
      "Health Care",
      "Alternative Medicine",
      "Acupuncture"
    ],
    "details": {
      "Material": "100% Cotton",
      "Brand": "Spoonk",
      "Color": "Pagoda Blue"
    },
    "parent_asin": "B07BYJ3389"
  }
}
```

This is what the search and recommendation functions ultimately retrieve from.

# Single turn vs Multi-turn. What's the diff?

The underlying task records are the same, but the **interaction protocol** is different.

## Single-turn

- no user interaction is allowed
- the agent gets one request and must act immediately
- after one action call to `search_product_by_query`, `get_recommendations_by_history`, or `add_product_review`, the task ends
- this is controlled by `max_steps == -1`

In effect, single-turn tests whether the agent can solve the task from the initial request plus any provided memory.

## Multi-turn

- the agent may ask follow-up questions using `respond`
- a user simulator replies based on the stored profile and the target product
- the total number of steps is capped
- the agent can end with `stop`
- this is controlled by `max_steps > 0`

In effect, multi-turn tests whether the agent can improve personalization by interacting efficiently with the user.

## Practical difference by task

- **Search:** multi-turn can help refine vague preferences before issuing the final search query.
- **Recommendation:** multi-turn is especially useful because recommendation requests are often the most underspecified.
- **Review:** the agent can clarify tone or emphasis, but many review tasks are already fairly constrained by the product and user style.

## One-sentence summary

Single-turn asks, "Can the agent personalize correctly in one shot?" Multi-turn asks, "Can the agent personalize better by asking a few good questions first?"

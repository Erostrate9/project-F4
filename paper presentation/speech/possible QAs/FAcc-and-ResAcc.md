On page 24, these mean:

- **`F. Acc.`** = **Function Accuracy**
- **`Res Acc`** = **Result Accuracy**

The table is defined in main.tex, and the metric meanings are described in summary.md.

## `F. Acc.` = Function Accuracy

This checks whether the agent chose the **correct web function** and used a **valid parameter format**.

In this paper, that means choosing the right action among things like:

- `search_product_by_query`
- `get_recommendations_by_history`
- `add_product_review`

So:

- if the task is search and the agent calls `search_product_by_query`, that helps `F. Acc.`
- if it calls the wrong function, `F. Acc.` is bad even if the output sounds plausible

## `Res Acc` = Result Accuracy

This checks whether the **returned result itself is good**.

It depends on the task:

- **Search / Recommendation:** based on the rank of the target item in the returned list  
  If the target is higher in the top-10, `Res Acc` is higher.
- **Review:** based on similarity between the generated review and the gold review

So `F. Acc.` asks:

> Did the agent choose the right tool?

while `Res Acc` asks:

> Did the tool call produce a good personalized result?

## Why both are needed

An agent can:

- choose the right function but poor parameters  
  → high `F. Acc.`, low `Res Acc.`
- choose the wrong function  
  → low `F. Acc.` and usually poor `Res Acc.` too

That is why the paper reports both.

`DPO` works in the **training / post-`SFT` fine-tuning stage**, not in runtime.

## Short answer

- `SFT` first trains a parameter-generation model.
- Then `DPO` further **updates that model’s weights** using preferred vs rejected parameter pairs.
- At runtime, the trained model is just **used for inference**. It does **not** compute `DPO` loss online and does not optimize itself during interaction.

## Why this is clear in the paper

The paper says the model first does `SFT`, then samples parameter candidates, scores them by result accuracy, builds preference pairs, and **then applies `DPO` to optimize the fine-tuned model**. See summary.md.

The converted paper text states the same thing: they “apply `DPO` to optimize the fine-tuned model `π_ref`,” where `π_ref` is the reference model from the `SFT` stage. See the paper text.

## What happens before `DPO`

There are really 3 phases:

1. **`SFT` stage**  
   Train a parameter generator on heuristic labels.

2. **Offline scoring stage**  
   Generate multiple candidate parameters, then score them by benchmark result accuracy.  
   This produces:
   - best candidate `p_i^b`
   - worst candidate `p_i^w`

3. **`DPO` stage**  
   Use those best/worst pairs to fine-tune the model with the `DPO` loss.

That workflow is summarized in summary.md.

## Does `DPO` run during runtime?

No.

At runtime:

- the agent retrieves memory,
- chooses a function,
- uses the already trained model to generate parameters.

The repo map also separates these phases clearly:

- sampled candidates are scored offline with AGENTS.md
- chosen/rejected pairs are built in AGENTS.md
- the `DPO` model is trained in AGENTS.md

## So what is page 23 showing?

It is showing a **training objective**, not a runtime decision rule.

The `DPO` loss is used to tell the model:

- increase probability of better parameter `p^b`
- decrease probability of worse parameter `p^w`

After training, that preference is baked into the model weights.

## Simple analogy

- `SFT`: teach the student example answers
- offline scoring: mark which generated answers are better or worse
- `DPO`: retrain the student to prefer the better style of answer
- runtime: student just answers; no more retraining happens live

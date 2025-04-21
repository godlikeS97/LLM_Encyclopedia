# Identify Optimal Hyper-parameters using Scaling Laws

The standard scaling laws (like those by Kaplan or Hoffmann/Chinchilla) primarily predict the final model loss based on model size, data size, and compute. They don't directly output the best learning rate (µ) or batch size (B).

Instead, researchers use scaling laws for hyper-parameters indirectly or by developing specific scaling laws for those hyper-parameters, typically through extensive empirical studies. Here's the breakdown:

1. **Finding Optimal Hyper-parameters at Smaller Scales (Empirical Foundation):**
    - Researchers run many training experiments at smaller, more manageable scales (e.g., smaller models like 100M, 500M, 1B parameters, trained on smaller data subsets like 10B, 50B, 100B tokens).
    - For each specific scale (defined by model size N, data size D, and architecture), they perform hyper-parameter sweeps. They train multiple copies of the model at that scale, varying the learning rate (µ) and batch size (B) systematically.
    - They identify the combination (`µ_opt`, `B_opt`) that leads to the best performance (lowest loss, fastest convergence) for that specific scale.
2. **Establishing Scaling Relationships for Hyper-parameters:**
    - Once they have a collection of these optimal pairs (`µ_opt`, `B_opt`) across various scales (different N's and D's), they try to find a pattern.
    - They plot how `µ_opt` changes as N (model size) increases, and how `B_opt` changes as N increases.
    - They attempt to fit a mathematical function (essentially, another scaling law, but specifically for the hyper-parameter) to these plots. For example, they might observe relationships like:
        - `µ_opt ∝ 1 / sqrt(N)` (Optimal learning rate might decrease with model size)
        - `B_opt ∝ N^α` (Optimal batch size might increase with model size, potentially related to memory capacity or gradient noise averaging)
        - Or more complex relationships involving both N and D.
3. **Extrapolation to Target Scale:**
    - Using the mathematical function derived in step 2, they extrapolate to predict what `µ_opt` and `B_opt` should be for their target large-scale model (e.g., Qwen2.5-72B trained on 18T tokens).
    

**Example Scenario:**

Imagine they run experiments and find:

- For N=1B params, `µ_opt` = 3e-4, `B_opt` = 1M tokens
- For N=7B params, `µ_opt` = 1e-4, `B_opt` = 2M tokens
- For N=14B params, `µ_opt` = 7e-5, `B_opt` = 4M tokens

They might plot these and find a relationship suggesting that `µ_opt` decreases roughly proportionally to `1/sqrt(N)` and `B_opt` increases roughly linearly or slightly super-linearly with N (within hardware limits). They'd use these observed trends to estimate the best µ and B for N=72B.

**Important Considerations:**

- **Batch Size and Hardware:** Batch size is often heavily constrained by GPU memory. So, researchers might find the largest batch size that fits in memory and then find the optimal learning rate for that batch size. There are also rules of thumb, like the "linear scaling rule" (increase learning rate proportionally to batch size, often with adjustments), which might be tested and incorporated into their scaling analysis.
- **Learning Rate Schedules:** This process usually predicts the peak or base learning rate. The actual training uses a learning rate schedule (e.g., cosine decay) where the rate changes over time. The scaling law helps set the crucial starting/peak value.
- **Dense vs. MoE:** As mentioned in the Qwen2.5 paper (Sec 3.2), these relationships can differ between model architectures (like standard dense models vs. Mixture-of-Experts). They need to develop separate scaling laws or adjustments for each architecture type.
- **Empirical, Not Perfect:** These scaling laws for hyperparameters are based on empirical observations and curve fitting. They provide strong guidance but aren't guaranteed to be perfectly optimal. They significantly narrow down the search space, saving vast amounts of compute compared to doing hyperparameter sweeps at the full scale.
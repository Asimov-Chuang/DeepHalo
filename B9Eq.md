We appreciate Reviewer 59ua’s positive feedback. To address the questions and concerns about our method, we provide point-wise responses as follows.

#### 1. Clarifying Section 4.1 — why Equation (8) needs only **log₂ l** layers to model *l*‑th‑order interactions

We agree that Section 4.1 should state this property explicitly.  
In the revision we will:

1. **Add a sentence to Section 4.1**  
   > “Because each Equation (8) layer doubles the maximum attainable interaction order, an *l*‑th‑order effect can be represented with at most ⌈log₂ l⌉ stacked layers.”

2. **Provide an intuitive explanation** (now in the appendix, to keep the main text concise):  
   - A single (8) layer captures up to second‑order effects.  
   - Stacking two layers composes these interactions, yielding up to 4‑th‑order effects, and so on.  
   - After *k* layers the highest order is $2^{k}$, giving the logarithmic bound.

We also add a cross‑reference from §4.1 to the worked example in §4.2, where the doubling effect is illustrated concretely. 

#### 2. Practical guidance on when to use Equation (5) vs Equation (8)

We appreciate this insightful question. One of the strengths of the DeepHalo architecture is its structural flexibility: the activation function σ can be instantiated as any polynomial function, and Equations (5) and (8) are not mutually exclusive. In practice, they can be freely mixed—for example, stacking layers with linear and quadratic activations can achieve effective interaction orders such as $((1+1+1)\times2+1)\times2$, offering nuanced control over expressivity.

In general:

1. **Higher-degree activations** (like Eq. (8)) allow *faster growth* in representational power with depth, efficiently capturing higher-order effects.
2. **However**, very high-degree activations in deep networks may introduce *training instability*, such as exploding gradients.
3. **Empirically**, we observe that for the same representable order, both variants achieve similar accuracy. Yet, **first-order activations (Eq. (5)) tend to be more stable**, especially on noisy or real-world datasets.

Therefore, we recommend:

- Using Eq. (5) when stability is a concern or when higher-order context effects are not evident.
- Using Eq. (8) to reduce network depth when higher-order interactions are known or hypothesized.
- Combining them for a tunable trade-off.

This modularity makes DeepHalo a unified and controllable framework for context-dependent choice modeling, in both feature-rich and featureless settings. We will add a guidance paragraph in the revision to clarify these trade-offs.

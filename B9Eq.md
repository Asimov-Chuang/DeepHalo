We appreciate Reviewer 59ua’s positive feedback. To address the questions and concerns about our method, we provide point-wise responses as follows.

#### 1. Clarifying Section 4.1 — why Equation (8) needs only **log₂ l** layers to model *l*‑th‑order interactions

We agree that Section 4.1 should state this property explicitly. Currently, this is mentioned in 4.2 and the explanation is defered in the appendix. 
In the revision we will:

1. **Add a sentence to Section 4.1**  
   > “Because each Equation (8) layer doubles the maximum attainable interaction order, an *l*‑th‑order effect can be represented with at most ⌈log₂ l⌉ stacked layers.”

2. **Provide an intuitive explanation** (now in the appendix, to keep the main text concise):  
   - A single (8) layer captures up to second‑order effects.  
   - Stacking two layers composes these interactions, yielding up to 4‑th‑order effects, and so on.  
   - After *k* layers the highest order is $2^{k}$, giving the logarithmic bound.


#### 2. Practical guidance on when to use Equation (5) vs Equation (8)

We appreciate this insightful question. One of the strengths of the DeepHalo architecture is its structural flexibility: the activation function σ can be instantiated as any polynomial function, and Equations (5) and (8) are not mutually exclusive. In practice, they can be freely mixed—for example, stacking layers with linear and quadratic activations can achieve effective interaction orders such as $((1+1+1)\times2+1)\times2$, offering nuanced control over expressivity.

In general:

1. **Higher-degree activations** (like Eq. (8)) allow *faster growth* in representational power with depth, efficiently capturing higher-order effects.
2. **However**, very high-degree activations in deep networks may introduce *training instability*, such as exploding gradients.
3. **Empirically**, we observe that for the same representable order, both variants achieve similar accuracy. Yet, **first-order activations (Eq. (5)) tend to be more stable**, especially on noisy or real-world datasets.

Therefore, we recommend:

- Using Eq. (5) when stability is a concern or when higher-order context effects are not considered.
- Using Eq. (8) to reduce network depth when higher-order interactions are known or hypothesized.
- Combining them for a tunable trade-off.

This modularity makes DeepHalo a unified and controllable framework for context-dependent choice modeling, in both feature-rich and featureless settings. We will add a guidance paragraph in the revision to clarify these trade-offs.

#### 3. Clarifying what makes DeepHalo more interpretable than prior neural context-effect models

We appreciate the reviewer’s thoughtful question. While many prior models (e.g., FATE, TCNet) acknowledge context effects, none have **formally defined feature-based halo effects in a utility-decomposable framework**. Our work is, to the best of our knowledge, the first to do so.

Concretely, given an offer set $\{A, B, C, D\}$, when estimating how the subset $\{A, B\}$ influences the utility of item $C$, **the function must depend only on the features of $A$, $B$, and $C$**, and **must not involve irrelevant items like $D$**. This subset-specific conditioning is critical for interpretability, but is not enforced in FATE, TCNet. Models such as FATE and TCNet attempt to learn context by directly entangling all item features. As the reviewer notes, their outputs can be interpreted via Eq. (10) as halo effects. However, **the interaction order in these models is uncontrolled**, and recovering a faithful decomposition would require evaluating up to $|S|-1$ orders—leading to exponentially many terms and breaking interpretability.

The issue lies in their architecture. FATE combines global context with individual features but cannot restrict interaction depth. TCNet’s attention mechanism inherently mixes all items, effectively modeling full-order interactions by design. As a result, these models **do not formulate context effects in a structured, decomposable way**, making interpretation difficult.

DeepHalo resolves this with a **modular design**:

- Each layer introduces **exactly one additional interaction order or $\times 2$ using quadratic activation**.
- The **maximum effect order is bounded** by the network depth $L$.
- Residual and polynomial structures guarantee **halo-decomposable utility functions**. We will give a simple example later for better illustration.

This enables users to control expressivity, avoid overfitting, and ensure that only desired-order interactions are modeled. For example, modeling third-order effects over a catalog of 100 items without such control would require handling over 160,000 subsets—computationally prohibitive and interpretively infeasible.

**In summary**, DeepHalo is the first model to:

1. Formally define feature-based halo effects, 
2. Provide a general decomposition-compatible framework,
3. Architecturally enforce subset attribution and interaction-order control.

These properties are essential for **reliable and scalable interpretability**, and are not achievable with existing models.

#### 4. Clarifying the distinction between DeepHalo and SDA [Rosenfeld et al., 2020]

Thank you for the insightful observation. While DeepHalo shares some high-level similarities with set-dependent embedding (SDA) approaches such as [Rosenfeld et al., 2020, Eq. (2)], we would like to emphasize several key distinctions in both motivation and design.

DeepHalo is fundamentally grounded in **halo effect theory**, which prioritizes interpretable higher-order interactions. While incorporating context information (e.g., through contextual functions like $\phi$) is straightforward in both frameworks, **interpretability requires controlling the maximum order of interactions**—a key principle not enforced by SDA.

As the reviewer rightly noted, if Eq. (10) were applied to arbitrary context-based architectures (e.g., SDA), the number of halo terms would grow **exponentially** with the size of the offer set. This would result in a **combinatorial explosion** of interaction terms that are computationally infeasible and practically uninterpretable.

In contrast:

- **SDA and related models** aggregate item features in a **fully entangled** way.
  - This makes it **virtually impossible to recover halo effects** without enumerating all subsets—an exponentially large and ill-posed task.
- **DeepHalo**, by design:
  - Uses **residual connections and polynomial activations** to explicitly control and construct the interaction order.
  - Ensures that utilities are **halo-decomposable** and that **item features remain disentangled**.
  - Caps model expressiveness to low-order interactions, preserving **interpretability and generalization**.

In short, while SDA enables generic set-dependent context modeling, **DeepHalo provides a theoretically-grounded and architecturally-enforced approach to interpretable, order-controlled context effects**, which is essential for practical discrete choice applications.


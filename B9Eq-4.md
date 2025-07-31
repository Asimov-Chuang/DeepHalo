We appreciate Reviewer B9Eq’s positive feedback. To address the questions and concerns about our method, we provide point-wise responses as follows.

### Weakness and Limitation

> Q: Could be clearer what the advantages are of the proposed method vs existing neural context effect models

A: The key advantage of DeepHalo lies in its ability to **explicitly control the maximum order of context (halo) effects**, enabling a flexible trade-off between model expressiveness and interpretability—something existing neural models lack.

- First-order models (e.g., CMNL [Yousefi Maragheh et al., 2020], FETA [Pfannschmidt et al., 2022]) are interpretable but limited to pairwise interactions and cannot scale to higher-order effects due to computational constraints.
- High-capacity models (e.g., FATE [Pfannschmidt et al., 2022], ResLogit [Wong and Farooq, 2021], and TCNet [Wang et al., 2023]) are expressive but entangle all item features, making it infeasible to control interaction order. Recovering meaningful low-order effects from these models requires evaluating up to $O(2^{|S|-1})$ terms.

In contrast, DeepHalo models context effects in a **structured and decomposable** way. It allows practitioners to specify the maximum interaction order $K$, with a complexity of $O(2^K)$ to recover the effect terms, making it both **efficient and interpretable**. This is especially useful in real-world settings like recommendation and bundle pricing, where lower-order effects are often sufficient and desirable.

> Q: The title, though, is a bit generic, and could apply to several of the cited papers; ...

A: Thanks for your suggestion! We will consider adding some unique elements to our title that help distinguish it from other context-dependent model papers.

> Q: How does the sample complexity and need for hyperparameter tuning compare to existing approaches?

A: Regarding sample complexity and data efficiency, we refer the reviewer to our response to Reviewer Unm8【CHECK】, where we provide an experiment evaluating model performance under varying data proportions.

As for the hyperparameter tuning, we find empirically that DeepHalo is relatively easy to tune. **Once the maximum effect order is specified, the overall effect complexity is fixed**, which is the main tuning difference compared to existing approaches. Additional parameters are then introduced only to improve the approximation accuracy of these effects, rather than to increase model capacity arbitrarily. 

#### 1. Clarifying Section 4.1 — why Equation (8) needs only $\log_2l$ layers to model $l$‑th‑order interactions

We agree that Section 4.1 should state this property explicitly. Currently, this is mentioned in Section 4.2, and the explanation is deferred in the appendix. 
In the revision, we will:

1) **Add a sentence to Section 4.1**  
   > “Because each Equation (8) layer doubles the maximum attainable interaction order, an *l*‑th‑order effect can be represented with at most ⌈log₂ l⌉ stacked layers.”

2) **Provide an intuitive explanation**  
   - A single layer (Equation (8)) captures up to second‑order effects.  
   - Stacking two layers composes these interactions, yielding up to 4‑th‑order effects, and so on.  
   - After *k* layers the highest order is $2^{k}$, giving the logarithmic bound.


#### 2. Practical guidance on when to use Equation (5) vs Equation (8)

We appreciate this insightful question. Generally, we recommend:

- **Using Eq. (5) when training stability is a concern or when higher-order context effects are not considered.**
- **Using Eq. (8) to reduce network depth when higher-order interactions are considered.**
- Combining them for a tunable trade-off.

Here we provide some details for further intuition. One of the strengths of the DeepHalo architecture is its structural flexibility: the activation function σ can be instantiated as any polynomial function. (5) and (8) can be used interchangeably within the same model. In practice, they can be freely mixed—for example, stacking layers with linear and quadratic activations can achieve effective interaction orders such as $((1+1+1)\times2+1)\times2$, offering nuanced control over expressivity.

- Higher-degree activations (like Eq. (8)) allow *faster growth* in representational power with depth, efficiently capturing higher-order effects.
- However, very high-degree activations in deep networks may introduce *training instability*, such as exploding gradients.
- Empirically, we observe that for the same representable order, both variants achieve similar accuracy. Yet, first-order activations (Eq. (5)) tend to be more stable, especially on noisy or real-world datasets.

This modularity makes DeepHalo a unified and controllable framework for context-dependent choice modeling, in both feature-rich and featureless settings. We will add a guidance paragraph in the revision to clarify these trade-offs.

#### 3. Clarifying what makes DeepHalo more interpretable than prior neural context-effect models
We appreciate the reviewer’s thoughtful question. While many prior models can interpret the effects using Eq. (10), this is under two premises:
- The model is expressive enough to include all interaction orders up to $(|S|-1)$ instead of some specific orders (which first-order models like CMNL and FETA cannot achieve)
- Existing expressive models like FATE and TCNet require a computational complexity of $O(2^{|S|})$ for recovering all halo effects in the model.

In contrast, **our expressive DeepHalo model can capture up to a specific order and only needs $O(2^K)$ computational time to obtain $k$-order interactions**, which provide more interpretability. For example, modeling third-order effects over a catalog of 100 items without such control would require handling over 160,000 subsets, which is computationally prohibitive and interpretively infeasible.

Besides, the exact-interaction-order control inherited in our model is of practical interest. As a concrete example, consider a choice set $\{A, B, C, D\}$. Then estimating how the subset $\{A, B\}$ influences the utility of item $C$, the function must depend only on the features of $A$, $B$, and $C$, and must not involve irrelevant items like $D$. This subset-specific conditioning is critical for interpretability, but is not enforced in existing context-dependent ML models. 

The reason they cannot capture exactly up to $k$-order interactions is that they attempt to learn context by directly entangling all item features. For instance, FATE combines global context with individual features but cannot restrict interaction depth. TCNet’s attention mechanism inherently mixes all items, effectively modeling full-order interactions by design. As a result, these models do not formulate context effects in a structured, decomposable way, making interpretation difficult.

On the contrary, **our model achieves more interpretability with a modular design**:
- Each layer introduces exactly one additional interaction order or $\times 2$ using quadratic activation.
- The maximum effect order is bounded by the network depth $L$.
- Residual and polynomial structures guarantee halo-decomposable utility functions. We will give a simple example later for better illustration.


#### 4. Clarifying the distinction between DeepHalo and SDA [Rosenfeld et al., 2020]

Thank you for the insightful observation. While DeepHalo shares some high-level similarities with set-dependent embedding (SDA) approaches such as [Rosenfeld et al., 2020, Eq. (2)], we would like to emphasize several key distinctions in both motivation and design, some are similar to Point 3.

DeepHalo is fundamentally grounded in halo effect theory, which prioritizes **interpretable higher-order interactions**. While simply incorporating context information (e.g., through contextual functions like $\phi$) is straightforward, interpretability requires controlling the maximum order of interactions—a key principle not enforced by SDA.

As the reviewer rightly noted, if Eq. (10) were applied to arbitrary context-based architectures (e.g., SDA), the number of halo terms would grow exponentially with the size of the offer set. This would result in a combinatorial explosion of interaction terms that are computationally hard and practically uninterpretable.

In contrast:

- SDA and related models aggregate item features in a fully entangled way.
  - This makes it virtually impossible to recover the halo effect without enumerating all subsets—an exponentially large and ill-posed task.
- DeepHalo, by design:
  - Uses residual connections and polynomial activations to explicitly control and construct the interaction order.
  - Ensures that utilities are halo-decomposable and that item features remain disentangled.
  - Caps model expressiveness to low-order interactions, preserving interpretability and generalization.

In short, while SDA enables generic set-dependent context modeling, DeepHalo provides a theoretically-grounded and architecturally-enforced approach to **interpretable, order-controlled context effects**, which is essential for practical discrete choice applications. 

For points 3 & 4, we refer the reviewer to Appendix A2.2 and A2.3, as well as the illustrative example in Question 2 of our response to reviewer Unm8, for a detailed explanation of how DeepHalo supports structured, decomposable modeling of context effects.





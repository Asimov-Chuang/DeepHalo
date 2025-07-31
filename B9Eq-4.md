We appreciate Reviewer B9Eq’s positive feedback. To address the questions and concerns about our method, we provide point-wise responses as follows.

### Weakness


Novelty: We appreciate the reviewer’s thoughtful question. While many prior models (e.g., FATE, TCNet) acknowledge context effects, none have formally defined feature-based halo effects in a utility-decomposable framework. Our work is, to the best of our knowledge, the first to do so.

In summary, DeepHalo is the first model to:

1. Formally define feature-based halo effects. 
2. Provide a general decomposition-compatible framework.
3. Architecturally enforce subset attribution and interaction-order control.

These properties are essential for reliable and scalable interpretability and are not achievable with existing models.


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

If the number of alternatives is small and the computational limit permits, use (5). Otherwise use (8). In fact, xxxxxxx we can use a mixture of linear and quadratic activations as well. [I remember here need to first directly answer the reviewer's question (your conclusion/recommendation) then give example.]

We appreciate this insightful question. One of the strengths of the DeepHalo architecture is its structural flexibility: the activation function σ can be instantiated as any polynomial function, and Equations (5) and (8) are not mutually exclusive. In practice, they can be freely mixed—for example, stacking layers with linear and quadratic activations can achieve effective interaction orders such as $((1+1+1)\times2+1)\times2$, offering nuanced control over expressivity.

In general:

- Higher-degree activations (like Eq. (8)) allow *faster growth* in representational power with depth, efficiently capturing higher-order effects.
- However, very high-degree activations in deep networks may introduce *training instability*, such as exploding gradients.
- Empirically, we observe that for the same representable order, both variants achieve similar accuracy. Yet, **first-order activations (Eq. (5)) tend to be more stable**, especially on noisy or real-world datasets.

Therefore, we recommend:

- Using Eq. (5) when stability is a concern or when higher-order context effects are not considered.
- Using Eq. (8) to reduce network depth when higher-order interactions are considered.
- Combining them for a tunable trade-off.

This modularity makes DeepHalo a unified and controllable framework for context-dependent choice modeling, in both feature-rich and featureless settings. We will add a guidance paragraph in the revision to clarify these trade-offs.

#### 3. Clarifying what makes DeepHalo more interpretable than prior neural context-effect models
We appreciate the reviewer’s thoughtful question. While many prior models can interpret the effects using Eq. (10), this is under two premises:
- The model is expressive enough to include all interaction orders up to $(|S|-1)$ instead of some specific orders (which first-order models like CMNL and FETA cannot achieve)
- Existing expressive models like FATE and TCNet require a computational time of $O(2^|S|)$.

In contrast, our expressive DeepHalo model can capture up to a specific order and only needs $O(2^k)$ computational time to obtain $k$-order interactions, which provide more interpretability. For example, modeling third-order effects over a catalog of 100 items without such control would require handling over 160,000 subsets, which is computationally prohibitive and interpretively infeasible.

Besides, the exact-interaction-order control inherited in our model is of practical interest. As a concrete example, consider a choice set $\{A, B, C, D\}$. hen estimating how the subset $\{A, B\}$ influences the utility of item $C$, the function must depend only on the features of $A$, $B$, and $C$, and must not involve irrelevant items like $D$. This subset-specific conditioning is critical for interpretability, but is not enforced in existing context-dependent ML models. 

The reason they cannot capture exactly up to $k$-order interactions is that they attempt to learn context by directly entangling all item features. For instance, FATE combines global context with individual features but cannot restrict interaction depth. TCNet’s attention mechanism inherently mixes all items, effectively modeling full-order interactions by design. As a result, these models do not formulate context effects in a structured, decomposable way, making interpretation difficult.

On the contrary, our model achieve more interpretability with a modular design:
- Each layer introduces exactly one additional interaction order or $\times 2$ using quadratic activation.
- The maximum effect order is bounded by the network depth $L$.
- Residual and polynomial structures guarantee halo-decomposable utility functions. We will give a simple example later for better illustration.


#### 4. Clarifying the distinction between DeepHalo and SDA [Rosenfeld et al., 2020]

Thank you for the insightful observation. While DeepHalo shares some high-level similarities with set-dependent embedding (SDA) approaches such as [Rosenfeld et al., 2020, Eq. (2)], we would like to emphasize several key distinctions in both motivation and design.

DeepHalo is fundamentally grounded in halo effect theory, which prioritizes interpretable higher-order interactions. While incorporating context information (e.g., through contextual functions like $\phi$) is straightforward in both frameworks, interpretability requires controlling the maximum order of interactions—a key principle not enforced by SDA.

As the reviewer rightly noted, if Eq. (10) were applied to arbitrary context-based architectures (e.g., SDA), the number of halo terms would grow exponentially with the size of the offer set. This would result in a combinatorial explosion of interaction terms that are computationally infeasible and practically uninterpretable.

In contrast:

- SDA and related models aggregate item features in a fully entangled way.
  - This makes it virtually impossible to recover halo effect without enumerating all subsets—an exponentially large and ill-posed task.
- DeepHalo, by design:
  - Uses residual connections and polynomial activations to explicitly control and construct the interaction order.
  - Ensures that utilities are halo-decomposable and that item features remain disentangled.
  - Caps model expressiveness to low-order interactions, preserving interpretability and generalization.

In short, while SDA enables generic set-dependent context modeling, DeepHalo provides a theoretically-grounded and architecturally-enforced approach to interpretable, order-controlled context effects, which is essential for practical discrete choice applications.


(Change to explain why other models cannot do this)
### 5. Simple Example: Utility Decomposition and Order Growth**
The key intuition is that every recursion depth l in Eqs. (4)–(5) adds **exactly one additional order of interaction** while preserving all lower‑order terms via the residual connection. The process is the same as Appendix A 2.2 and A 2.3. To better facilitate the understanding, we provide a simple example below.


Consider an offer set ${j, k, l}$ and let $\phi$ be an identity mapping. We focus on the change of $z_{j}$ in DeepHalo.
1. **Pairwise (1‑st order) layer.**  
   The first layer aggregates the linear summary vector $\bar Z^{1}$ from raw embedding vector $z^{0}$ and modulates them with $z_{j}^{0}$. This yields  
   $z_{j}^{1}=z_{j}^{0} + \frac1H\sum_{h}\bar Z_{h}^{1} \cdot z_{j}^{0}$, which contains only **pairwise interactions** between the target alternative $j$ and every other item $k, l$ in the set.
   If we denote $f_j$ as any function containing only item j's feature information, the above output can be formulated as,
   $z_{j}^{1} = f_j + \frac1H\sum_{h}(f_j + f_k + f_l) \cdot f_j$, thus it contains zero order term $f_j$ and first order term $f_k \cdot f_j$ and $f_l \cdot f_j$, second order term $f_j \cdot f_k \cdot f_l$ won't appear. For simplicity, we use $f_{ab}$ to represent any term like $f_a \cdot f_b$ and $f_b \cdot f_a$, which only contains the features of item a and item b.

2. **Second‑order layer.**  
   After the first layer, $z_{k}^{1}$ and $z_{l}^{1}$ already embed their pairwise effects with every other item.  
   The second layer builds
    
   $\bar Z^{2}=\tfrac1S\sum_{m}W^{2}z_{m}^{1}$, which contains term $f_j, f_k, f_l, f_{jk}, f_{jl}, f_{kl}$.
   
   $z_{j}^{2}=z_{j}^{1}+\tfrac1H\sum_{h} \bar Z_{h}^{2} \cdot z_{j}^{0}$ which is analogous to $(f_j + f_{jk} + f_{jl}) + \sum_h(f_j + f_k + f_l + f_{jk} + f_{jl} + f_{kl}) \cdot f_j$,
   which brings about second order term $f_{jkl} = f_{kl} \cdot f_{j}$. It can be interpreted as the effect of ${k,l}$ on $j$.

This example illustrates how we can add up orders by increasing the number of layers and get a decomposable utility representation. If we change the $\cdot$ in layers to some other polynomial function, the term order will grow faster, like in the first layer we change to $(f_j + f_k + f_l)^2 \cdot f_j$, in the sum will directly bring about the second order term $f_{jkl}$. The quadratic activation naturally brings a $log_2$ form depth requirement.


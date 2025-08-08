We sincerely appreciate your recognition of our problem modeling, methodological soundness, real-world validation, and thoughtful baseline selection. 

To address the concerns, we provide detailed point-wise responses below.


> **Q: As far as I can see, the specific decomposition of the utility function which DeepHalo uses mirrors FETA’s almost exactly...**
 
A: The key methodological contribution of DeepHalo lies in introducing a **structured and order-controllable framework** that balances expressiveness and interpretability—a capability that is lacking in existing context-dependent choice models. Among context-dependent models, 

- First-order models such as CMNL (Yousefi Maragheh et al., 2020), low-rank Halo MNL (Ko and Li, 2024), and FETA (Pfannschmidt et al., 2022) are **inherently restricted to pairwise interactions**. For instance, FETA proposes a decomposition that resembles the halo effect, but in practice, it is limited to first-order pairwise interactions. The architecture is not scalable to higher-order effects, as doing so would require exponentially large tensor storage and computational resources, rendering it impractical for real-world applications.

- Other higher-order models, such as FATE (Pfannschmidt et al.,2022) and TCNet (Wang et al., 2023), attempt to learn context effects by **directly entangling all item features** within the offer set. While their outputs can be retrospectively interpreted as halo effects via Eq. (10), the interaction order is not explicitly controlled. Recovering a meaningful decomposition in these models would require computational complexity on the order of $O(2^{∣S∣−1})$, due to the exponential number of possible interaction terms. This limitation stems from their architectural design: context effects are not modeled in a structured or decomposable way. Specifically, FATE aggregates all item features into a set embedding, which is then combined with individual item features through complex nonlinear transformations, obscuring the origin and order of interactions. TCNet, similarly, uses attention mechanisms where the softmax denominator implicitly mixes information from all items in the offer set, making it difficult to isolate individual effects. As a result, neither model can recover a truly decomposable utility function. 

In contrast, DeepHalo is explicitly designed to address this gap by enabling **precise control over the maximum order of halo interactions**. To the best of our knowledge, it is the first model that supports flexible trade-offs between model complexity and interpretability in this context.

In e-commerce applications such as recommendation systems and bundle pricing, practitioners are often primarily interested in lower-order context effects, which are more stable, interpretable, and practically useful. While expressive models can theoretically recover such effects, they do so implicitly as part of an entangled representation involving many higher-order interactions. Consequently, low-order effects cannot be meaningfully isolated. DeepHalo offers a principled approximation that explicitly models limited-order halo effects. It allows users to control the maximum interaction order $K$, with computational complexity only $O(2^K)$, offering a **flexible trade-off between efficiency and expressiveness**.

Our contribution goes beyond what Deep Set offers. While DeepSets provides permutation invariance, DeepHalo integrates domain-specific insights from discrete choice theory and the halo literature, along with architectural elements like residual connections and polynomial activations. This enables interpretable, high-order interaction modeling in a principled and controllable way.


> **Q: Furthermore, I’m not convinced that the proposed network architecture replicates the utility decomposition the paper aims for (line 189)...**

A: Thank you for raising this point. The key intuition is that every recursion depth l in Eqs. (4)–(5) adds **exactly one additional order of interaction** while preserving all lower‑order terms via the residual connection. The process is the same as Appendix A 2.2 and A 2.3. To better facilitate the understanding, we provide a simple example below.

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

> **Q: The purpose of $(-1)^{|T|-|R|}$.**

A: The $(−1)^{∣T∣−∣R∣}$ factor in Eq. (3) is the **inclusion–exclusion inversion coefficient** on the Boolean lattice of subsets. Our $v_j(⋅)$ must invert the cumulative‑sum relation $u_j(X_S)=\sum_{T\subset S\setminus\{j\}} v_j(X_{T\cup\{j\}})$; effects contributed to many supersets are otherwise over‑counted. Alternating signs guarantee that terms appearing an even number of times cancel and those appearing an odd number of times net to the correct single contribution, yielding a unique decomposition. 

> **Q: About Performance Difference between DeepHalo and TCnet...**

A: Thanks for the insightful observation. We acknowledge that similar performance between DeepHalo and TCNet is expected; however, we wish to emphasize a key difference in their modeling philosophies.

While TCNet can approximate utility decompositions (maximum order), its Transformer architecture lacks control over the maximum interaction order. Attention layers aggregate across the full set, leading to large quantities of high-order effects that are difficult to interpret. In contrast, DeepHalo explicitly controls interaction order through its recursive structure, enabling better interpretability and identifiability.

Given that low-order interactions often dominate in real-world settings, it is not surprising that TCNet matches DeepHalo's accuracy. However, DeepHalo achieves this with controlled expressiveness, offering a more interpretable alternative. Rather than outperforming black-box models in accuracy, our main goal is to provide a principled, order-aware design that balances predictive performance with transparency.

> **Q: About data efficiency**

| Ratio (%) | CMNL  | MNL   | FATE   | MLP   | MixedMNL | TCnet | **DeepHalo** |
|-----------|-------|-------|--------|-------|-----------|--------|--------------|
| 10        | 1.574 | 1.898 | 1.577 | 1.571 | 1.674     | 1.589 | **1.566**     |
| 20        | 1.567 | 1.869 | 1.576 | 1.563 | 1.617     | 1.573 | **1.558**     |
| 30        | 1.564 | 1.844 | 1.574 | 1.557 | 1.590     | 1.569 | **1.551**     |
| 40        | 1.562 | 1.820 | 1.574 | 1.554 | 1.576     | 1.568 | **1.548**     |
| 50        | 1.561 | 1.779 | 1.576 | 1.550 | 1.566     | 1.570 | **1.544**     |
| 60        | 1.562 | 1.760 | 1.576 | 1.545 | 1.563     | 1.551 | **1.537**     |
| 70        | 1.555 | 1.742 | 1.576 | 1.542 | 1.560     | 1.539 | **1.532**     |
| 80        | 1.544 | 1.726 | 1.576 | 1.537 | 1.558     | 1.560 | **1.528**     |
| 90        | 1.550 | 1.699 | 1.576 | 1.539 | 1.555     | 1.549 | **1.526**     |
| 100       | 1.550 | 1.726 | 1.560 | 1.537 | 1.558     | 1.538 | **1.528**     |

To evaluate data efficiency, we subsample the SFOshop dataset—known for strong context effects—at ratios from 10% to 100%, training all neural models under a strict ~1k parameter budget.

Across all settings, **DeepHalo consistently achieves the lowest Test NLL**, showing strong generalization even with limited data. Its Test NLL steadily improves with more data, while other models (TCNet, MLP, FATE) either plateau early or fluctuate.


### **Clarification**
The expression $u_j(x_j, X_R)$ in Eq. (3) refers to the same function as $u_j(X_{R \cup \{j\}})$ in Eq. (2), and we will revise the notation for consistency. We will also correct the duplicate citation of Deep Sets [Zaheer et al., 2018], fix the misspelling of FETA and FATE (“E” stands for “evaluate”), and use $|S|$ consistently to denote set size. These issues will be addressed in the final version. We appreciate the reviewer’s careful reading.

### **Additional Experiment**
Thank you for your response. We truly appreciated the insightful exchange, and we are also very grateful that you are willing to raise your score.

Regarding your comment:

> “I understand the argument that for practitioners using these models there are advantages from being able to control the order of interactions, but I would have expected more significant experimental evidence to support this claim.”

We would like to provide an additional experiment to further support this point.

We trained an MLP on a synthetic featureless choice dataset with assortment size 10, where the ground-truth halo effect has a maximum order of 2. We then applied our proposed algorithm to recover all orders of halo effects from the trained MLP. Using the recovered effects, we reconstructed the utility and choice probabilities, and varied the maximum order of included effects to study how the in-sample performance depends on the maximum order we use to reconstruct the utility.

We repeated the same procedure for a DeepHalo model trained on the same dataset, whose architecture was limited to capturing up to second-order effects.

The results are shown in the table below. We observe that:

- The MLP, due to the lack of explicit order control, required up to order 9 effects to reach optimal performance.
- In contrast, DeepHalo achieved optimal performance already at order 2, demonstrating the effectiveness of its explicit order control for low-order approximation.

Interestingly, we observed that once a model captures higher-order effects, the lower-order components often become functionally dependent on them. These effects operate jointly—lower-order terms contribute meaningfully only in the presence of complementary higher-order interactions. Simply adding a lower-order term without accounting for the captured higher-order structure can even degrade performance (e.g., MLP from order 2 to 3, DeepHalo from order 0 to 1). This demonstrates that post hoc truncation of higher-order effects to isolate lower-order behavior can be misleading and harmful. In contrast, explicit order control allows the model to learn faithful low-order approximations without relying on fragile truncation procedures.


We believe this experiment further highlights the practical importance of having interpretable, order-controllable approximations like DeepHalo.

| Max Effect Order | MLP In-sample NLL | DeepHalo In-sample NLL |
|----------------------------|---------------|---------------------|
| 0                          | 3.7148        | 2.3020              |
| 1                          | 3.2303        | 4.1364              |
| 2                          | 0.7643        | 0.0922              |
| 3                          | 1.6429        | 0.0922              |
| 4                          | 0.8172        | 0.0922              |
| 5                          | 0.6719        | 0.0922              |
| 6                          | 0.3280        | 0.0922              |
| 7                          | 0.1502        | 0.0922              |
| 8                          | 0.0958        | 0.0922              |
| 9                          | 0.0910        | 0.0922              |




Dear Reviewer,

To provide further experimental evidence supporting the benefit of controlling interaction order, we conducted the following controlled experiment.

We generated a dataset of 10 items where the ground-truth utility function includes interaction effects up to the second order only. This setting is meant to mimic practical scenarios in which interaction effects are limited to low orders. We compared two modeling approaches for estimating first- and second-order effects:

(1) DeepHalo: Fit our model with architecture explicitly constrained to capture up to second-order interactions. Report the in-sample negative log-likelihood (NLL) of the observed choices using the fitted model.

(2) MLP with Post Hoc Truncation: Fit a standard MLP. Use Equation (10) to extract interaction effects of all orders, then truncate post hoc to include only terms up to order $k$. Evaluate the in-sample NLL using only the truncated interaction effects.

The in-sample NLL results for various truncation orders k are shown below:


| Truncation Order $k$| MLP In-sample NLL | DeepHalo In-sample NLL |
|----------------------------|---------------|---------------------|
| 2                          | 0.7643        | 0.0922              |
| 3                          | 1.6429        | 0.0922              |
| 4                          | 0.8172        | 0.0922              |
| 5                          | 0.6719        | 0.0922              |
| 6                          | 0.3280        | 0.0922              |
| 7                          | 0.1502        | 0.0922              |
| 8                          | 0.0958        | 0.0922              |
| 9                          | 0.0910        | 0.0922              |

We observe that:

- The MLP matches DeepHalo’s performance only when all interaction terms—including spurious higher-order ones—are included (i.e., no truncation).

- Interestingly, increasing the truncation order beyond the true interaction order can worsen performance (e.g., from $k=2$ to $k=3$).

This happens because the interaction effects of different orders interact nonlinearly. When higher-order terms are included but not properly estimated, they can distort the estimation of lower-order effects, leading to biased predictions. In contrast, DeepHalo’s explicit order control ensures a faithful low-order approximation without relying on fragile post hoc truncation.

We hope this simple experiment clarifies the practical importance of interpretable, order-controllable models like DeepHalo. Thank you again for your thoughtful feedback.

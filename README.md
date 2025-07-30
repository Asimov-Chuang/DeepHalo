We sincerely appreciate the reviewer Uum8’s recognition of our modeling problem, 
methodological soundness, real-world validation, and thoughtful baseline selection. 

To address the reviewer’s concerns on novelty and interpretability, 
we provide detailed point-wise responses below.

### 1&nbsp;&nbsp;Beyond FETA and other context based choice models: controllable modeling of higher‑order effects  
Among context based models, 

- **first-order models** such as CMNL (Yousefi Maragheh et al., 2020), low-rank Halo MNL (Ko and Li, 2024) and FETA (Pfannschmidt et al., 2022) are only capable of capturing first-order interaction effects. Even though FETA proposed the utility decomposition method similar to Halo effect, in practice, it evaluates only first‑order (pairwise) terms because enumerating higher‑order subsets is combinatorially expensive, especially when facing large assortment sizes. This severely limits their ability to model complex choice behavior involving higher-order item interactions.

- **Other context based models**, such as FATE (Pfannschmidt et al., 2022) and TCnet (Wang et al., 2023), try to learn an overall context information by directly entangling all items' features together. While we acknowledge that these models, as the reviewer has pointed out, can be explained by Halo effects recovered with equation (10), a critical and often overlooked issue is that **the maximum order of interactions is uncontrollable and we need to recover up to |S - 1| order to interpret these models, otherwise the halo model recovered by (10) is different from the one you got from training**. This results in exponentially many context terms, which fundamentally harms interpretability. The dilemma lies in the model architecture. For example, the $U^{'}$ function in FATE entangles all the item features, then concatenates it with the original feature and does transformations with it. The utility calculation does have context information, but cannot control the maximum effect order. In TCnet, transformer-based attention mechanism inherently computes a softmax over all items — meaning each item's representation is a function of the entire set, pushing the interaction order to the maximum by design. In one word, these models don't formulate context effect in a decomposible way.

Therefore, first-order models have interpretability but lack of expressiveness, while other models have expressiveness but cannot control maximum effect order and is difficult to interpret. DeepHalo is designed to fill this gap, which manages to model higher Halo effects up to any order we want. How do we do?

- **Formally Define Feature-based high order Halo Effect** Our work is, to the best of our knowledge, the first to provide such a definition. Specifically, in an offer set\{A, B, C, D\}, when estimating how the subset \{A, B\} influences the utility of item C, the function representing this effect must depend only on the features of A, B, C—and must not involve features from irrelevant items like D. Violating this would render the subset effect uninterpretable. This highlights why indiscriminately entangling all item features within the offered set inevitably results in uncontrolled and often maximal-order interactions.

- **Decomposible Utility Representation and Controllable Maximum Effect Order** In Appendix A, we provide derivation showing the utilities output by DeepHalo can be decomposed by the way in Line 189. We have proved that the maximum order is constrained by the model depth, and the increasing speed of effect order is controlled by which activation function you choose. **We are glad that the reviewer is interested in this part, and we will elaborate this later.**
  
- **Identifiable Effect Recover Algorithm without Formulate Linear System** In previous literature, people need to formulate the whole linear system to get the halo effects, which is inefficient when the assortment size is big. With equation (3) and (10), we can skip this step. **We know the reviewer has some questions in this part, and we will elaborate them later.**

This is also why our model is not a simple imitation of DeepSets. DeepSets merely provides a foundational tool for implementing permutation invariant models. Building upon this, we integrate insights from discrete choice theory and the halo effect literature, and further incorporate architectural innovations such as residual connections and polynomial activations. These design choices make it possible to learn high-order interaction effects in a structured and controllable manner.

### **2&nbsp;&nbsp;Utility Decomposition and Order Growth**
We thank the author for raising this point.  The key intuition is that every recursion depth l in Eqs. (4)–(5) adds **exactly one additional order of interaction** while preserving all lower‑order terms via the residual connection. The process is same with Appendix A 2.2 and A 2.3, to better help the reviewer, we provide an simple example here. Formally, **let's take offer set ${j, k, l}$ as an example and take $\phi$ as identity mapping for simplicity**:

1. **Pairwise (1‑st order) layer.**  
   The first layer aggregates the linear summaries $\bar Z^{1}$ from raw embeddings $z^{0}$ and modulates them with $z_{j}^{0}$. This yields  
   $z_{j}^{1}=z_{j}^{0} + \frac1H\sum_{h}\bar Z_{h}^{1} \cdot z_{j}^{0},$  
   which contains only **pairwise interactions** between the target alternative \(j\) and every other item \(k\) in the set.  (Derivation lines 90‑116 in the Paper)
   if we take notation $f_j$ to represent any function containing only item j's feature information, the above process can be formulated as,
   $z_{j}^{1} = f_j + \frac1H\sum_{h}(f_j + f_k + f_l) \cdot f_j,$ thus it contains zero order term $f_j$ and first order term $f_k \cdot f_j$ and $f_l \cdot f_j$, second order term $f_j \cdot f_k \cdot f_l$ won't appear. For simplicity, we use $f_{ab}$ to represent any term like $f_a \cdot f_b$ and $f_b \cdot f_a$, which only contains the features of item a and item b.

2. **Second‑order layer.**  
   Consider a choice set $\{j,k,\ell\}$.  After the first layer, $z_{k}^{1}$ and $z_{\ell}^{1}$ already embed their pairwise effects with every other item.  
   The second layer builds
    
   $\bar Z^{2}=\tfrac1S\sum_{m}W^{2}z_{m}^{1}$, which contains term $f_j, f_k, f_l, f_{jk}, f_{jl}, f_{kl}$
   
   $z_{j}^{2}=z_{j}^{1}+\tfrac1H\sum_{h} \bar Z_{h}^{2} \cdot z_{j}^{0}$ which is analogous to $(f_j + f_{jk} + f_{jl}) + \sum_h(f_j + f_k + f_l + f_{jk} + f_{jl} + f_{kl}) \cdot f_j$
   
   which brings about second order term $f_{jkl} = f_{kl} \cdot f_{j}$. It can be interpreted as effect of ${k,l}$ on $j$

then it's clear how we can add up order by increasing layer and get a decomposable utility representation. If we change the $\cdot$ in layers to some other polynomial function the term order will grow faster, like in first layer we change to $(f_j + f_k + f_l)^2 \cdot f_j$ in the sum will directly bring about second order term $f_{jkl}$. The quadratic activation naturally brings $log_2$ form depth requirement.

### **3&nbsp;&nbsp;The Sign $(−1)^{∣T∣−∣R∣}$ in (3)**
The $(−1)^{∣T∣−∣R∣}$ factor in Eq. (3) is the inclusion–exclusion inversion coefficient on the Boolean lattice of subsets. Our vj(⋅) must invert the cumulative‑sum relation $u_j(X_S)=\sum_{T\subset S\setminus\{j\}} v_j(X_{T\cup\{j\}})$; effects contributed to many supersets are otherwise over‑counted. Alternating signs guarantee that terms appearing an even number of times cancel and those appearing an odd number of times net to the correct single contribution, yielding a unique decomposition. 

### **4&nbsp;&nbsp;Performance Difference between DeepHalo and TCnet**
We appreciate the reviewer’s thoughtful observation. In fact, we believe the comparable performance is both expected and reasonable, and it highlights an important distinction in modeling philosophy:

- As the reviewer rightly points out, the outputs of TCNet can also be interpreted through a utility decomposition perspective. However, **TCNet’s Transformer backbone lacks any mechanism to control the maximum interaction order**. The attention layers aggregate over the entire offer set, inherently blending information from all other items, which results in **unconstrained high-order interaction effects**. This makes the learned utilities difficult to interpret or isolate.

- In contrast, **DeepHalo provides explicit control over the interaction order** via its recursive architecture. For example, a depth-$l$ network captures only up to $l$-th order effects (see Appendix A.2), preserving identifiability and interpretability. The model structure enforces this decomposition, rather than approximating it implicitly.

- Given that low-order interactions often dominate in real-world choice tasks, it is **unsurprising that a sufficiently expressive Transformer (TCNet)** is able to match the predictive performance of DeepHalo. In fact, this reinforces the idea that **DeepHalo has captured the essential utility structure using fewer degrees of freedom**, while maintaining interpretability.

In summary, **our contribution is not about surpassing TCNet in raw accuracy**, but rather about offering a **principled tradeoff between expressiveness and interpretability**. DeepHalo is designed to act as a **bridge between highly expressive black-box models and simpler order-controllable models**, making it a valuable addition to the modeling toolkit for choice tasks with context effects.

### **4&nbsp;&nbsp;Data Efficiency**
| Ratio (%) | CMNL | MNL | FATE | MLP | MixedMNL | TCnet | **DeepHalo** |
|---:|---:|---:|---:|---:|---:|---:|---:|
|  10 | 1.574 | 1.898 | 1.583 | 1.571 | 1.674 | 1.589 | **1.566** |
|  20 | 1.567 | 1.869 | 1.577 | 1.563 | 1.617 | 1.573 | **1.558** |
|  30 | 1.564 | 1.844 | 1.577 | 1.557 | 1.590 | 1.569 | **1.551** |
|  40 | 1.562 | 1.820 | 1.577 | 1.554 | 1.576 | 1.568 | **1.548** |
|  50 | 1.561 | 1.779 | 1.577 | 1.550 | 1.566 | 1.570 | **1.544** |
|  60 | 1.562 | 1.760 | 1.577 | 1.545 | 1.563 | 1.551 | **1.537** |
|  70 | 1.555 | 1.742 | 1.577 | 1.542 | 1.560 | 1.539 | **1.532** |
|  80 | 1.544 | 1.726 | 1.577 | 1.537 | 1.558 | 1.560 | **1.528** |
|  90 | 1.550 | 1.699 | 1.577 | 1.539 | 1.555 | 1.549 | **1.526** |
| 100 | 1.550 | 1.726 | 1.577 | 1.537 | 1.558 | 1.538 | **1.528** |

To assess the data efficiency of DeepHalo, we conduct experiments on the SFOshop dataset, which exhibits strong context effects, by subsampling the training data at various proportions from 10% to 100%. All neural models are strictly constrained to a parameter budget of approximately 1k. Across all subsample ratios, DeepHalo consistently outperforms all other models in terms of Test NLL, demonstrating robust generalization even under limited data. When plotted, DeepHalo’s Test NLL exhibits the most pronounced and stable downward trend as the training data proportion increases—highlighting a clear advantage in data efficiency compared to other context-aware baselines such as TCnet, MLP, and FATE, which either plateau early or fluctuate as data increases.
Crucially, in these experiments DeepHalo uses a single layer with a quadratic activation, explicitly constraining the highest interaction order to 2. Despite competing models (TCnet, MLP, FATE) having uncontrolled higher-order expressivity, they do not generalize better under the same parameter budget. This indicates that controlling the maximum interaction order is beneficial in real applications with context effects: it reduces overfitting to spurious higher-order interactions, focuses capacity on the dominant (up to second-order) structure, and yields consistently superior NLL across all data regimes.

### **4&nbsp;&nbsp;Clarification**
We sincerely thank the reviewer for pointing out the following typos and notational inconsistencies:

- The use of $u_j(x_j, X_R)$ in Eq. (3) is indeed intended to denote the same function as $u_j(X_{R \cup \{j\}})$ in Eq. (2). We appreciate the suggestion and will revise the notation to maintain consistency throughout.

- The duplicate citation of “Deep Sets [Zaheer et al., 2018]” and the misspelling of FETA and FATE (where “E” should stand for “evaluate”) will be corrected in the final version.

- We acknowledge the inconsistent usage of $S$ for both the set and its cardinality. We will revise the manuscript to consistently use $|S|$ when referring to the size of the set, to avoid confusion.

Thank you again for your careful reading — these corrections will be implemented in the final revision.



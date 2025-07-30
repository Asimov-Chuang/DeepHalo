We sincerely appreciate the reviewer Uum8’s recognition of our modeling problem, 
methodological soundness, real-world validation, and thoughtful baseline selection. 

To address the reviewer’s concerns on novelty and interpretability, 
we provide detailed point-wise responses below.

### 1&nbsp;&nbsp;Beyond FETA and other context based choice models: controllable modeling of higher‑order effects  
Among context based models, 

First-order models like CMNL (Yousefi Maragheh et al., 2020), low-rank Halo MNL (Ko and Li, 2024), and FETA (Pfannschmidt et al., 2022) can only capture pairwise interactions. While FETA proposes a utility decomposition similar to the halo effect, it only evaluates first-order terms in practice, as enumerating higher-order subsets is computationally infeasible for large assortments—limiting its ability to model complex choices.

Other models such as FATE and TCNet attempt to learn context by directly entangling all item features. As the reviewer notes, their outputs can be interpreted via Eq. (10) as halo effects. However, **the interaction order in these models is uncontrolled**, and recovering a faithful decomposition would require evaluating up to $|S|-1$ orders—leading to exponentially many terms and breaking interpretability.

The issue lies in their architecture. FATE combines global context with individual features but cannot restrict interaction depth. TCNet’s attention mechanism inherently mixes all items, effectively modeling full-order interactions by design. As a result, these models **do not formulate context effects in a structured, decomposable way**, making interpretation difficult.


Therefore, first-order models have interpretability but lack of expressiveness, while other models have expressiveness but cannot control maximum effect order and is difficult to interpret. DeepHalo is designed to fill this gap, which manages to model higher Halo effects up to any order we want. How do we do?

- **Feature-Based High-Order Halo Effects**  
  To our knowledge, we are the first to formally define high-order halo effects in terms of item features. For example, when assessing how subset $\{A, B\}$ influences the utility of item $C$ in an offer set $\{A, B, C, D\}$, the effect should depend only on the features of $A$, $B$, and $C$, not on unrelated items like $D$. Including irrelevant items makes the subset effect uninterpretable. This illustrates why models that entangle all item features inevitably introduce uncontrolled, high-order interactions.

- **Structured Decomposition and Controllable Effect Order**  
  As shown in Appendix A, DeepHalo’s architecture ensures the utility can be decomposed as in Line 189. The maximum interaction order is determined by the model depth, while the choice of activation function affects how quickly the order increases. We appreciate the reviewer’s interest and will clarify this further in the revision.

- **Direct Effect Recovery Without Solving Linear Systems**  
  Prior approaches require solving large linear systems to recover halo effects, which becomes infeasible for large assortments. In contrast, DeepHalo leverages Eq. (3) and Eq. (10) to recover effects directly and efficiently. We will elaborate on this in response to the reviewer’s questions.

These innovations go beyond what DeepSets offers. While DeepSets provides permutation invariance, DeepHalo integrates domain-specific insights from discrete choice theory and the halo literature, along with architectural elements like residual connections and polynomial activations. This enables interpretable, high-order interaction modeling in a principled and controllable way.


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
We thank the reviewer for the insightful observation. The similar performance between DeepHalo and TCNet is expected and highlights a key difference in modeling philosophy.

While TCNet can approximate utility decompositions (maximum order), its Transformer architecture lacks control over the maximum interaction order. Attention layers aggregate across the full set, leading to high-order effects that are difficult to interpret. In contrast, DeepHalo explicitly controls interaction order through its recursive structure, enabling better interpretability and identifiability.

Given that low-order interactions often dominate in real-world settings, it is natural for TCNet to match DeepHalo's accuracy. However, DeepHalo achieves this with controlled expressiveness, offering a more interpretable alternative. Our goal is not to outperform black-box models in accuracy, but to provide a principled, order-aware design that balances predictive performance with transparency.

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

To evaluate data efficiency, we subsample the SFOshop dataset—known for strong context effects—at ratios from 10% to 100%, training all neural models under a strict ~1k parameter budget.

Across all settings, **DeepHalo consistently achieves the lowest Test NLL**, showing strong generalization even with limited data. Its Test NLL steadily improves with more data, while other models (TCNet, MLP, FATE) either plateau early or fluctuate.

Importantly, DeepHalo uses just a **single layer with quadratic activation**, explicitly limiting interaction order to 2. Competing models, though more expressive, do not generalize better under the same capacity. This suggests that **controlling interaction order improves generalization** by avoiding spurious high-order effects and focusing on dominant low-order structure.


### **5&nbsp;&nbsp;Clarification**
The expression $u_j(x_j, X_R)$ in Eq. (3) refers to the same function as $u_j(X_{R \cup \{j\}})$ in Eq. (2), and we will revise the notation for consistency. We will also correct the duplicate citation of Deep Sets [Zaheer et al., 2018], fix the misspelling of FETA and FATE (“E” stands for “evaluate”), and use $|S|$ consistently to denote set size. These issues will be addressed in the final version. We appreciate the reviewer’s careful reading.



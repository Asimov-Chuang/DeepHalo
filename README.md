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
  
- **Effect Recover Algorithm without Formulate Linear System** In previous literature, people need to formulate the whole linear system to get the halo effects, which is inefficient when the assortment size is big. With equation (10), we can skip this step. **We know the reviewer has some questions in this part, and we will elaborate them later.**

This is also why our model is not a simple imitation of DeepSets. DeepSets merely provides a foundational tool for implementing permutation invariance. Building upon this, we integrate insights from discrete choice theory and the halo effect literature, and further incorporate architectural innovations such as residual connections and polynomial activations. These design choices make it possible to learn high-order interaction effects in a structured and controllable manner.

### 2&nbsp;&nbsp;Utility Decomposition Elaboration
We thank the author for raising this point.  The key intuition is that **every recursion depth *l*** in Eqs. (4)–(5) adds **exactly one additional order of interaction** while preserving all lower‑order terms via the residual connection.  Formally:

1. **Pairwise (1‑st order) layer.**  
   The first layer aggregates the linear summaries $\bar Z^{1}$ from raw embeddings $z^{0}$ and modulates them with $z_{j}^{0}$. This yields  
   $z_{j}^{1}=z_{j}^{0} + \frac1H\sum_{h}\bar Z_{h}^{1} \varphi_{h}^{1}(z_{j}^{0}),$  
   which contains only **pairwise interactions** between the target alternative \(j\) and every other item \(k\) in the set.  (Derivation lines 90‑116 in the Paper)

2. **Second‑order layer.**  
   Consider a choice set $\{j,k,\ell\}$.  After the first layer, $z_{k}^{1}$ and $z_{\ell}^{1}$ already embed their pairwise effects with every other item.  
   The second layer builds  
   $\bar Z^{2}=\tfrac1S\sum_{m}W^{2}z_{m}^{1}, \qquad z_{j}^{2}=z_{j}^{1}+\tfrac1H\sum_{h} \bar Z_{h}^{2} \varphi_{h}^{2}(z_{j}^{0}),$  
   where $\bar Z^{2}$ is **linear in each $z^{1}_{m}$** and therefore bilinear in the original embeddings.  Multiplying $\bar Z_{h}^{2}$ with $\varphi_{h}^{2}(z_{j}^{0})$ produces terms proportional to  
   $\langle W^{2}W^{1}x_{k}, \varphi_{h}^{2}(x_{j})\rangle$ and $\langle W^{2}W^{1}x_{\ell}, \varphi_{h}^{2}(x_{j})\rangle$,  
   **jointly involving $\{j,k,\ell\}$**—exactly the second‑order contribution $v_{j}(\{k,\ell\})$ in the decomposition.  Higher‑order cross‑terms cannot arise until deeper recursions, so the separation is preserved.

### Two‑layer expansion on an offer set $\{a,b,c,d\}$  

Let the raw embeddings be $z_{i}^{0}=x_{i}$ for every item $i\!\in\!\{a,b,c,d\}$.

---

#### **Layer 1 — pairwise terms**

1. **Set summary**
   $$s^{1}= \tfrac14\!\sum_{k\in\{a,b,c,d\}} W^{1}x_{k}
          = \tfrac14\!\bigl(W^{1}x_{a}+W^{1}x_{b}+W^{1}x_{c}+W^{1}x_{d}\bigr).$$  

2. **Updated embedding for $a$**
   $$z_{a}^{1}=x_{a}
      +\tfrac1H\!\sum_{h} s_{h}^{1}\,\varphi_{h}^{1}(x_{a}).$$  

   Expanding $s_{h}^{1}$ shows that $z_{a}^{1}$ contains the three **pairwise interactions**  
   $$v_{a}^{1}(b)=\tfrac1H\!\sum_{h} \bigl(W^{1}x_{b}\bigr)_{h}\,\varphi_{h}^{1}(x_{a}),\qquad
     v_{a}^{1}(c),\qquad
     v_{a}^{1}(d).$$  

   Symmetrically, $z_{b}^{1},z_{c}^{1},z_{d}^{1}$ collect their own pairwise effects.  
   Hence layer 1 realizes the order‑1 component
   $$u_{a}^{(1)}(S\setminus\{a\})
     =v_{a}^{1}(b)+v_{a}^{1}(c)+v_{a}^{1}(d).$$  

---

#### **Layer 2 — second‑order terms**

1. **Set summary**
   $$s^{2}= \tfrac14\sum_{k} W^{2}z_{k}^{1}
          = \tfrac14\sum_{k} W^{2}\bigl(x_{k}+v_{k}^{1}(\cdot)\bigr).$$  

   The first part $W^{2}x_{k}$ is **linear**, reproducing lower‑order terms.  
   The second part $W^{2}v_{k}^{1}(\cdot)$ is **bilinear** in two distinct items.  
   Thus $s^{2}$ already mixes **pairs of other items.**

2. **Updated embedding for $a$**
   $$z_{a}^{2}=z_{a}^{1}
      +\tfrac1H\sum_{h} s_{h}^{2}\,\varphi_{h}^{2}(x_{a}).$$  

   Substituting $s^{2}$ yields six distinct **second‑order utilities**
   $$v_{a}^{2}(b,c)=\tfrac1H\sum_{h}\bigl\langle W^{2}v_{b}^{1}(c),e_{h}\bigr\rangle\varphi_{h}^{2}(x_{a}), v_{a}^{2}(b,d), v_{a}^{2}(c,d), \text{and their symmetric counterparts}.$$  

   Collecting them gives the order‑2 component
   $$u_{a}^{(2)}(S\setminus\{a\})
     =v_{a}^{2}(b,c)+v_{a}^{2}(b,d)+v_{a}^{2}(c,d).$$  

---

### **Utility decomposition achieved so far**

After two layers the utility of item $a$ decomposes as  
$$u_{a}(S)=u_{a}^{(0)} +u_{a}^{(1)}(S\setminus\{a\})+u_{a}^{(2)}(S\setminus\{a\})$$  

Hence **layer count = maximum interaction order represented**:  
- depth 1 ⇒ pairwise;  
- depth 2 ⇒ up to second‑order;  
and deeper recursions continue this pattern inductively.





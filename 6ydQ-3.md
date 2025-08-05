Thank you very much for your thoughtful comments. Please see our response as follows.

**Weakness & Questions:**
> **Q: The methodological ML contribution ... the contributions are not very novel.**

A: The contribution of DeepHalo lies not merely in modeling interactions or equivariance, but in introducing a **structured and order-controllable framework** that balances expressiveness and interpretability, which is lacking in existing context-dependent choice models.

While prior models have explored interaction effects, they face fundamental challenges:

- First-order models such as CMNL [Yousefi Maragheh et al., 2020], low-rank Halo MNL [Ko and Li, 2024], and FETA [Pfannschmidt et al., 2022] are **inherently restricted to pairwise interactions**. For instance, FETA proposes a decomposition that resembles the halo effect, but practically, it is limited to first-order pairwise interactions. The architecture is not scalable to higher-order effects, as doing so would require exponentially large tensor storage and computational resources, rendering it impractical for real-world applications.

- Other higher-order models, such as FATE [Pfannschmidt et al., 2022] and TCNet [Wang et al., 2023], attempt to capture context effects by **entangling all item features** in the offer set. While their outputs can be post hoc interpreted as halo effects via Eq. (10), the interaction order is uncontrolled, and recovering a meaningful decomposition requires $O(2^{|S|-1})$ complexity due to exponentially many terms. This stems from their architectures: context is not modeled in a structured or decomposable way. For instance, FATE compresses all item features into a set embedding, while TCNet’s softmax-based attention mixes information across items. We refer the reviewer to Appendix A2.2, A2.3, and Question 2 of our response to reviewer Unm8 for details on how DeepHalo addresses this limitation.

In contrast, DeepHalo is explicitly designed to address this gap by enabling **precise control over the maximum order of halo interactions**. In e-commerce applications like recommendation and bundle pricing, lower-order context effects are often more relevant, interpretable, and practically useful. While expressive models can recover such effects, they require up to $O(2^{|S|-1})$ terms to isolate. DeepHalo explicitly models limited-order halo effects, allowing users to control the maximum order $K$ with only $O(2^K)$ complexity.


> **Q: If the contribution relates to DCM behaviour modeling for the ML community, then some aspects should be considered... (Poor choice of (classical DCM) baselines)**

A: Since our model targets context effects, we primarily compare DeepHalo with context-dependent DCMs like Contextual MNL. MNL serves as a sanity check to highlight the role of context. To include context-independent RUM-based models, we add Mixed Logit in the featureless setting, as it can approximate any RUM. For ML baselines, we include TCNet and FATE (suggested by reviewer D2h9). We respectfully refer the reviewer to our detailed response to Reviewer D2h9.

> **Q: Lack of training (computational) performance statistics.**

A: Thanks for pointing this out. Below, we add the total training time (until early stop) in a sample run under the setting of the LPMC experiment in Section 5.2.
| Model Name | Total Training Time (s) |
|------------|-------------------------------|
| MNL        | 96.5                          |
| MLP        | 32.3                          |
| TasteNet   | 45.5                          |
| RUMNet     | 102.2                         |
| DLCL       | 298.1                         |
| ResLogit   | 143.3                         |
| FATE       | 61.3                          |
| TCnet      | 141.6                         |
| DeepHalo   | 202.0                         |

> **Q: However, it’s very unclear to me why this fits in the NeurIPS context.**

A: We believe this paper belongs to this community. Our work directly aligns with NeurIPS’s growing focus on **human-AI alignment** and **feedback modeling**. Understanding and mathematically formalizing human choice behavior is central to building AI systems that interact reliably and effectively with humans across various fields like recommender systems and large language models. Current alignment approaches (e.g., RLHF, LfD) implicitly assume humans are rational and feedback is unbiased, which are routinely violated in practice. By introducing a principled, interpretable framework for modeling context-dependent human choices, our paper enables more realistic models of human feedback, crucial for advancing ethical and user-centric AI.

Additionally, we list below several noteworthy works related to our topic from previous years’ NeurIPS:

[1]Oh S, Thekumparampil KK, Xu J. Collaboratively learning preferences from ordinal data. Advances in Neural Information Processing Systems. 28, 2015

[2]Stephen Ragain and Johan Ugander. Pairwise choice markov chains. Advances in Neural Information Processing Systems, 29, 2016

[3]Arjun Seshadri, Stephen Ragain, and Johan Ugander. Learning rich rankings. Advances in Neural Information Processing Systems, 33:9435–9446, 2020

[4]Yiqun Hu, David Simchi-Levi, and Zhenzhen Yan. Learning Mixed Multinomial Logits with Provable Guarantees. Advances in Neural Information Processing Systems, 9447-9459, 2022

[5]De Peuter S, Zhu S, Guo Y, Howes A, Kaski S. Preference learning of latent decision utilities with a human-like model of preferential choice. Advances in Neural Information Processing Systems, 37:123608-36, 2024


**Clarifications:**
> **Q: Some things are a bit unclear in intro.**

A: We will explain the ‘Featureless setting’ further by elaborating on ‘items without access to their features’.

> **Q: “For each j ∈S, define a utility function uj(T), ...”**

A: We will only introduce the utility function $u_j(S)$ instead, and only introduce the notation $T$, which emphasizes the context information (subset excluding the item j itself).

> **Q: “Here, $XT∪{j}∈R^{d_x ×J}$ denotes the matrix formed by replacing the feature vectors not in the subset T ∪{j}with zero.” —> rephrase.**

A: We will rewrite the definition of $X_{T∪{j}}$ as “formed from the feature matrix $X_S$ by replacing the feature vectors not in the subset T∪{j} with zero.”

> **Q: This sentence got me a bit confused: “Here, H denotes the number of interaction heads...”**

A: Analogous to multi-head attention, in our context, this can be interpreted as H different consumer types with different tastes. The h-th row of W^1 represents the h-th taste vector, which can map the input with various preferences. Thus, W^1 provides H variety of mappings, which are then aggregated by the sum over $Z_h^1$. This is indeed training each of the h-th mapping, and we use matrix W^1 to compact the notation. 

> **Q: “Residual Connection for Large Choice Sets” is quite confusing**

A: As the reviewer noted, one key computational advantage of the polynomial formulation is that a small number of layers can cover a large range of effect orders, avoiding the need for deep architectures (line 234). We elaborate on this in Section 4.2 (lines 262–264), showing a logarithmic relationship: a linear number of layers can express exponentially many interaction orders. We will clarify this in the revised version by referring explicitly to the quadratic example. See also Appendix A2.2, A2.3, and the example in our response to reviewer Unm8 (Section 2).

As for numerical stability, training remains stable in the featureless setting even with high-order interactions (Figure 2). In the feature-rich setting, deep quadratic stacks may cause instability, but we find that modeling up to 4th or 5th order suffices in practice (e.g., Expedia), striking a good balance between expressiveness and interpretability.

> **Q: For section 4.2., $e_S$ is a vector of 1s...**

A: Regarding Section 4.2: $e_S$ is a unit vector with the $(e_S)_j = 1$ if j ∈ S and 0 otherwise. The recursive expression in equation 9 can be represented as a polynomial of $e_S$. Let us consider the case with up to second-order interaction, then the second-order term relates to $e_S \otimes e_S$ where the (i,j)-th element refers to whether both item i and item j exist in the choice set. The coefficients in the polynomial corresponding to each element of the second-order term measure the specific interactions in equation (1).

> **Q: confusing with the meaning of $\cup$ in equation 3.**

A: We apologize for the typo in equation 3: $X_R$ should be $X_{R∪{j}}$, which hopefully entangles the confusion. In equation (10), the relative context effect is specified on item j and item k, whose definition is introduced in Park and Hahn [1998].

> **Q: About number of parameters in Fig 2...**

A: The number of nn parameters in Fig 2 is the total number of trainable parameters, which specifies the model complexity and is influenced by the problem size and the selection of hyperparameters like H.

> **Q: $\bar{Z}_1 := 1/S$…**

A: Yes, thanks for catching that. We will correct to |S|.


**Beverage Experiment**

Thank you for the thoughtful comment.

**This classical hypothetical dataset has been widely used in the literature on halo effects** (e.g., Park and Hahn [1998], Batsell and Polking [1985]) for its simplicity and clarity, making it a standard example for evaluating interpretability. The experiment is conducted in a featureless setting, demonstrating that DeepHalo can extract meaningful interaction effects purely from set-level preferences.

Subfigure (b) illustrates how different item subsets (horizontal axis) influence the utility gap between item pairs (vertical axis). The rightmost column (∅) shows baseline preferences without contextual influence—for example, strong preferences for Pepsi over Coke and 7-Up over Sprite are evident both here and in the market shares (Subfigure a). The leftmost column (item 1) reveals a strong negative halo effect of Pepsi on Coke, reducing Coke’s competitiveness when co-offered. A similar pattern is observed between 7-Up and Sprite.



Following your suggestion, we tested the nested logit model on the SFOshop and SFOwork datasets. As is well-known, specifying the nesting structure is a often a pratically challenging task. Fortunatelly, for this dataset, we are able to follow the guidelines provided in Koppelman and Bhat (2006). The results are presented below.

We observe that the nested logit model is consistently outperformed by all context-dependent choice models, including our proposed DeepHalo model. Note also that its performance falls between that of MNL and mixed logit (not RUMNet but the original one), which is consistent with our intuition. This highlights the importance of capturing context effects when modeling the complex choice behavior observed in real-world datasets.

Additionally, we plan to include a paragraph in the introduction to discuss the differences between traditional IIA-relaxing models and our approach. We hope this will help provide readers with a clearer and more intuitive understanding of our contribution.

| Model             | SFOshop (Train/Test) | SFOwork (Train/Test) |
|-------------------|---------------------|---------------------|
| MNL               | 1.7281 / 1.7262     | 0.9423 / 0.9482     |
| MLP               | 1.5556 / 1.5523     | 0.8074 / 0.8120     |
| CMNL              | 1.5676 / 1.5686     | 0.8116 / 0.8164     |
| Mixed MNL         | 1.5599 / 1.5577     | 0.8092 / 0.8153     |
| Nested Logit      | 1.5788 / 1.5875     | 0.8366 / 0.8457     |
| FateNet           | 1.5726 / 1.5765     | 0.8133 / 0.8167     |
| TCNet             | 1.5694 / 1.5742     | 0.8114 / 0.8145     |
| DeepHalo (Ours)   | 1.5385 / 1.5263     | 0.8040 / 0.8066     |

Reference:  
Koppelman, F. S., & Bhat, C. (2006). *A self-instructing course in mode choice modeling: Multinomial and nested logit models.*



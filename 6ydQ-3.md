Thank you for your thoughtful comments. Please see our response as follows.

**Weakness & Questions:**
Regarding the choice of baselines for the experiment: If we understand your point correctly, you refer to the experiment in Section 5.1, Table 1. This experiment focuses on measuring the context effect in the featureless setting. We compare with the context-dependent DCM benchmark, Contextual MNL. To cover the context-independent DCM benchmarks you mentioned, which follow the RUM principle, we add mixed logit, which can approximate any RUM model arbitrarily well. The baseline MNL, usually included in the literature, functions merely as a sanity check. Regarding ML baselines, we add additional baselines with transformer (TCnet) and deep sets (FATE), as displayed below.

| Model         | Hotel (Train/Test) | SFOshop (Train/Test) | SFOwork (Train/Test) |
|---------------|--------------------|------------------------|-----------------------|
| MNL           | 0.7743 / 0.7743    | 1.7281 / 1.7262        | 0.9423 / 0.9482       |
| MLP           | 0.7569 / 0.7523    | 1.5556 / 1.5523        | 0.8074 / 0.8120       |
| CMNL          | 0.7566 / 0.7561    | 1.5676 / 1.5686        | 0.8116 / 0.8164       |
| Mixed MNL     | 0.7635 / 0.7613    | 1.5599 / 1.5577        | 0.8092 / 0.8153       |
| FateNet       | 0.7575 / 0.7568    | 1.5726 / 1.5765        | 0.8133 / 0.8167       |
| TCNet         | **0.7467** / 0.7670| 1.5694 / 1.5742        | 0.8114 / 0.8145       |
| **DeepHalo** (Ours) | 0.7479 / **0.7483** | **1.5385** / **1.5263** | **0.8040** / **0.8066** |


- Regarding the computational performance statistics: The training speed of the ML baselines and our model does not exhibit significant gaps, thus we do not include them in the statistics. On the other hand, our model enjoys more computational capability in recovering the interaction coefficients, because we don’t need to formulate the whole linear halo system. We can recover the effect iteratively up to any order effect we want.
- Fitting in Neurips venue: We believe this paper belongs to this community, since several wonderful works related to our topic have appeared in previous years’ Neurips; some are listed below:

[1]Oh S, Thekumparampil KK, Xu J. Collaboratively learning preferences from ordinal data. Advances in Neural Information Processing Systems. 28, 2015

[2]Stephen Ragain and Johan Ugander. Pairwise choice markov chains. Advances in Neural Information Processing Systems, 29, 2016

[3]Arjun Seshadri, Stephen Ragain, and Johan Ugander. Learning rich rankings. Advances in Neural Information Processing Systems, 33:9435–9446, 2020

[4]Yiqun Hu, David Simchi-Levi, and Zhenzhen Yan. Learning Mixed Multinomial Logits with Provable Guarantees. Advances in Neural Information Processing Systems, 9447-9459, 2022

[5]De Peuter S, Zhu S, Guo Y, Howes A, Kaski S. Preference learning of latent decision utilities with a human-like model of preferential choice. Advances in Neural Information Processing Systems, 37:123608-36, 2024


**Clarifications:**
- We will explain the ‘Featureless setting’ further by elaborating on ‘items without access to their features’.
- We hope to clarify here that the mentioned definition regarding u_j(T) is regarding the universal logit model, which generalizes the MNL model by allowing context-dependent utilities. MNL, as a special case of the universal logit model, is less expressive as it assumes IIA and the utility u_j is only related to item j, not the whole choice set. Regarding the confusing notation, we will only introduce the utility function u_j(S) instead, and only introduce the notation $T$ which emphasizes the context information (subset excluding the item j itself).
- Thanks for pointing that out. We will rewrite the definition of X_{T∪{j}} as “formed from the feature matrix X_S by replacing the feature vectors not in the subset T∪{j}with zero.”
- Each row of W^1 represents a specific mapping at index h, so W^1 provides H variety of mappings, which are then aggregated by the sum over Z_h^1. This is indeed training each of the h-th mapping, and the notion W^1 provides a clean representation of the H-head linear mapping. 
- Regarding Section 4.1: One of the key computational benefits of the polynomial is exactly as you suggested – a few layers can include a large universe, which prevents the very deep structure (line 234). We further discuss the quadratic benefit in Section 4.2 (line 262-264) and provide a logarithmic relation, namely, a linear number of layers can cover exponentially many items. To enrich section 4.1, we will particularly refer to the quadratic example in the revised version.
- Regarding Section 4.2: e_S is a unit vector with the (e_S)_j = 1 if j ∈ S and 0 otherwise. The recursive expression in equation 9 can be represented as a polynomial of e_S. Let us consider the case with up to second-order interaction, then the second-order term relates to $e_S \otimes e_S$ where the (i,j)-th element refers to whether both item i and item j exist in the choice set. The coefficients in the polynomial corresponding to each element of the second-order term measure the specific interactions in equation (1).
- Regarding \union in Equation 3: If I understand your question, I think your confusion might have originated from a typo in equation 3: X_R should be X_{R∪{j}}. In equation (10), the relative context effect is specified on item j and item k, whose definition is introduced in Park and Hahn [1998].
- The number of nn parameters in Fig 2 is the total number of trainable parameters, which specifies the model complexity and is influenced by the problem size and the selection of hyperparameters like H.
- Yes, thanks for catching that. We will correct to |S|.


**Beverage Experiment**
Thank you for the thoughtful comment. We encourage the reviewer to examine Figure 1 to better understand how DeepHalo captures interpretable and structured halo effects.
Panel (b) visualizes the relative halo effect exerted by different subsets of items (horizontal axis) on the utility gap between competing item pairs (vertical axis). The rightmost column (∅) represents the baseline utility differences between item pairs in the absence of any contextual influence. For example, the strong baseline signal in pairs (1,2) and (3,4) corresponds to stable preferences such as Pepsi over Coke and 7-Up over Sprite, which are also clearly reflected in the observed market shares (panel a).
The leftmost column (item 1) shows a strong negative halo effect from Pepsi onto Coke: the presence of item 1 (Pepsi) reduces item 2’s (Coke’s) competitiveness when it is competing with item 3 or 4. Similarly, item 3 (7-Up) exerts a negative contextual effect on item 4 (Sprite) when competing with other items, highlighting how DeepHalo uncovers asymmetric substitution effects across the set.
Crucially, this experiment is conducted in a featureless setting, where no item features are used, demonstrating that DeepHalo can extract meaningful interaction effects purely from set-level preferences. We also wish to note that this hypothetical dataset has been widely used in the halo effect literature due to its simplicity and clarity, making it a standard example for validating interpretability.


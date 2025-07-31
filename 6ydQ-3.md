Thank you very much for your thoughtful comments. Please see our response as follows.

**Weakness & Questions:**
> Q: The methodological ML contribution lies practically exclusively in the modeling of interactions and equivariance, and in both cases, the contributions are not very novel.

A: We appreciate the reviewer’s comment and respectfully clarify that the key methodological contribution of DeepHalo lies not merely in modeling interactions or equivariance per se, but in introducing a structured and order-controllable framework that balances expressiveness and interpretability, which is lacking in existing context-dependent choice models.

While prior models have explored interaction effects, they face fundamental challenges:

- First-order models such as CMNL [Yousefi Maragheh et al., 2020], low-rank Halo MNL [Ko and Li, 2024], and FETA [Pfannschmidt et al., 2022] are inherently restricted to pairwise interactions. For instance, FETA proposes a decomposition that resembles the halo effect, but practically, it is limited to first-order pairwise interactions. The architecture is not scalable to higher-order effects, as doing so would require exponentially large tensor storage and computational resources, rendering it impractical for real-world applications.

- Other higher-order models, such as FATE [Pfannschmidt et al., 2022], ResLogit [Wong and Farooq, 2021], and TCNet [Wang et al., 2023], attempt to learn context effects by directly entangling all item features within the offer set. While their outputs can be retrospectively interpreted as halo effects via Eq. (10), the interaction order is not explicitly controlled. Recovering a meaningful decomposition in these models would require computational complexity on the order of $O(2^{∣S∣−1})$, due to the exponential number of possible interaction terms. This limitation stems from their architectural design: context effects are not modeled in a structured or decomposable way. Specifically, FATE aggregates all item features into a set embedding, which is then combined with individual item features through complex nonlinear transformations, obscuring the origin and order of interactions. TCNet uses attention mechanisms where the softmax denominator implicitly mixes information from all items in the offer set, making it difficult to isolate individual effects. Similar entanglement of all items applies to ResLogit. As a result, none of them can recover a truly decomposable utility function. We refer the reviewer to Appendix A2.2 and A2.3, as well as the illustrative example in Question 2 of our response to reviewer Unm8, for a detailed explanation of how DeepHalo supports structured, decomposable modeling of context effects.

In contrast, DeepHalo is explicitly designed to address this gap by enabling precise control over the maximum order of halo interactions. To the best of our knowledge, it is the first model that supports flexible trade-offs between model complexity and interpretability in this context.

In e-commerce applications such as recommendation systems and bundle pricing, practitioners are often primarily interested in lower-order context effects, which are more stable, interpretable, and practically useful. While expressive models can theoretically recover such effects, they do so implicitly as part of an entangled representation involving many higher-order interactions. Consequently, low-order effects cannot be meaningfully isolated and require evaluating up to $O(2^{|S|-1})$ terms.

In contrast, DeepHalo offers a principled approximation that explicitly models low-order halo effects. It allows users to control the maximum interaction order $K$, with computational complexity only $O(2^K)$, offering a flexible trade-off between efficiency and expressiveness.



> Q: If the contribution relates to DCM behaviour modeling for the ML community, then some aspects should be considered... (Poor choice of (classical DCM) baselines)

A: Regarding the choice of baselines: Since our model focuses on context effects, we primarily compare DeepHalo with context-dependent DCMs such as Contextual MNL. MNL is included as a sanity check, and the comparison between MNL and Contextual MNL illustrates the impact of context. To cover the context-independent RUM-based models you mentioned, we include Mixed Logit in the featureless setting, as it can approximate any RUM arbitrarily well. For ML baselines, we incorporate TCNet (Transformer-based) and FATE (DeepSets-based), as suggested by reviewer D2h9, as shown below.

| Model         | Hotel (Train/Test) | SFOshop (Train/Test) | SFOwork (Train/Test) |
|---------------|--------------------|------------------------|-----------------------|
| MNL           | 0.7743 / 0.7743    | 1.7281 / 1.7262        | 0.9423 / 0.9482       |
| MLP           | 0.7569 / 0.7523    | 1.5556 / 1.5523        | 0.8074 / 0.8120       |
| CMNL          | 0.7566 / 0.7561    | 1.5676 / 1.5686        | 0.8116 / 0.8164       |
| Mixed Logit     | 0.7635 / 0.7613    | 1.5599 / 1.5577        | 0.8092 / 0.8153       |
| FateNet       | 0.7575 / 0.7568    | 1.5726 / 1.5765        | 0.8133 / 0.8167       |
| TCNet         | **0.7467** / 0.7670| 1.5694 / 1.5742        | 0.8114 / 0.8145       |
| **DeepHalo** (Ours) | 0.7479 / **0.7483** | **1.5385** / **1.5263** | **0.8040** / **0.8066** |

> Q: Lack of training (computational) performance statistics. How fast/slow is it, compared with the other DNNs?

A: Thanks for pointing this out. Below, we add the total training time (until early stop) in a sample run under the setting of the LPMC experiment in Section 5.2.
| Model Name | Total Training Time (seconds) |
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

Our model exhibits moderate training time relative to other DNNs, which can be further reduced through hyperparameter tuning (e.g., early stopping criteria, batch size). Notably, all DNN models complete training on the dataset within 5 minutes, demonstrating efficient and scalable training. This makes rendering training speed a less critical concern in this context. More importantly, our model offers enhanced computational capability in recovering interaction coefficients, as discussed in our response to your first question.

> Q: However, it’s very unclear to me why this fits in the NeurIPS context.

A: We believe this paper belongs to this community since several wonderful works related to our topic have appeared in previous years’ NeurIPS and other ML conferences; some are listed below:

[1]Oh S, Thekumparampil KK, Xu J. Collaboratively learning preferences from ordinal data. Advances in Neural Information Processing Systems. 28, 2015

[2]Stephen Ragain and Johan Ugander. Pairwise choice markov chains. Advances in Neural Information Processing Systems, 29, 2016

[3]Arjun Seshadri, Stephen Ragain, and Johan Ugander. Learning rich rankings. Advances in Neural Information Processing Systems, 33:9435–9446, 2020

[4]Yiqun Hu, David Simchi-Levi, and Zhenzhen Yan. Learning Mixed Multinomial Logits with Provable Guarantees. Advances in Neural Information Processing Systems, 9447-9459, 2022

[5]De Peuter S, Zhu S, Guo Y, Howes A, Kaski S. Preference learning of latent decision utilities with a human-like model of preferential choice. Advances in Neural Information Processing Systems, 37:123608-36, 2024

[6] Arjun Seshadri, Alex Peysakhovich, and Johan Ugander. Discovering context effects from raw choice data. In International conference on machine learning, pages 5660–5669. PMLR, 2019

[7] Amanda Bower and Laura Balzano. Preference modeling with context-dependent salient features. In International Conference on Machine Learning, pages 1067–1077. PMLR, 2020.

[8] Nir Rosenfeld, Kojin Oshiba, and Yaron Singer. Predicting choice with set-dependent aggregation. In International Conference on Machine Learning, pages 8220–8229. PMLR, 2020.

[9] Kiran Tomlinson and Austin R Benson. Learning interpretable feature context effects in discrete choice. In Proceedings of the 27th ACM SIGKDD conference on knowledge discovery & data mining, pages 1582–1592, 2021

Our primary area is machine learning for sciences (e.g., climate, health, life sciences, physics, social sciences). Transparent understanding of human decision-making processes is very important in the social sciences and medical fields. In addition, establishing more reasonable preference models is also an important part of commonly used algorithms such as RLHF. Therefore, we believe that our article is very suitable for this primary area.

**Clarifications:**
> Q: The intro is good but some things are a bit unclear.

A: We will explain the ‘Featureless setting’ further by elaborating on ‘items without access to their features’.

> Q: “For each j ∈S, define a utility function uj(T), which represents the utility of alternative j when the context is the subset T ⊆S{j}.” This is quite confusing.

A: We will only introduce the utility function $u_j(S)$ instead, and only introduce the notation $T$, which emphasizes the context information (subset excluding the item j itself).

> Q: “Here, $XT∪{j}∈R^{d_x ×J}$ denotes the matrix formed by replacing the feature vectors not in the subset T ∪{j}with zero.” —> rephrase. 


A: Thanks for pointing that out. We will rewrite the definition of $X_{T∪{j}}$ as “formed from the feature matrix $X_S$ by replacing the feature vectors not in the subset T∪{j} with zero.”

> Q: This sentence got me a bit confused: “Here, H denotes the number of interaction heads...”

A: Analogous to multi-head attention, in our context, this can be interpreted as H different consumer types with different tastes. The h-th row of W^1 represents the h-th taste vector, which can map the input with various preferences. Thus, W^1 provides H variety of mappings, which are then aggregated by the sum over $Z_h^1$. This is indeed training each of the h-th mapping, and we use matrix W^1 to compact the notation. 

> Q: Section “4.1 Residual Connection for Large Choice Sets” is quite confusing

A: Regarding Section 4.1: One of the key computational benefits of the polynomial is exactly as you suggested – a few layers can include a large universe, which prevents the very deep structure (line 234). We further discuss the quadratic benefit in Section 4.2 (line 262-264) and provide a logarithmic relation, namely, a linear number of layers can cover exponentially many effect orders. To enrich section 4.1, we will particularly refer to the quadratic example in the revised version. We refer the reviewer to Appendix A 2.2 and A 2.3 and the example (Section 2) we give in our response to reviewer Unm8. Regarding potential numerical issues, we note empirically that training remains stable in the featureless setting, even with very high-order interactions (see Figure 2). In the feature-rich setting, stacking too many quadratic layers may lead to instability during training. However, it is important to emphasize that achieving very high orders is not necessary in practice. In experiments such as Expedia, we find that modeling effects up to the 4th or 5th order already provides sufficient expressiveness. This strikes a desirable balance—yielding interpretable models with a manageable number of effects, while maintaining strong performance.

> Q: For section 4.2., if I understand well, $e_S$ is a vector of 1s...

A: Regarding Section 4.2: $e_S$ is a unit vector with the $(e_S)_j = 1$ if j ∈ S and 0 otherwise. The recursive expression in equation 9 can be represented as a polynomial of $e_S$. Let us consider the case with up to second-order interaction, then the second-order term relates to $e_S \otimes e_S$ where the (i,j)-th element refers to whether both item i and item j exist in the choice set. The coefficients in the polynomial corresponding to each element of the second-order term measure the specific interactions in equation (1).

> Q: There’s something confusing with the meaning of $\cup$ in equation 3.

A: We apologize for the typo in equation 3: $X_R$ should be $X_{R∪{j}}$, which hopefully entangles the confusion. In equation (10), the relative context effect is specified on item j and item k, whose definition is introduced in Park and Hahn [1998].

> Q: About number of parameters in Fig 2...

A: The number of nn parameters in Fig 2 is the total number of trainable parameters, which specifies the model complexity and is influenced by the problem size and the selection of hyperparameters like H.

> Q: $\bar{Z}_1 := 1/S$…

A: Yes, thanks for catching that. We will correct to |S|.


**Beverage Experiment**

Thank you for the thoughtful comment. 

We wish to note that this classical hypothetical dataset has been widely used in literature from the definition of Halo effect is proposed (Park and Hahn [1998], Batsell and Polking, [1985]) due to its simplicity and clarity, making it a standard example for validating interpretability. Indeed, this experiment is conducted in a featureless setting, where no item features are used, demonstrating that DeepHalo can extract meaningful interaction effects purely from set-level preferences. 

Here we provide a more detailed explanation of the contents of the figure. Subfigure (b) visualizes the relative halo effect exerted by different subsets of items (horizontal axis) on the utility gap between competing item pairs (vertical axis). The rightmost column (∅) represents the baseline utility differences between item pairs in the absence of any contextual influence. For example, the strong baseline signal in pairs (1,2) and (3,4) corresponds to stable preferences such as Pepsi over Coke and 7-Up over Sprite, which are also clearly reflected in the observed market shares (Subfigure a). The leftmost column (item 1) shows a strong negative halo effect from Pepsi onto Coke: the presence of item 1 (Pepsi) reduces item 2’s (Coke’s) competitiveness when it is competing with item 3 or 4. Similarly, item 3 (7-Up) exerts a negative contextual effect on item 4 (Sprite) when competing with other items.



**Final Remarks**

We greatly appreciate the time, effort, and constructive feedback from all reviewers and the area chair. The discussions have been highly insightful, substantially improving both the clarity and technical depth of our work. Here, we provide a concise summary of our main contributions and the key outcomes from the discussion phase.

**Contributions**  
1. Proposed an order-controllable context-dependent choice model that balances interpretability and predictive performance.  
2. Extended the Halo effect framework to settings with item features.  
3. Developed an identifiable effect recovery algorithm to extract meaningful context effects.  

**Outcomes from the Discussion**  

**1. Addressed the primary novelty concern** 

Reviewers’ novelty concerns focused on three questions:  
(i) how DeepHalo controls the order of context effects,  
(ii) why such control is necessary, and  
(iii) why existing models cannot achieve it.  

We addressed (i) with a simplified example in our discussion with Uum8 and by referring to the derivation in the Appendix. For (ii) and (iii), we conducted extensive experiments on synthetic and real datasets, whose results were well recognized by reviewers Uum8 and B9Eq for their clarity and persuasiveness. (i) is also further validated through the experiments. These experiments showed that: 

- Lower- and higher-order effects work in tandem. Leaving the maximum order uncontrolled leads to exponential growth in effects and loss of interpretability.  
- Naive truncation to obtain lower-order effects can severely distort predicted choice probabilities.  
- Existing models, by entangling all item features, can only recover a full-order model, whereas our method enables faithful lower-order approximations via explicit order constraints.  


**2. More baseline models, training details, and additional experiments**  

We further improved the empirical evaluation by:  
- Incorporating reviewer 6ydQ’s suggestion to include classical non-IIA models (mixed logit, nested logit) and report training time.  
- Following reviewer D2h9’s advice to provide parameter counts and add baselines in featureless experiments.  
- Responding to Uum8’s request by adding a data efficiency experiment.  

Feedback from reviewers indicates these concerns have been satisfactorily addressed.  

**3. Clarifications on presentation-related points**  

Reviewers raised several minor points regarding presentation and confusing points. We have addressed these by:  
- Providing point-by-point clarifications during the discussion.  
- Outlining specific revision plans to improve clarity and readability.  

We aim to make the paper as straightforward and easy to follow as possible. 

**Summary**  

Across the discussion phase, all reviewers reached consensus in giving positive evaluations of our work. The feedback was constructive and converged on the view that the paper makes a clear and valuable contribution. Nearly all concerns raised were fully resolved through additional experiments, clarifications, and revisions. 


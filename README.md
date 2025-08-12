**Final Remarks**

We greatly appreciate the time, effort, and constructive feedback from all reviewers and the area chair. The discussions have been highly insightful, substantially improving both the clarity and technical depth of our work. Here, we provide a concise summary of our main contributions and the key outcomes from the discussion phase.

**Contributions**  
1. Proposed an order-controllable context-dependent choice model that balances interpretability and predictive performance.  
2. Extended the Halo effect framework to settings with item features.  
3. Developed an identifiable effect recovery algorithm to extract meaningful context effects.  

**Outcomes from the Discussion**  
1. Addressed the primary novelty concern — why controlling the order of context effects is necessary and why existing models cannot achieve this.  
   - Demonstrated (see discussions with reviewers Uum8 and B9Eq) that lower-order effects work in tandem with higher-order effects, and that leaving the maximum order uncontrolled results in an exponential number of effects, severely compromising interpretability.  
   - Showed that simply truncating to obtain lower-order effects can greatly distort predicted choice probabilities.  
   - Clarified that existing models, due to entangling all item features, can only recover the full-order model, whereas our approach enables faithful lower-order approximations by explicitly constraining the maximum order.  
2. Incorporated reviewer 6ydQ’s suggestion to include classical non-IIA discrete choice models (mixed logit, nested logit) for broader comparisons.  
3. Followed reviewer D2h9’s advice to report parameter counts and include additional baselines in the featureless experiments, making the empirical evaluation more convincing.

In addition, we have provided clarifications for other points raised and will incorporate corresponding improvements in future versions. We look forward to continued constructive exchanges with the community.

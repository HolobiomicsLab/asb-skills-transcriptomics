---
name: presence-absence-matrix-construction
description: Use when comparing which of the top-ranked ligand-receptor predictions
  are shared across the 7 CCC methods or 16 prior-knowledge resources, i.e., when
  the analysis goal is overlap of predicted interaction sets rather than score correlation.
when_to_use_negative:
- When agreement in score magnitude or ranking order matters (e.g., correlating LRscore
  or probability values) — this skill discards magnitudes and keeps only set membership
- When predictions have not been reduced to a common ranked shortlist per method-resource
  combination (e.g., all scored interactions rather than top 1000), since Jaccard
  on untrimmed, differently sized sets is not comparable
- When comparing methods that infer intracellular processes (NicheNet, Cytotalk, SoptSC),
  which the article states are not directly comparable to the 7 ligand-receptor methods
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LIANA
  role: Wrapper that decouples each CCC method from its default resource and produces
    ranked interaction scores per method-resource-dataset combination, providing the
    ranked lists reduced to top 1000 for the presence-absence matrices
  repo: https://github.com/saezlab/liana
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:06032edc3163209f23abc9a9d82fd73f855297fb753118d48f351cc79c3cda6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - Connectome
  - Cytotalk
  - LIANA
  - NATMI
  - SingleCellSignalR
  - iTALK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# presence-absence-matrix-construction

## Summary

Converts top-ranked cell-cell communication (CCC) interaction predictions (e.g., the 1000 highest-ranked interactions per method-resource-dataset combination) into binary presence-absence matrices so that set-level agreement between methods or prior-knowledge resources can be quantified with pairwise Jaccard indices.

## When to use

Use when comparing which of the top-ranked ligand-receptor predictions are shared across the 7 CCC methods or 16 prior-knowledge resources, i.e., when the analysis goal is overlap of predicted interaction sets rather than score correlation.

## When NOT to use

- When agreement in score magnitude or ranking order matters (e.g., correlating LRscore or probability values) — this skill discards magnitudes and keeps only set membership
- When predictions have not been reduced to a common ranked shortlist per method-resource combination (e.g., all scored interactions rather than top 1000), since Jaccard on untrimmed, differently sized sets is not comparable
- When comparing methods that infer intracellular processes (NicheNet, Cytotalk, SoptSC), which the article states are not directly comparable to the 7 ligand-receptor methods

## Inputs

- Ranked interaction lists (LIANA magnitude-rank scores) for each method-resource-dataset combination
- Top-1000 highest-ranked interactions per method-resource-dataset combination
- Six preprocessed scRNA-seq datasets (three breast cancer subtypes, cord blood mononuclear cells, pancreatic islets, colorectal cancer)

## Outputs

- Presence-absence matrix (binary entries per top-1000 interaction set per method and per resource)
- Pairwise Jaccard similarity matrices for method-method and resource-resource comparisons
- median_jaccard_fig5.csv with median Jaccard values across datasets (~0.080 methods, ~0.119 resources)
- Fig 5-style similarity heatmap

## How to apply

Run each of the 7 CCC methods (CellChat probability, CellPhoneDBv2 truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, iTALK-inspired logFC Mean, Cytotalk-inspired Crosstalk score) decoupled from its default resource against each of the 16 resources on each scRNA-seq dataset via LIANA. Rank interactions by each method's magnitude rank and retain the top 1000 per method-resource-dataset combination. Build a presence-absence matrix whose rows are interactions and whose binary entries indicate whether an interaction falls within a given top-1000 set. From these binary vectors compute all pairwise Jaccard indices, then take the median across the six datasets separately for method-method and resource-resource comparisons; expected medians are ~0.080 (different methods) and ~0.119 (different resources), written to median_jaccard_fig5.csv and rendered as a Fig 5-style similarity heatmap.

## Related tools

- **LIANA** (Wrapper that decouples each CCC method from its default resource and produces ranked interaction scores per method-resource-dataset combination, providing the ranked lists reduced to top 1000 for the presence-absence matrices) — https://github.com/saezlab/liana

## Examples

```
from sklearn.metrics import pairwise_distances; J = 1 - pairwise_distances(pa_matrix.T, metric='jaccard')  # pa_matrix: interactions x method/resource presence-absence matrix
```

## Evaluation signals

- Median pairwise Jaccard across the six datasets falls near the reported values: ~0.080 for different methods and ~0.119 for different resources
- Jaccard matrix is symmetric with diagonal equal to 1 and all off-diagonal values in [0, 1]
- Each presence-absence column (or row) for a top-1000 set contains exactly 1000 positive entries
- Output CSV median_jaccard_fig5.csv contains exactly two summary values (method-method median and resource-resource median) computed as medians over the six datasets
- Heatmap structure matches Fig 5: separate method-method and resource-resource similarity blocks

## Limitations

- Low overlap does not identify a best method: no gold standard exists and the biological ground truth for CCC is largely unknown
- Results depend on the top-1000 cutoff; overlap statistics reflect set membership of that threshold only, not agreement in score magnitudes
- Consistently low median Jaccard (~0.08–0.12) means method/resource choice strongly influences predictions, so downstream biological interpretation is sensitive to these choices

## Evidence

- [results] The top-ranked predicted interactions show consistently low overlap when using either different methods or different resources, with median pairwise Jaccard indices of 0.080 (different methods) and 0.119 (different resources).: [evidence span withheld — anchor status: not_found]
- [abstract] we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions: "we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions"
- [intro] Then, we analysed how different combinations of resources and methods influence CCC inference by decoupling the methods from their corresponding resources and applying all method-resource: "Then, we analysed how different combinations of resources and methods influence CCC inference by decoupling the methods from their corresponding resources and applying all method-resource"
- [readme] LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus.: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."

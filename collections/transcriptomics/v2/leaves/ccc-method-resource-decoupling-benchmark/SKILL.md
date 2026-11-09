---
name: ccc-method-resource-decoupling-benchmark
description: Use when assessing how much the choice of CCC inference method versus
  prior-knowledge resource drives disagreement in ligand-receptor predictions from
  single-cell RNA-seq data.
when_to_use_negative:
- The goal is to determine which CCC method is best — no gold standard exists, so
  the benchmark cannot identify a definitive best method
- The methods of interest additionally infer intracellular processes (e.g. NicheNet,
  Cytotalk, SoptSC), which are not directly comparable in this magnitude-rank decoupling
  design
- Long-distance endocrine signalling is the question of interest — scRNA-seq gene
  expression does not capture it
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: LIANA
  role: Framework used to decouple methods from resources, run all method-resource
    combinations, and compute consensus/magnitude ranks
  repo: https://github.com/saezlab/liana
- name: CellChat
  role: Source of the probability-based CCC method and CellChatDB resource being benchmarked
- name: CellPhoneDB
  role: Source of the truncated-mean CCC method and its default resource being benchmarked
- name: Connectome
  role: Source of the weight_norm CCC method being benchmarked
- name: NATMI
  role: Source of the mean-expression edge-weight CCC method and ConnectomeDB resource
    being benchmarked
  repo: https://github.com/asrhou/NATMI
- name: SingleCellSignalR
  role: Source of the LRscore method and LRdb resource; a custom version allows custom
    ligand-receptor resources
  repo: https://github.com/SCA-IRCM/SingleCellSignalR_v1
- name: OmnipathR
  role: Client providing access to OmniPath ligand-receptor resources used in the
    resource comparison
  repo: https://github.com/saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:72bcdd6a1ee82944c76445f097f7da3b0243c2cf77daacb34c54e7b4bac7dd0a
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
  - OmnipathR
  - SingleCellSignalR
  - iTALK
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# ccc-method-resource-decoupling-benchmark

## Summary

A benchmarking skill that decouples 7 cell-cell communication (CCC) inference methods from their default ligand-receptor resources, runs all 7x16 method-resource combinations on multiple scRNA-seq datasets via LIANA, and quantifies the overlap of top-1000 predicted interactions using pairwise Jaccard indices.

## When to use

Use when assessing how much the choice of CCC inference method versus prior-knowledge resource drives disagreement in ligand-receptor predictions from single-cell RNA-seq data.

## When NOT to use

- The goal is to determine which CCC method is best — no gold standard exists, so the benchmark cannot identify a definitive best method
- The methods of interest additionally infer intracellular processes (e.g. NicheNet, Cytotalk, SoptSC), which are not directly comparable in this magnitude-rank decoupling design
- Long-distance endocrine signalling is the question of interest — scRNA-seq gene expression does not capture it

## Inputs

- Six scRNA-seq datasets (three breast cancer subtypes, cord blood mononuclear cells, pancreatic islets, colorectal cancer)
- 16 ligand-receptor prior-knowledge resources (e.g. CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb)
- 7 CCC scoring methods (CellChat probability, CellPhoneDBv2 truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, iTALK logFC Mean, Cytotalk Crosstalk score)

## Outputs

- Top-1000 ranked interaction lists per method-resource-dataset combination
- Presence-absence matrices of predicted interactions per method and per resource
- median_jaccard_fig5.csv with median pairwise Jaccard indices (different methods ~0.080, different resources ~0.119)
- Fig 5-style similarity heatmap of Jaccard indices

## How to apply

Load the six scRNA-seq datasets (three breast cancer subtypes, cord blood mononuclear cells, pancreatic islets, colorectal cancer) and preprocess them with LIANA's pipeline. Decouple each of the 7 methods (CellChat probability, CellPhoneDBv2 truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, iTALK-inspired logFC Mean, Cytotalk-inspired Crosstalk score) from its default resource and run every method with each of the 16 resources on every dataset. Rank interactions by each method's magnitude rank, retain the top 1000 highest-ranked interactions per method-resource-dataset combination, and build presence-absence matrices. Compute pairwise Jaccard indices from these matrices and take the median across datasets separately for method-method and resource-resource comparisons; expect median Jaccard ~0.080 for different methods and ~0.119 for different resources, written to median_jaccard_fig5.csv and visualised as a Fig 5-style similarity heatmap.

## Related tools

- **LIANA** (Framework used to decouple methods from resources, run all method-resource combinations, and compute consensus/magnitude ranks) — https://github.com/saezlab/liana
- **CellChat** (Source of the probability-based CCC method and CellChatDB resource being benchmarked)
- **CellPhoneDB** (Source of the truncated-mean CCC method and its default resource being benchmarked)
- **Connectome** (Source of the weight_norm CCC method being benchmarked)
- **NATMI** (Source of the mean-expression edge-weight CCC method and ConnectomeDB resource being benchmarked) — https://github.com/asrhou/NATMI
- **SingleCellSignalR** (Source of the LRscore method and LRdb resource; a custom version allows custom ligand-receptor resources) — https://github.com/SCA-IRCM/SingleCellSignalR_v1
- **OmnipathR** (Client providing access to OmniPath ligand-receptor resources used in the resource comparison) — https://github.com/saezlab/OmnipathR

## Examples

```
liana_wrap(adata, method='natmi', resource='CellPhoneDB')  # run each method-resource combination in LIANA, then rank, binarize top-1000, and compute pairwise Jaccard
```

## Evaluation signals

- Median pairwise Jaccard across datasets for different methods should be ~0.080 and for different resources ~0.119, matching the values written to median_jaccard_fig5.csv
- Every one of the 7 methods must be run against all 16 resources on all six datasets (full method-resource grid)
- Exactly the top 1000 highest-ranked interactions by each method's magnitude rank are retained per combination
- Jaccard indices are computed from binary presence-absence matrices and reported as medians across datasets, not raw values

## Limitations

- No gold standard is available and the biological ground truth for CCC is largely unknown, so no definitive best method can be identified
- Benchmark analyses cannot distinguish simple co-expression from actual CCC events and do not capture complex relationships between CCC events
- Top-ranked predicted interactions show consistently low overlap (median Jaccard 0.080–0.119) regardless of whether method or resource is varied, so results are strongly sensitive to both choices
- Gene expression is a proxy of protein levels alone; processes preceding transmitter-receiver interactions (translation, secretion, diffusion) and long-distance endocrine signalling are not captured

## Evidence

- [results] median pairwise Jaccard indices of 0.080 (different methods) and 0.119 (different resources): [evidence span withheld — anchor status: not_found]
- [abstract] we systematically compare 16 cell-cell communication inference resources and 7 methods: [evidence span withheld — anchor status: not_found]
- [intro] we analysed how different combinations of resources and methods influence CCC inference by decoupling the methods from their corresponding resources: "Then, we analysed how different combinations of resources and methods influence CCC inference by decoupling the methods from their corresponding resources and applying all method-resource"
- [readme] LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [discussion] Since a gold standard is currently not available and the biological ground truth is largely unknown: "Since a gold standard is currently not available and the biological ground truth is largely unknown2,25, our analyses cannot give a definitive answer of what method is best."

---
name: network-topology-metric-computation
description: Use when comparing multiple GRN inference methods on single-cell expression
  data and you need quantitative, method-agnostic topology and reproducibility metrics
  (scale-free fit, edge counts, betweenness variance, edge overlap) to judge which
  method's networks are structurally most plausible.
when_to_use_negative:
- The network has not yet been inferred — this skill evaluates existing GRNs and cannot
  substitute for running SCING, ppcor, PIDC, or GRNBOOST2 on the expression data
- You need perturbation-based accuracy validation (e.g., perturb-seq target prediction
  or cosine-similarity held-out expression tests) rather than structural topology
  comparison
- The network was built from spatial transcriptomics where spatial information should
  be leveraged during construction — SCING itself does not use spatial coordinates
  in GRN construction, so topology metrics alone will not capture spatial specificity
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: GRN inference method under evaluation; its networks are assessed for scale-free
    structure alongside competitor methods
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Gradient boosting-based GRN method compared in the topology benchmark
- name: ppcor
  role: Partial correlation-based GRN method compared in the topology benchmark
- name: PIDC
  role: Mutual information-based GRN method compared in the topology benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:223a32630c60c88d3ad7fc9364197e1cb22226d4f15f98035d085f303e64e149
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# network-topology-metric-computation

## Summary

Compute structural topology metrics on inferred gene regulatory networks — scale-free degree-distribution fit R-squared, edge/gene counts, betweenness centrality variance, and normalized edge overlap between independently built networks — to benchmark GRN inference methods such as SCING, ppcor, PIDC, and GRNBOOST2.

## When to use

Use when comparing multiple GRN inference methods on single-cell expression data and you need quantitative, method-agnostic topology and reproducibility metrics (scale-free fit, edge counts, betweenness variance, edge overlap) to judge which method's networks are structurally most plausible.

## When NOT to use

- The network has not yet been inferred — this skill evaluates existing GRNs and cannot substitute for running SCING, ppcor, PIDC, or GRNBOOST2 on the expression data
- You need perturbation-based accuracy validation (e.g., perturb-seq target prediction or cosine-similarity held-out expression tests) rather than structural topology comparison
- The network was built from spatial transcriptomics where spatial information should be leveraged during construction — SCING itself does not use spatial coordinates in GRN construction, so topology metrics alone will not capture spatial specificity

## Inputs

- Inferred GRN network objects (edge lists or graph objects) from SCING, ppcor, PIDC, or GRNBOOST2, one per method x cell type x replicate
- snRNAseq expression matrices per cell type (e.g., oligodendrocyte, astrocyte, microglia) used to derive the networks
- Gene expression sparsity values (fraction of cells with zero expression per gene) for the sparsity > 0.7 exclusion filter

## Outputs

- Per-network scale-free fit R-squared values
- Per-network edge counts and gene counts
- Per-network betweenness centrality variance
- Normalized edge overlap scores between GRNs built on the two 50/50 cell-split halves
- Runtime measurements (seconds per GRN) across gene and cell subset sizes
- A consolidated metrics table (grn_benchmark_metrics.csv) covering all methods x cell types x replicates

## How to apply

For each inferred GRN (built per cell type and method from snRNAseq expression matrices), load the network into Python networkx and compute: (1) the R-squared of a scale-free fit on the node degree distribution, excluding genes whose expression sparsity exceeds 0.7; (2) total edge counts and gene (node) counts; (3) the variance of betweenness centrality across nodes. To quantify reproducibility, split the cells 50/50 into two independent subsets, infer a GRN on each half with the same method, and compute the normalized edge overlap score between the two networks. Optionally time GRN construction across gene subsets (1,000/2,000/4,000 genes) and cell subsets (250/500/1,000 cells) to compare computational efficiency. Aggregate all topology, overlap, and runtime values for every method x cell type x replicate into a single table (e.g., grn_benchmark_metrics.csv) with no missing rows.

## Related tools

- **SCING** (GRN inference method under evaluation; its networks are assessed for scale-free structure alongside competitor methods) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Gradient boosting-based GRN method compared in the topology benchmark)
- **ppcor** (Partial correlation-based GRN method compared in the topology benchmark)
- **PIDC** (Mutual information-based GRN method compared in the topology benchmark)

## Examples

```
import networkx as nx; G = nx.from_edgelist(edges, create_using=nx.DiGraph); r2 = fit_scale_free_powerlaw(G.degree()); bet_var = np.var(list(nx.betweenness_centrality(G).values()))
```

## Evaluation signals

- The metrics table contains complete entries for all 4 methods x 3 cell types x 10 replicates with no missing method or cell-type rows
- Every network row has non-missing values for scale-free R-squared, edge count, gene count, betweenness centrality variance, normalized edge overlap, and runtime
- Genes with expression sparsity above 0.7 are excluded from the scale-free degree-distribution fit
- Edge overlap scores are only computed between network pairs built on the two independent 50/50 cell-split halves of the same cell type and method
- Runtime is recorded for each method across the 1,000/2,000/4,000 gene and 250/500/1,000 cell subsets, in seconds per GRN

## Limitations

- Links between genes in the inferred GRNs are not necessarily causal or directional, so topology metrics do not certify regulatory causality
- Many hyperparameters require selection, and selection can be dataset dependent, which may affect the networks being measured
- The scale-free fit is sensitive to gene sparsity and must exclude highly sparse genes (sparsity above 0.7) to be meaningful
- Topology metrics alone capture structural plausibility, not accuracy — the paper supplements them with perturbation target prediction, held-out cosine-similarity tests, and disease subnetwork modeling

## Evidence

- [results] SCING networks more closely follow a scale-free network structure than the other methods: [evidence span withheld — anchor status: not_found]
- [intro] Evaluate against competitor methods via network characteristic metrics: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics"
- [readme] Network construction workflow from subsamples and supercells: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
- [intro] Existing methods limited by sparsity and heterogeneity: [evidence span withheld — anchor status: not_found]
- [discussion] Links are not necessarily causal or directional: "the links between genes are not necessarily causal or directional"

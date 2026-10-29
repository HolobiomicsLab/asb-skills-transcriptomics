---
name: betweenness-centrality-analysis
description: 'Use when comparing GRN inference methods on topology: you have inferred
  network graphs (edges between genes) and need scale-free fit R-squared, edge/gene
  counts, betweenness centrality variance, and split-half edge overlap scores.'
when_to_use_negative:
- You need perturbation-accuracy evaluation (AUROC / TPR at FPR 0.05 on Perturb-seq
  data) rather than network topology statistics
- Your networks are not GRN graphs between genes (e.g., protein-protein interaction
  or co-occurrence networks) where the scale-free fit assumption is not the benchmark's
  target
- You need biological interpretation of GRN modules (leiden subnetworks, AUCell scores,
  pathway enrichment) rather than quantitative topology comparison
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: SCING
  role: GRN inference method benchmarked; its networks are evaluated for scale-free
    structure and topology metrics
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Gradient boosting GRN method used as a topology-comparison baseline
- name: ppcor
  role: Partial correlation GRN method used as a topology-comparison baseline
- name: PIDC
  role: Mutual information GRN method used as a topology-comparison baseline
- name: networkx
  role: Python graph library used to compute scale-free fit R-squared, edge/gene counts,
    and betweenness centrality variance for every GRN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:75ac9a5b42d4d0809535923bf917d1567b70fcfa62979394435ad36cb2208243
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - networkx
  - ppcor
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# betweenness-centrality-analysis

## Summary

Compute network topology metrics (scale-free degree-distribution fit R-squared, edge/gene counts, betweenness centrality variance, normalized edge overlap) for inferred gene regulatory networks to compare GRN methods such as SCING, ppcor, PIDC, and GRNBOOST2.

## When to use

Use when comparing GRN inference methods on topology: you have inferred network graphs (edges between genes) and need scale-free fit R-squared, edge/gene counts, betweenness centrality variance, and split-half edge overlap scores.

## When NOT to use

- You need perturbation-accuracy evaluation (AUROC / TPR at FPR 0.05 on Perturb-seq data) rather than network topology statistics
- Your networks are not GRN graphs between genes (e.g., protein-protein interaction or co-occurrence networks) where the scale-free fit assumption is not the benchmark's target
- You need biological interpretation of GRN modules (leiden subnetworks, AUCell scores, pathway enrichment) rather than quantitative topology comparison

## Inputs

- Inferred GRN graphs (edge lists between genes) from SCING, GRNBOOST2, ppcor, and PIDC runs
- snRNAseq expression matrices for cell types (e.g., oligodendrocyte, astrocyte, microglia) with per-gene sparsity values
- Two 50/50 cell-split halves per cell type for reproducibility (overlap) scoring
- Runtime logs (seconds per GRN) across gene subsets (1,000/2,000/4,000 genes) and cell subsets (250/500/1,000 cells)

## Outputs

- Per-GRN topology metrics: scale-free fit R-squared, edge counts, gene counts, betweenness centrality variance
- Normalized edge overlap scores between 50/50 split-half GRNs per method and cell type
- Runtime (seconds per GRN) per method across gene and cell subset sizes
- Aggregated metrics table (grn_benchmark_metrics.csv) for all 4 methods x 3 cell types x 10 replicates

## How to apply

Infer replicate GRNs per cell type and method using each method's native implementation (e.g., 10 replicates per method per cell type for SCING, GRNBOOST2, ppcor, and PIDC). Load each network into networkx as a graph and compute: scale-free fit R-squared on the degree distribution, excluding genes with expression sparsity above 0.7; total edge and gene counts; and the variance of betweenness centrality across nodes. To quantify reproducibility, split cells 50/50 into two independent subsets, build networks on each half, and compute a normalized edge overlap score between the two GRNs. Aggregate topology, overlap, and per-GRN runtime measurements into a single table (grn_benchmark_metrics.csv) covering all method x cell type x replicate combinations with no missing rows.

## Related tools

- **SCING** (GRN inference method benchmarked; its networks are evaluated for scale-free structure and topology metrics) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Gradient boosting GRN method used as a topology-comparison baseline)
- **ppcor** (Partial correlation GRN method used as a topology-comparison baseline)
- **PIDC** (Mutual information GRN method used as a topology-comparison baseline)
- **networkx** (Python graph library used to compute scale-free fit R-squared, edge/gene counts, and betweenness centrality variance for every GRN)

## Examples

```
import networkx as nx; G = nx.from_pandas_edgelist(edges, source='TF', target='gene', create_using=nx.DiGraph); bc_var = pd.Series(nx.betweenness_centrality(G)).var()
```

## Evaluation signals

- Metrics table (grn_benchmark_metrics.csv) contains complete entries for all 4 methods x 3 cell types x 10 replicates with no missing method or cell-type rows
- Every GRN row has non-null scale-free R-squared, edge count, gene count, betweenness centrality variance, normalized edge overlap, and runtime values
- Genes with expression sparsity above 0.7 are excluded from the scale-free degree-distribution fit
- Normalized edge overlap is computed between GRNs built on the two independent 50/50 cell-split halves for each method and cell type
- Runtimes are recorded in seconds per GRN across gene subsets (1,000/2,000/4,000) and cell subsets (250/500/1,000)

## Limitations

- Inferred GRN links are not necessarily causal or directional, so topology metrics describe association structure rather than verified regulatory direction
- Hyperparameter selection can be dataset dependent, which may confound cross-method topology comparisons
- Genes with expression sparsity above 0.7 are excluded from the scale-free fit, so sparse-gene topology is not represented in the R-squared metric

## Evidence

- [intro] scale-free fit via degree distribution: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [other] SCING topology comparison finding: [evidence span withheld — anchor status: not_found]
- [abstract] SCING accuracy claim from article entities: "Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability of"
- [readme] Workflow from README: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."

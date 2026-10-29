---
name: grn-reproducibility-overlap-assessment
description: Use when benchmarking multiple GRN inference methods on single-cell (snRNAseq/scRNAseq)
  expression matrices and you need quantified edge reproducibility, scale-free topology
  fit, network characteristic metrics, and runtime per method.
when_to_use_negative:
- When the goal is perturbation target-prediction accuracy in Perturb-seq data rather
  than reproducibility/topology benchmarking — the article treats perturb-seq AUROC/TPR-at-FPR-0.05
  evaluation as a separate validation axis
- When assessing a single pre-built GRN with no cell replicates, since the overlap
  score requires two independent network builds on split cell halves
- When links must be interpreted as causal/directional — the article notes inferred
  GRN links 'are not necessarily causal or directional', so overlap of edges is a
  structural, not mechanistic, reproducibility measure
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SCING
  role: GRN inference method under benchmark; expected to yield the highest scale-free
    R-squared networks
  repo: github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Gradient-boosting GRN inference baseline in the reproducibility and runtime
    comparison
- name: ppcor
  role: Partial-correlation GRN inference baseline in the comparison
- name: PIDC
  role: Information-theoretic GRN inference baseline in the comparison
- name: networkx
  role: Compute degree-distribution scale-free fit R-squared, edge/gene counts, and
    betweenness centrality variance for every inferred GRN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:78f17b3d0c6858dddd44db3a6b8f28c56872004031e6e9e57a177b6ea05f0366
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - networkx
  - ppcor
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# grn-reproducibility-overlap-assessment

## Summary

Quantifies how reproducible and structurally consistent GRNs are across replicate inference runs: normalized edge overlap between networks built on independent 50/50 cell-split halves, scale-free fit R-squared on the degree distribution, edge/gene counts, betweenness centrality variance, and seconds-per-network runtime, benchmarked across SCING, GRNBOOST2, ppcor, and PIDC.

## When to use

Use when benchmarking multiple GRN inference methods on single-cell (snRNAseq/scRNAseq) expression matrices and you need quantified edge reproducibility, scale-free topology fit, network characteristic metrics, and runtime per method.

## When NOT to use

- When the goal is perturbation target-prediction accuracy in Perturb-seq data rather than reproducibility/topology benchmarking — the article treats perturb-seq AUROC/TPR-at-FPR-0.05 evaluation as a separate validation axis
- When assessing a single pre-built GRN with no cell replicates, since the overlap score requires two independent network builds on split cell halves
- When links must be interpreted as causal/directional — the article notes inferred GRN links 'are not necessarily causal or directional', so overlap of edges is a structural, not mechanistic, reproducibility measure

## Inputs

- snRNAseq expression matrices for individual cell types (e.g., oligodendrocyte, astrocyte, microglia)
- Per-gene expression sparsity annotations (for applying the sparsity > 0.7 exclusion)
- Native GRN inference implementations for SCING, GRNBOOST2, ppcor, and PIDC

## Outputs

- grn_benchmark_metrics.csv with scale-free R-squared, edge counts, gene counts, betweenness centrality variance, normalized edge overlap, and runtime for all method x cell type x replicate combinations
- Per-GRN networkx topology metric table
- Normalized edge overlap scores between 50/50 cell-split network halves per method and cell type
- Runtime table (seconds per GRN) across 1,000/2,000/4,000 gene subsets and 250/500/1,000 cell subsets

## How to apply

Load the snRNAseq expression matrices per cell type (e.g., oligodendrocyte, astrocyte, microglia), sample 3,000 random genes per type, and split cells 50/50 into two independent halves. Infer 10 replicate GRNs per cell type per method (SCING, GRNBOOST2, ppcor, PIDC) using each method's native implementation for a total of 120 networks. Compute topology metrics with networkx for every GRN: scale-free fit R-squared on the degree distribution (excluding genes with expression sparsity above 0.7), edge counts, gene counts, and betweenness centrality variance; then compute the normalized edge overlap score between the two 50/50 halves to quantify reproducibility of inferred edges. Time GRN construction across gene subsets (1,000/2,000/4,000 genes) and cell subsets (250/500/1,000 cells), recording seconds per GRN. Aggregate all topology, overlap, and runtime measurements into grn_benchmark_metrics.csv and validate completeness for all 4 methods x 3 cell types x 10 replicates.

## Related tools

- **SCING** (GRN inference method under benchmark; expected to yield the highest scale-free R-squared networks) — github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Gradient-boosting GRN inference baseline in the reproducibility and runtime comparison)
- **ppcor** (Partial-correlation GRN inference baseline in the comparison)
- **PIDC** (Information-theoretic GRN inference baseline in the comparison)
- **networkx** (Compute degree-distribution scale-free fit R-squared, edge/gene counts, and betweenness centrality variance for every inferred GRN)

## Examples

```
import networkx as nx; G = nx.from_pandas_edgelist(grn_edges_df, source='regulator', target='target'); bet_var = np.var(list(nx.betweenness_centrality(G).values()))
```

## Evaluation signals

- grn_benchmark_metrics.csv contains complete entries for all 4 methods x 3 cell types x 10 replicates (120 networks) with no missing method or cell-type rows
- Scale-free fit R-squared excludes genes with expression sparsity above 0.7 from the degree distribution
- Normalized edge overlap is computed between the two 50/50 cell-split halves for every method x cell type pair
- SCING's scale-free R-squared should be significantly higher than PIDC and GRNBOOST2 and trending higher than ppcor, per the article's finding
- Runtime entries exist for each gene subset (1,000/2,000/4,000) and cell subset (250/500/1,000) per method

## Limitations

- Inferred GRN links are not necessarily causal or directional, so edge overlap measures structural reproducibility rather than correctness of regulatory mechanism
- GRN methods have many hyperparameters whose selection can be dataset dependent, which may confound cross-method reproducibility comparisons
- Genes with expression sparsity above 0.7 are excluded from the scale-free fit, so topology metrics reflect a sparsity-filtered gene set rather than the full sampled transcriptome

## Evidence

- [intro] We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics"
- [readme] We split data into train and test sets and build a network on the train set.: "Gene prediction validation, to determine network overfitting (b). We split data into train and test sets and build a network on the train set."
- [readme] We utilize bagging through subsamples of supercells to keep robust edges in the final GRN.: "We utilize bagging through subsamples of supercells to keep robust edges in the final GRN."

---
name: gene-network-benchmarking-metrics
description: Use when comparing gene network inference methods or model variants against
  a curated directed binary ground truth (e.g. Omnipath) and reporting AUPRC and EPR
  per cell type or condition.
when_to_use_negative:
- When no literature or experimental ground truth exists for the gene set of interest
  — the article notes that no perfect ground truths exist for GN inference benchmarking,
  so metrics must be interpreted cautiously
- When comparing methods run on different gene sets or different cell subsets — the
  benchmark requires identical cells and genes across methods for fair EPR/AUPRC comparison
- When evaluating causal directionality — Omnipath sources have sparse out-degree,
  requiring network transposition, so the benchmark is about literature-curated interaction
  recovery, not causality
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BenGRN
  role: Benchmark package used to compute EPR and prevalence-floored AUPRC of inferred
    gene networks against ground truths like Omnipath
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Benchmarking companion package for gene networks inferred from single-cell
    RNA sequencing
  repo: https://github.com/cantinilab/GRnnData
- name: Omnipath
  role: Source of the directed binary ground-truth interaction network built from
    all interactions in the Omnipath Python package
- name: scikit-learn
  role: RidgeClassifier (L2 penalty 1, positive coefficients, no intercept) used to
    select informative attention heads for one scPRINT network variant
- name: GENIE3
  role: Regression-based baseline (default parameters, 100 trees) used as the reference
    method for out-performance and connection-recovery comparisons
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:e1e3b007f77199781e2bbb0d3d1b0596b80d70950acaaa9d0e7f9af7b4ec45c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jkobject/benGRN
  tools:
  - BenGRN
  - DeepSEM
  - GENIE3
  - GRnnData
  - Geneformer
  - Omnipath
  - Python
  - scGPT
  - scPRINT
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# gene-network-benchmarking-metrics

## Summary

Benchmark inferred gene networks (e.g. scPRINT, scGPT, Geneformer, DeepSEM, GENIE3 variants) against a literature-derived directed ground truth such as Omnipath using prevalence-floored AUPRC and EPR (diagnostic odds ratio at K = number of ground-truth positives), implemented via the BenGRN/GRnnData benchmark suite.

## When to use

Use when comparing gene network inference methods or model variants against a curated directed binary ground truth (e.g. Omnipath) and reporting AUPRC and EPR per cell type or condition.

## When NOT to use

- When no literature or experimental ground truth exists for the gene set of interest — the article notes that no perfect ground truths exist for GN inference benchmarking, so metrics must be interpreted cautiously
- When comparing methods run on different gene sets or different cell subsets — the benchmark requires identical cells and genes across methods for fair EPR/AUPRC comparison
- When evaluating causal directionality — Omnipath sources have sparse out-degree, requiring network transposition, so the benchmark is about literature-curated interaction recovery, not causality

## Inputs

- Directed binary source-to-target ground truth network from the Omnipath Python package
- Single-cell RNA-seq subsets per cell type (1024 cells) from kidney, retina, and colon atlases
- Top 5000 differentially expressed genes defining the shared gene space
- Inferred weighted adjacency matrices from each method (attention-based or regression-based)
- GRnnData/BenGRN benchmark objects for metric computation

## Outputs

- Per-method, per-cell-type EPR values (diagnostic odds ratio at K = ground-truth positives)
- Per-method, per-cell-type prevalence-floored AUPRC values
- Fraction of ground-truth connections recovered relative to a reference method (e.g. scPRINT vs GENIE3)
- Aggregated results table such as omnipath_benchmark_results.csv

## How to apply

Build the ground truth as a directed binary source-to-target network from the Omnipath Python package, excluding small molecules, lncRNAs, and elements without a unique HGNC symbol. Run each method on the same cells (e.g. 1024 cells per cell type) and the same gene set (e.g. the 5000 most differentially expressed genes), then take the transpose of each inferred network so Omnipath sources map to network sources — Omnipath sources have sparse out-degree. Compute EPR as the diagnostic odds ratio (TP×TN)/(FP×FN) at the cutoff giving K = number of ground-truth positives, and prevalence-floored AUPRC, using the BenGRN and GRnnData packages. Validation criterion: an EPR > 1 is required for a predictor to beat random (EPR = 1 is random, infinity is perfect); in the scPRINT study, acceptance required the scPRINT variants to outperform GENIE3 on the Omnipath benchmark. Results are assembled per method and per cell type (e.g. into omnipath_benchmark_results.csv).

## Related tools

- **BenGRN** (Benchmark package used to compute EPR and prevalence-floored AUPRC of inferred gene networks against ground truths like Omnipath) — https://github.com/jkobject/benGRN
- **GRnnData** (Benchmarking companion package for gene networks inferred from single-cell RNA sequencing) — https://github.com/cantinilab/GRnnData
- **Omnipath** (Source of the directed binary ground-truth interaction network built from all interactions in the Omnipath Python package)
- **scikit-learn** (RidgeClassifier (L2 penalty 1, positive coefficients, no intercept) used to select informative attention heads for one scPRINT network variant)
- **GENIE3** (Regression-based baseline (default parameters, 100 trees) used as the reference method for out-performance and connection-recovery comparisons)

## Examples

```
from sklearn.linear_model import RidgeClassifier; head_selector = RidgeClassifier(alpha=1.0, positive=True, fit_intercept=False).fit(head_features, omnipath_edge_labels)
```

## Evaluation signals

- EPR > 1 for a predictor to be considered better than random (EPR = 1 is random, infinity is perfect)
- scPRINT variants must outperform GENIE3 on the Omnipath benchmark (both AUPRC and EPR) for the reported finding to hold
- All methods must be run on identical cells (1024 per cell type) and identical genes (5000 most DE) before metric computation
- Inferred networks must be transposed before comparison so Omnipath sources match network sources (sparse out-degree check)
- Ground truth must be a directed binary network excluding small molecules, lncRNAs, and non-unique HGNC symbols

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths are scarce and ambiguous
- EPR is evaluated only at a single cutoff (K = number of ground-truth positives), not across the full precision-recall curve
- Many GN methods are benchmarked on ODE-generated mock-up expression data rather than real ground truths, limiting comparability

## Evidence

- [methods] We generate the Omnipath network using all the interactions from the Omnipath Python package: "We generate the Omnipath network using all the interactions from the Omnipath Python package"
- [methods] RidgeClassifier head selection: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [discussion] BenGRN and GRnnData benchmark suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [discussion] No perfect ground truths: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [discussion] GENIE3 as competitive baseline: "while GENIE3 is still a competitive tool, we outperformed it on many of our benchmarks"

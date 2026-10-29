---
name: gene-network-heads-selection
description: Use when extracting a gene regulatory network from a pretrained transformer's
  attention matrices and you must decide which attention heads to average, using a
  literature ground truth such as Omnipath.
when_to_use_negative:
- There is no ground-truth edge set (e.g., Omnipath) available to train the RidgeClassifier
  for head selection — the all-head average or genome-wide variant must be used instead
- The goal is de novo GRN inference from expression counts alone without a pretrained
  transformer (use GENIE3 or DeepSEM directly)
- Benchmarking on ODE-generated mock-up expression data rather than literature-based
  or experimental ground truths, which the article cautions against
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPRINT
  role: Pretrained foundation model whose attention heads (Keys/Queries averaged over
    cells U) are extracted and combined into candidate gene networks
  repo: https://github.com/cantinilab/scPRINT
- name: BenGRN
  role: Benchmarking package used to compute EPR and prevalence-floored AUPRC against
    Omnipath
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene regulatory network data structure used with BenGRN for benchmarking inferred
    networks
  repo: https://github.com/cantinilab/GRnnData
- name: Omnipath
  role: Python package providing the ground-truth interactions used to train the RidgeClassifier
    and evaluate networks
- name: scikit-learn
  role: Provides the RidgeClassifier (L2 penalty 1, positive coefficients, no intercept)
    used for attention head selection
- name: GENIE3
  role: Baseline (100 trees, default parameters) against which recovery of Omnipath
    connections is compared
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:bb1a5b4254610ad57d37fda62d1fb531cc9dc833002a8bdaa3a50c778a363763
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# gene-network-heads-selection

## Summary

Select informative transformer attention heads and aggregate them into a gene network using a RidgeClassifier trained on ground-truth edges (e.g., Omnipath), as done for scPRINT's gene network inference benchmark against GENIE3, scGPT, Geneformer, and DeepSEM.

## When to use

Use when extracting a gene regulatory network from a pretrained transformer's attention matrices and you must decide which attention heads to average, using a literature ground truth such as Omnipath.

## When NOT to use

- There is no ground-truth edge set (e.g., Omnipath) available to train the RidgeClassifier for head selection — the all-head average or genome-wide variant must be used instead
- The goal is de novo GRN inference from expression counts alone without a pretrained transformer (use GENIE3 or DeepSEM directly)
- Benchmarking on ODE-generated mock-up expression data rather than literature-based or experimental ground truths, which the article cautions against

## Inputs

- Pretrained (non-fine-tuned) scPRINT checkpoint from Hugging Face (https://huggingface.co/jkobject/scPRINT)
- Cell-type-specific single-cell expression subsets (~1024 cells per cell type) from kidney, retina, colon datasets
- Omnipath ground truth built from the Omnipath Python package as a directed binary source-to-target network
- Set of 5000 most differentially expressed genes per cell type

## Outputs

- Per-cell-type gene networks (three scPRINT variants: all-head average, RidgeClassifier-selected heads average, genome-wide Keys/Queries average)
- Per-method, per-cell-type AUPRC and EPR values (e.g., assembled into omnipath_benchmark_results.csv)
- Fitted RidgeClassifier head coefficients used for head selection

## How to apply

Extract attention matrices from the pretrained scPRINT checkpoint (loaded from Hugging Face) on ~1024 cells per cell type using the 5000 most differentially expressed genes, then average Keys and Queries embeddings over the set of cells U to produce per-head networks. To select heads, train a RidgeClassifier from scikit-learn on Omnipath edges with an L2 penalty set to 1, a positivity constraint on the coefficients, and no intercept; average only heads with positive coefficients. Produce three scPRINT network variants: average over all attention heads, average over RidgeClassifier-selected heads, and the genome-wide network. Take the transpose of each inferred network so regulators map to sources and targets to regulators (Omnipath sources have sparse out-degree), then evaluate with EPR (diagnostic odds ratio (TP×TN)/(FP×FN) at K = number of ground-truth positives) and prevalence-floored AUPRC using the BenGRN/GRnnData packages.

## Related tools

- **scPRINT** (Pretrained foundation model whose attention heads (Keys/Queries averaged over cells U) are extracted and combined into candidate gene networks) — https://github.com/cantinilab/scPRINT
- **BenGRN** (Benchmarking package used to compute EPR and prevalence-floored AUPRC against Omnipath) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene regulatory network data structure used with BenGRN for benchmarking inferred networks) — https://github.com/cantinilab/GRnnData
- **Omnipath** (Python package providing the ground-truth interactions used to train the RidgeClassifier and evaluate networks)
- **scikit-learn** (Provides the RidgeClassifier (L2 penalty 1, positive coefficients, no intercept) used for attention head selection)
- **GENIE3** (Baseline (100 trees, default parameters) against which recovery of Omnipath connections is compared)

## Examples

```
from sklearn.linear_model import RidgeClassifier; clf = RidgeClassifier(alpha=1.0, positive=True, fit_intercept=False); clf.fit(head_features, omnipath_edges); sel = clf.coef_ > 0
```

## Evaluation signals

- EPR > 1 is required for a predictor to beat random (EPR = 1 is random; EPR → infinity is perfect); scPRINT variants should exceed GENIE3 on the Omnipath benchmark
- scPRINT should recover ~67% more Omnipath connections than GENIE3 on average (scGPT ~42%)
- Omnipath ground truth must exclude small molecules, lncRNAs, and elements without a unique HGNC symbol before comparison
- Inferred networks must be transposed so rows/columns match Omnipath source→target orientation before computing EPR/AUPRC
- Per-method, per-cell-type AUPRC and EPR values should assemble cleanly into omnipath_benchmark_results.csv across all 26 cell types

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths like Omnipath remain scarce and ambiguous
- General collections like Omnipath did not improve performance in most of the paper's tests when used beyond benchmarking
- EPR = 1 corresponds to random performance, so evaluation near the random baseline is uninformative

## Evidence

- [methods] heads selection via RidgeClassifier on ground truth edges: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [methods] gene network extraction from averaged attention matrices: [evidence span withheld — anchor status: not_found]
- [methods] Omnipath ground truth construction: "We generate the Omnipath network using all the interactions from the Omnipath Python package"
- [discussion] benchmark conclusion vs GENIE3: "while GENIE3 is still a competitive tool, we outperformed it on many of our benchmarks"
- [discussion] benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"

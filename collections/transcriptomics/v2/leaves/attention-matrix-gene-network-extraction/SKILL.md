---
name: attention-matrix-gene-network-extraction
description: Use when you have a pretrained single-cell transformer (scPRINT, scGPT,
  Geneformer v2) and want a zero-shot gene network for a cell type or cell line that
  must be scored against a directed ground-truth network.
when_to_use_negative:
- No ground-truth network (ChIP-seq, perturb-seq, or literature-based) exists for
  the cell type — the article notes 'no perfect ground truths exist' and EPR > 1 acceptance
  requires a positive set
- The expression data are ODE-generated mock-ups rather than real single-cell profiles,
  which the article criticizes as a weak benchmarking basis
- You need causal, genome-wide, cell-type-specific ground truth — the article flags
  that such networks remain scarce and ambiguous
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Foundation model whose Keys/Queries and attention heads are averaged to infer
    gene networks
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Baseline GRN extraction via last-layer head-averaged attention without sqrt(dk)
    rescaling
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: Baseline GRN extraction via per-cell mean of all attention heads mapped back
    to full network size
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: GENIE3
  role: Tree-ensemble regression baseline (default parameters, 100 trees) on raw counts
- name: DeepSEM
  role: Structural-equation baseline run after 10,000-count normalization, log1p,
    and per-gene z-scoring
- name: BenGRN
  role: Benchmarking package for EPR and prevalence-floored AUPRC computation
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Benchmarking/data structure package used with BenGRN to score inferred networks
  repo: https://github.com/cantinilab/GRnnData
- name: Omnipath
  role: Literature-derived ground-truth directed interaction network
- name: scikit-learn
  role: RidgeClassifier (L2 penalty 1, positive coefficients, no intercept) for attention-head
    selection
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:6bb0d24771ad1fc03bcefec9e1ffe27c64f0ffee3d00a73cae3970ebb534c040
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# attention-matrix-gene-network-extraction

## Summary

Extract gene networks from transformer foundation-model attention matrices (e.g. scPRINT, scGPT, Geneformer v2) by averaging Keys/Queries embeddings or per-head attention over a cell set, optionally selecting informative heads with a RidgeClassifier, and benchmark the result against Omnipath or perturb-seq ground truths with AUPRC and EPR.

## When to use

Use when you have a pretrained single-cell transformer (scPRINT, scGPT, Geneformer v2) and want a zero-shot gene network for a cell type or cell line that must be scored against a directed ground-truth network.

## When NOT to use

- No ground-truth network (ChIP-seq, perturb-seq, or literature-based) exists for the cell type — the article notes 'no perfect ground truths exist' and EPR > 1 acceptance requires a positive set
- The expression data are ODE-generated mock-ups rather than real single-cell profiles, which the article criticizes as a weak benchmarking basis
- You need causal, genome-wide, cell-type-specific ground truth — the article flags that such networks remain scarce and ambiguous

## Inputs

- Pretrained scPRINT checkpoint from Hugging Face (jkobject/scPRINT)
- Single-cell expression matrix (.h5ad or cellxgene atlas subset) with cell-type annotations
- Omnipath Python package interactions (directed binary source-to-target network, excluding small molecules, lncRNAs, non-unique HGNC symbols)
- gwps supplementary multiple-hypothesis-testing-corrected p-value matrix for K562 (binarized at 0.05 cutoff)
- Baseline model checkpoints (scGPT_human/best_model.pt, Geneformer v2 weights) and SERGIO/RegNetwork simulated data (3546 connections, 813 genes, 1000 cells)

## Outputs

- Per-method, per-cell-type AUPRC and EPR table (e.g. omnipath_benchmark_results.csv, gwps_benchmark_metrics.csv)
- Directed gene-by-gene network matrices from scPRINT (all-heads average, selected-heads average, genome-wide Keys/Queries average) and each baseline
- Fraction of scPRINT network connections recovered relative to GENIE3

## How to apply

Run the pretrained (non-fine-tuned) scPRINT checkpoint from Hugging Face on a subset of cells (e.g. 1024 cells per cell type, 5000 most differentially expressed genes) and build the genome-wide network by averaging Keys and Queries embeddings over the cell set U; also produce variants averaging over all attention heads and over heads selected by a RidgeClassifier (L2 penalty 1, positive coefficients, no intercept) trained on ground-truth edges. Run comparators on the same cells and genes: scGPT (last-layer head-averaged attention with softmax without sqrt(dk) rescaling, cell token dropped), Geneformer v2 (mean of all attention heads per cell, mapped back to full network size), DeepSEM (10,000-count normalization, log1p, per-gene z-score), and GENIE3 (default, 100 trees). Orient the inferred network to match the ground truth: for Omnipath take the transpose (sources have sparse out-degree); for gwps use regulators as columns and sparsely regulated targets as rows with no transposition. Score with prevalence-floored AUPRC and EPR (diagnostic odds ratio (TP×TN)/(FP×FN) at the cutoff giving K = number of ground-truth positives) using the BenGRN/GRnnData packages.

## Related tools

- **scPRINT** (Foundation model whose Keys/Queries and attention heads are averaged to infer gene networks) — https://github.com/cantinilab/scPRINT
- **scGPT** (Baseline GRN extraction via last-layer head-averaged attention without sqrt(dk) rescaling) — https://github.com/jkobject/scGPT
- **Geneformer** (Baseline GRN extraction via per-cell mean of all attention heads mapped back to full network size) — https://huggingface.co/ctheodoris/Geneformer
- **GENIE3** (Tree-ensemble regression baseline (default parameters, 100 trees) on raw counts)
- **DeepSEM** (Structural-equation baseline run after 10,000-count normalization, log1p, and per-gene z-scoring)
- **BenGRN** (Benchmarking package for EPR and prevalence-floored AUPRC computation) — https://github.com/jkobject/benGRN
- **GRnnData** (Benchmarking/data structure package used with BenGRN to score inferred networks) — https://github.com/cantinilab/GRnnData
- **Omnipath** (Literature-derived ground-truth directed interaction network)
- **scikit-learn** (RidgeClassifier (L2 penalty 1, positive coefficients, no intercept) for attention-head selection)

## Examples

```
from sklearn.linear_model import RidgeClassifier; clf = RidgeClassifier(alpha=1.0, positive=True, fit_intercept=False); clf.fit(head_edge_features, omnipath_edge_labels)
```

## Evaluation signals

- EPR > 1 is required for a predictor to beat random (EPR = 1 is random, infinity is perfect)
- scPRINT variants should outperform GENIE3 on the Omnipath benchmark (67% more connections recovered on average; scGPT 42%)
- On gwps, scPRINT with gwps-selected heads should outperform GENIE3 (which otherwise ranks first, ahead of plain scPRINT)
- Network orientation must match the ground truth: transposed for Omnipath, non-transposed with regulators as columns for gwps
- AUPRC must be floored at the positive-class prevalence and computed with the composite trapezoidal rule without interpolation to perfect recall

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous
- Head selection via a RidgeClassifier trained on the ground truth means the 'selected heads' variant uses benchmark information and is not fully zero-shot
- Transformer ability to generate graphs and their explainability remain unresolved; general collections like Omnipath did not improve performance in most tests

## Evidence

- [methods] attention averaging over cells: [evidence span withheld — anchor status: not_found]
- [methods] head selection classifier: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [discussion] no perfect ground truths: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [results] network orientation for Omnipath: [evidence span withheld — anchor status: not_found]
- [discussion] GENIE3 remains competitive: "while GENIE3 is still a competitive tool, we outperformed it on many of our benchmarks"

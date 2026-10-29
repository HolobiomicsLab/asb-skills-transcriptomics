---
name: zero-shot-foundation-model-grn-inference
description: Use when inferring a gene network from scRNA-seq cells without training
  a model, or when benchmarking foundation-model-derived GRNs (scPRINT, scGPT, Geneformer)
  against GENIE3/DeepSEM on a ground truth like genome-wide perturb-seq K562.
when_to_use_negative:
- No validated ground-truth network (e.g. perturb-seq corrected p-values) exists for
  the cell type, since no perfect ground truths exist for GN inference benchmarking
- The task requires causal, cell-type-specific genome-wide networks from real experiments
  rather than zero-shot attention-based inference
- Only ODE-generated mock-up expression data are available for benchmarking, which
  the authors note is an unreliable substitute for real ground truths
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scPRINT
  role: Pretrained foundation model run zero-shot on K562 cells; gene networks extracted
    by averaging Keys and Queries embeddings over attention heads
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Comparator foundation model; attention GRN extracted via Tutorial_Attention_GRN.ipynb
    with mean over cells and last-layer heads
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: Comparator foundation model (v2); per-cell mean of all attention heads mapped
    back to full network size
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: GENIE3
  role: Tree-based comparator run with default parameters and 100 trees on raw counts
- name: DeepSEM
  role: Comparator run with default parameters after 10,000-count normalization, log1p,
    and per-gene z-scoring
- name: BenGRN
  role: Benchmarking suite used to match inferred networks to the gwps ground truth
    and compute metrics
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Data structure used with BenGRN to represent and evaluate gene regulatory
    networks
  repo: https://github.com/cantinilab/GRnnData
- name: Python
  role: Implementation language for the full inference and benchmarking workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:66b0e5b874bc1ce88e162822ab8cc5a03bfcb322289aefa114deeddd5e854c64
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
  - Python
  - scGPT
  - scPRINT
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# zero-shot-foundation-model-grn-inference

## Summary

Extract gene regulatory networks zero-shot from pretrained single-cell foundation models (scPRINT, scGPT, Geneformer v2) by averaging attention Key/Query embeddings over a cell set, and benchmark them against methods like GENIE3 and DeepSEM on a perturb-seq ground truth using AUPRC and EPR.

## When to use

Use when inferring a gene network from scRNA-seq cells without training a model, or when benchmarking foundation-model-derived GRNs (scPRINT, scGPT, Geneformer) against GENIE3/DeepSEM on a ground truth like genome-wide perturb-seq K562.

## When NOT to use

- No validated ground-truth network (e.g. perturb-seq corrected p-values) exists for the cell type, since no perfect ground truths exist for GN inference benchmarking
- The task requires causal, cell-type-specific genome-wide networks from real experiments rather than zero-shot attention-based inference
- Only ODE-generated mock-up expression data are available for benchmarking, which the authors note is an unreliable substitute for real ground truths

## Inputs

- Preprocessed scRNA-seq cell x gene count matrix for K562 cells (8000+ expressed genes)
- gwps supplementary multiple-hypothesis-testing-corrected p-value matrix for K562 (to binarize into a directed ground-truth network at 0.05)
- Pretrained scPRINT model checkpoint (e.g. from https://huggingface.co/jkobject/scPRINT)
- scGPT pretrained weights (scGPT_human/best_model.pt) and Tutorial_Attention_GRN.ipynb procedure
- Geneformer v2 model from https://huggingface.co/ctheodoris/Geneformer

## Outputs

- Inferred directed gene-by-gene networks per method (regulators as columns, sparsely regulated targets as rows)
- Per-method AUPRC and EPR metrics assembled into gwps_benchmark_metrics.csv
- gwps-selected attention head sets from the RidgeClassifier head selection step

## How to apply

Run pretrained scPRINT zero-shot on the K562 cells and extract gene networks by averaging the Keys and Queries embeddings over the cell set, both as a mean over all attention heads and restricted to heads selected against the ground truth with a RidgeClassifier (L2 penalty 1, positive coefficients, no intercept). Run comparators on the same cells and genes: scGPT via the Tutorial_Attention_GRN.ipynb procedure (mean over cells and last-layer heads, no sqrt(dk) rescaling, dropping the cell token), Geneformer v2 as the mean of all attention heads per cell mapped back to full network size, DeepSEM with default parameters after normalizing to 10,000 counts, log1p, and per-gene z-scoring, and GENIE3 with 100 trees on raw counts. Build the ground truth by binarizing the gwps supplementary multiple-hypothesis-testing-corrected p-value matrix for K562 at a 0.05 cutoff (8000+ expressed genes). Match each inferred network to the ground truth with regulators as columns and sparsely regulated targets as rows (no transposition) using benGRN and GRnnData. Score every method with AUPRC (area above the positive-class prevalence floor, composite trapezoidal rule, no interpolation to perfect recall) and EPR (diagnostic odds ratio (TP x TN)/(FP x FN) at the cutoff giving K positive predictions, where K equals the number of ground-truth positives), then assemble gwps_benchmark_metrics.csv.

## Related tools

- **scPRINT** (Pretrained foundation model run zero-shot on K562 cells; gene networks extracted by averaging Keys and Queries embeddings over attention heads) — https://github.com/cantinilab/scPRINT
- **scGPT** (Comparator foundation model; attention GRN extracted via Tutorial_Attention_GRN.ipynb with mean over cells and last-layer heads) — https://github.com/jkobject/scGPT
- **Geneformer** (Comparator foundation model (v2); per-cell mean of all attention heads mapped back to full network size) — https://huggingface.co/ctheodoris/Geneformer
- **GENIE3** (Tree-based comparator run with default parameters and 100 trees on raw counts)
- **DeepSEM** (Comparator run with default parameters after 10,000-count normalization, log1p, and per-gene z-scoring)
- **BenGRN** (Benchmarking suite used to match inferred networks to the gwps ground truth and compute metrics) — https://github.com/jkobject/benGRN
- **GRnnData** (Data structure used with BenGRN to represent and evaluate gene regulatory networks) — https://github.com/cantinilab/GRnnData
- **Python** (Implementation language for the full inference and benchmarking workflow)

## Examples

```
from scprint import scPrint; model = scPrint.load_from_checkpoint('model.ckpt'); net = model.get_genes_network(cells=k562_adata, average_over_heads='gwps')
```

## Evaluation signals

- gwps-selected-heads scPRINT EPR exceeds GENIE3's EPR in gwps_benchmark_metrics.csv (the paper's verification step)
- Ground-truth network is a directed binary gene-by-gene matrix built with a 0.05 cutoff on the corrected p-values over 8000+ expressed K562 genes
- Inferred networks align to the ground truth orientation with regulators as columns and sparsely regulated targets as rows (no transposition)
- EPR equals (TP x TN)/(FP x FN) evaluated at the score cutoff giving exactly K positive predictions, where K is the number of ground-truth positives
- AUPRC is computed as area above the positive-class prevalence floor using the composite trapezoidal rule without interpolation to perfect recall

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous
- Without head selection against the ground truth, scPRINT was second-best behind GENIE3 on the gwps benchmark; only with gwps-selected heads did it outperform GENIE3
- Geneformer v2 showed poor performance on the genome-wide perturb-seq benchmark
- Transformer ability to generate graphs and their explainability remain unresolved

## Evidence

- [abstract] scPRINT demonstrates superior performance in gene network inference to the state of the art: "scPRINT demonstrates superior performance in gene network inference to the state of the art"
- [methods] averaging keys/queries: [evidence span withheld — anchor status: not_found]
- [methods] head selection RidgeClassifier: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [methods] gwps cutoff: "We used a cutoff of 0.05 on these values to define the directed binary connection between genes"
- [discussion] no perfect ground truths: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [discussion] benchmarking tools: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"

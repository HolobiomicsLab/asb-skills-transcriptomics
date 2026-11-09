---
name: single-cell-count-preprocessing-for-benchmarking
description: 'Use when benchmarking gene network inference methods on raw single-cell
  count matrices: each method demands different preprocessing (e.'
when_to_use_negative:
- Input is already an inferred network or a paper-published results table — no count
  preprocessing is needed.
- 'The goal is model pre-training rather than downstream benchmarking: scPRINT''s
  own pretraining pipeline uses its own filtering (drop cells with <200 expressed
  genes, drop datasets <100 cells, <10,000 genes, or >95% cells removed) rather than
  these benchmark-specific normalizations.'
- Ground truth is ODE/SERGIO-simulated expression data rather than orthogonal ChIP-seq/perturb-seq
  networks — the article warns many GN methods are benchmarked on ODE-generated mock-up
  data instead of real ground truths.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPRINT
  role: Foundation model whose Keys/Queries attention embeddings are averaged over
    1024 cells and 5000 most variable genes to produce genome-wide gene networks
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Comparison method run via Tutorial_Attention_GRN notebook, averaging attention
    over last-layer heads and cells, dropping the cell-embedding token
  repo: https://github.com/jkobject/scGPT
- name: DeepSEM
  role: Comparison method run with default parameters after 10,000-count normalization,
    logp1 transform, and per-gene z-scoring
- name: GENIE3
  role: TF-only comparison method with 100 trees run on raw counts using only expressed
    transcription factors as inputs
- name: Geneformer
  role: Comparison method whose v2 networks are extracted as the mean of all attention
    heads per cell mapped back to full network size
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: BenGRN
  role: Benchmarking package used to compute EPR and AUPRC against gold-standard networks
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Benchmarking package (with BenGRN) for evaluating inferred gene networks
  repo: https://github.com/cantinilab/GRnnData
- name: scikit-learn
  role: RidgeClassifier (L2 penalty 1, positivity constraint on coefficients, no intercept)
    selects scPRINT attention heads trained on ground-truth edges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:26e05fd3df655293d79fc16f0c5755c0c92763c7dd885ed2b8f7ae20fac3515f
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
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# Single-cell count preprocessing for benchmarking

## Summary

Standardizes single-cell RNA-seq expression matrices (e.g., MCalla et al. and Yan E-GEOD-36552 hESC/mESC datasets) so that multiple gene network inference methods — scPRINT, scGPT, DeepSEM, Geneformer v2, GENIE3 — can be benchmarked against the same ChIP-seq/perturb-seq gold standards with method-appropriate normalization per method.

## When to use

Use when benchmarking gene network inference methods on raw single-cell count matrices: each method demands different preprocessing (e.g., DeepSEM needs 10,000-count normalization + logp1 + z-scoring, GENIE3 needs raw counts, scPRINT uses 1024 cells × 5000 most variable genes), and ground-truth edge orientation must be preserved.

## When NOT to use

- Input is already an inferred network or a paper-published results table — no count preprocessing is needed.
- The goal is model pre-training rather than downstream benchmarking: scPRINT's own pretraining pipeline uses its own filtering (drop cells with <200 expressed genes, drop datasets <100 cells, <10,000 genes, or >95% cells removed) rather than these benchmark-specific normalizations.
- Ground truth is ODE/SERGIO-simulated expression data rather than orthogonal ChIP-seq/perturb-seq networks — the article warns many GN methods are benchmarked on ODE-generated mock-up data instead of real ground truths.

## Inputs

- Raw single-cell RNA-seq count matrices (MCalla et al. hESC/mESC supplementary datasets; Yan dataset E-GEOD-36552)
- Intersection gold-standard ground-truth directed binary networks (source-to-target edge lists converted to gene-by-gene binary graphs)
- List of expressed transcription factors for TF-only GENIE3 runs

## Outputs

- Method-specific preprocessed expression matrices (normalized/logp1/z-scored for DeepSEM; raw counts for GENIE3; 1024-cell × 5000-HVG embeddings for scPRINT)
- esc_benchmark_results.csv containing AUPRC and EPR for every method-dataset combination (scPRINT variants, scGPT, DeepSEM, Geneformer, GENIE3 on two human and two mouse ESC datasets)

## How to apply

Load the ESC expression datasets (MCalla et al. supplementary data and E-GEOD-36552 from the gene expression atlas) and the intersection gold-standard directed binary networks, converting source-to-target lists into a gene-by-gene binary graph with rows as targets and columns as regulators (use the non-transposed orientation for ChIP-seq/perturb-seq ground truths). Apply method-specific preprocessing: for DeepSEM, normalize to 10,000 counts, logp1-transform, and per-gene z-score; for GENIE3, run TF-only with 100 trees on raw counts using only expressed transcription factors as inputs; for scPRINT, average Keys and Queries embeddings over 1024 cells on the 5000 most variable genes; for scGPT, follow the Tutorial_Attention_GRN notebook averaging last-layer heads and cells while dropping the cell-embedding token. The rationale is that attention-based foundation models consume embeddings over a cell sample, while regression-based methods like GENIE3 and DeepSEM operate directly on the expression matrix, so a single shared normalization would bias the comparison. Compute EPR as the diagnostic odds ratio (TP x TN) / (FP x FN) at the score cutoff yielding K = number of ground-truth positives, and AUPRC with the area floored at the positive-class prevalence and no interpolation to perfect recall, using the BenGRN/GRnnData packages.

## Related tools

- **scPRINT** (Foundation model whose Keys/Queries attention embeddings are averaged over 1024 cells and 5000 most variable genes to produce genome-wide gene networks) — https://github.com/cantinilab/scPRINT
- **scGPT** (Comparison method run via Tutorial_Attention_GRN notebook, averaging attention over last-layer heads and cells, dropping the cell-embedding token) — https://github.com/jkobject/scGPT
- **DeepSEM** (Comparison method run with default parameters after 10,000-count normalization, logp1 transform, and per-gene z-scoring)
- **GENIE3** (TF-only comparison method with 100 trees run on raw counts using only expressed transcription factors as inputs)
- **Geneformer** (Comparison method whose v2 networks are extracted as the mean of all attention heads per cell mapped back to full network size) — https://huggingface.co/ctheodoris/Geneformer
- **BenGRN** (Benchmarking package used to compute EPR and AUPRC against gold-standard networks) — https://github.com/jkobject/benGRN
- **GRnnData** (Benchmarking package (with BenGRN) for evaluating inferred gene networks) — https://github.com/cantinilab/GRnnData
- **scikit-learn** (RidgeClassifier (L2 penalty 1, positivity constraint on coefficients, no intercept) selects scPRINT attention heads trained on ground-truth edges)

## Examples

```
from sklearn.linear_model import RidgeClassifier; heads = RidgeClassifier(alpha=1, positive=True, fit_intercept=False).fit(head_scores, edge_labels)
```

## Evaluation signals

- esc_benchmark_results.csv contains both AUPRC and EPR entries for every method-dataset combination (5+ methods × 4 ESC datasets).
- EPR near 1 indicates random edge recovery; values above 1 indicate better-than-random recovery of the gold-standard edges.
- DeepSEM inputs are verifiably normalized to 10,000 counts, logp1-transformed, and per-gene z-scored, while GENIE3 receives raw counts — a single shared normalization across all methods signals incorrect application.
- Ground-truth networks respect the orientation convention that row elements are targets and column elements are regulators, using the non-transposed orientation for ChIP-seq/perturb-seq ground truths.
- AUPRC is computed with the area floored at the positive-class prevalence and no interpolation to perfect recall, matching the BenGRN implementation.

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous.
- Many GN methods are benchmarked on ODE-generated mock-up expression data rather than real ground truths, so results on such data may not transfer.
- AUPRC and EPR can diverge: scPRINT outperformed all other methods on the MCalla et al. cell type-specific benchmark on both metrics, but individual method-dataset combinations must be checked on both.

## Evidence

- [methods] DeepSEM normalization parameters: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [methods] attention network extraction: [evidence span withheld — anchor status: not_found]
- [methods] scPRINT filtering: "We also drop any cells with <200 expressed genes."
- [methods] MCalla data source: "For the MCalla et al. dataset, we downloaded the data from the supplementary datasets of their paper"
- [discussion] benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [discussion] ground truth scarcity: "ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous"

---
name: gene-network-inference-from-attention-matrices
description: Use when inferring genome-wide gene regulatory networks from a single-cell
  transformer model's attention matrices and benchmarking against ChIP-seq/perturb-seq
  gold-standard networks with EPR and AUPRC metrics.
when_to_use_negative:
- The task only requires zero-shot abilities such as denoising, batch correction,
  or cell label prediction rather than gene network extraction
- The expression data are ODE-generated mock-up (e.g., SERGIO-simulated) rather than
  real single-cell profiles with orthogonal ChIP-seq/perturb-seq ground truths
- A literature-based aggregate network like Omnipath alone is the intended target,
  since general collections like Omnipath did not improve performance in most tests
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Foundation model pre-trained on 50M+ cells whose Keys/Queries attention embeddings
    are averaged over cells to produce genome-wide gene networks
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Comparison method; attention GRN extracted by averaging last-layer heads over
    cells after dropping the cell-embedding token
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: Comparison method; networks taken as the mean of all attention heads per cell
    mapped back to full network size
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: GENIE3
  role: Comparison method; TF-only run with default parameters and 100 trees on raw
    counts using only expressed transcription factors as inputs
- name: DeepSEM
  role: Comparison method; run with default parameters after normalizing expression
    to 10,000 counts, logp1 transform, and per-gene z-scoring
- name: BenGRN
  role: Benchmarking package used to compute EPR and AUPRC against gold-standard gene
    networks
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Benchmarking package used alongside BenGRN to evaluate inferred gene networks
  repo: https://github.com/cantinilab/GRnnData
- name: scikit-learn
  role: RidgeClassifier (L2 penalty of 1, positive coefficients, no intercept) used
    to select scPRINT attention heads trained on ground-truth edges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:1dc588d1f276318dc09032ecce8a14cf642f97186f5625285dd8ee349007b2df
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# gene-network-inference-from-attention-matrices

## Summary

Extracts gene regulatory networks from transformer foundation-model attention (scPRINT, scGPT, Geneformer) by averaging Keys/Queries embeddings over cells, and benchmarks them against cell type-specific ChIP-seq/perturb-seq gold standards using EPR and AUPRC via the BenGRN/GRnnData packages.

## When to use

Use when inferring genome-wide gene regulatory networks from a single-cell transformer model's attention matrices and benchmarking against ChIP-seq/perturb-seq gold-standard networks with EPR and AUPRC metrics.

## When NOT to use

- The task only requires zero-shot abilities such as denoising, batch correction, or cell label prediction rather than gene network extraction
- The expression data are ODE-generated mock-up (e.g., SERGIO-simulated) rather than real single-cell profiles with orthogonal ChIP-seq/perturb-seq ground truths
- A literature-based aggregate network like Omnipath alone is the intended target, since general collections like Omnipath did not improve performance in most tests

## Inputs

- Single-cell ESC expression matrices (e.g., MCalla et al. human/mouse ESC datasets and Yan E-GEOD-36552 from the gene expression atlas)
- Gold-standard directed binary networks (intersection of ChIP-seq/perturb-seq ground truths, source-to-target lists converted to a gene-by-gene binary graph)
- Pre-trained scPRINT model checkpoints (available on Hugging Face at jkobject/scPRINT)
- Set of cells for attention averaging (e.g., 1024 cells over 5000 most variable genes)

## Outputs

- Genome-wide weighted gene-by-gene regulatory networks from each method (rows = targets, columns = regulators)
- Results table (e.g., esc_benchmark_results.csv) with AUPRC and EPR values for every method-dataset combination

## How to apply

Generate scPRINT networks by averaging the Keys and Queries embeddings over a set of cells (e.g., 1024 cells, 5000 most variable genes) to obtain genome-wide attention matrices; optionally include a variant where attention heads are selected with scikit-learn's RidgeClassifier (L2 penalty of 1, positivity constraint on coefficients, no intercept) trained on ground-truth edges. Build comparison networks: scGPT by averaging last-layer heads over cells (dropping the cell-embedding token, Tutorial_Attention_GRN notebook), DeepSEM after normalizing to 10,000 counts with logp1 and per-gene z-scoring, Geneformer v2 as the mean of all attention heads per cell, and TF-only GENIE3 with 100 trees on raw counts. Match inferred networks to the ground truth respecting the convention that row elements are targets and column elements are regulators (non-transposed orientation for ChIP-seq/perturb-seq ground truths). Compute EPR as the diagnostic odds ratio (TP x TN) / (FP x FN) at the cutoff yielding K positive predictions (K = number of ground-truth positives) and AUPRC with area floored at the positive-class prevalence and no interpolation to perfect recall, using BenGRN/GRnnData.

## Related tools

- **scPRINT** (Foundation model pre-trained on 50M+ cells whose Keys/Queries attention embeddings are averaged over cells to produce genome-wide gene networks) — https://github.com/cantinilab/scPRINT
- **scGPT** (Comparison method; attention GRN extracted by averaging last-layer heads over cells after dropping the cell-embedding token) — https://github.com/jkobject/scGPT
- **Geneformer** (Comparison method; networks taken as the mean of all attention heads per cell mapped back to full network size) — https://huggingface.co/ctheodoris/Geneformer
- **GENIE3** (Comparison method; TF-only run with default parameters and 100 trees on raw counts using only expressed transcription factors as inputs)
- **DeepSEM** (Comparison method; run with default parameters after normalizing expression to 10,000 counts, logp1 transform, and per-gene z-scoring)
- **BenGRN** (Benchmarking package used to compute EPR and AUPRC against gold-standard gene networks) — https://github.com/jkobject/benGRN
- **GRnnData** (Benchmarking package used alongside BenGRN to evaluate inferred gene networks) — https://github.com/cantinilab/GRnnData
- **scikit-learn** (RidgeClassifier (L2 penalty of 1, positive coefficients, no intercept) used to select scPRINT attention heads trained on ground-truth edges)

## Examples

```
from sklearn.linear_model import RidgeClassifier; head_selector = RidgeClassifier(alpha=1.0, positive=True, fit_intercept=False); head_selector.fit(attention_heads, ground_truth_edges)
```

## Evaluation signals

- Results table contains an AUPRC and EPR entry for every method-dataset combination (scPRINT variants, scGPT, DeepSEM, Geneformer, GENIE3 on two human and two mouse ESC datasets)
- EPR near 1 indicates random recovery; values above 1 indicate better-than-random recovery of gold-standard edges
- Inferred network orientation matches the ground-truth convention: rows are targets and columns are regulators (non-transposed for ChIP-seq/perturb-seq ground truths)
- AUPRC area is floored at the positive-class prevalence with no interpolation to perfect recall
- Attention networks are averaged over the specified cell set (1024 cells) and gene set (5000 most variable genes)

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous
- Attention-head selection with RidgeClassifier requires training on the ground-truth edges, so this variant is not fully zero-shot
- Orientation mistakes (transposed vs non-transposed matrices) will corrupt the benchmark since rows must be targets and columns regulators

## Evidence

- [methods] gene network extraction from averaged attention matrices: [evidence span withheld — anchor status: not_found]
- [methods] heads selection via RidgeClassifier on ground truth edges: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [discussion] BenGRN and GRnnData benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [methods] MCalla gold-standard data source: "For the MCalla et al. dataset, we downloaded the data from the supplementary datasets of their paper"
- [abstract] scPRINT scale and purpose: "we present scPRINT, a large cell model for the inference of gene networks pre-trained on more than 50 million cells from the cellxgene database"

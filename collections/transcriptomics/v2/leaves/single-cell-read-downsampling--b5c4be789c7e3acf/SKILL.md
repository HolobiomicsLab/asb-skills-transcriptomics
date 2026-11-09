---
name: single-cell-read-downsampling
description: Use when evaluating how an estimator of cell-level quantities (e.g.,
  allelic proportions) performs under reduced per-cell sequencing depth, using full-depth
  single-cell read counts to create matched low-coverage replicates.
when_to_use_negative:
- Data are already low-coverage scRNA-Seq reads — no simulation of low depth is needed
- The analysis targets transcriptome-wide quantitative expression rather than per-cell
  allelic proportions, where sub-sampling destroys count fidelity without an evaluation
  purpose
- Input counts are UMI-based allelic transcript counts (as in Larsson et al.), where
  read down-sampling does not correspond to reduced molecule counts
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBASE
  role: Downstream estimator applied to the down-sampled matrix to measure precision
    (MSE) of partial-pooled allelic proportion estimates
  repo: github.com/churchill-lab/scBASE
- name: EMASE
  role: Upstream producer of the allele-specific weighted-allocation count structure
    that down-sampling operates on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:74d5ff71172fa536dba62aa83d712febd375fa40bb4216ef57983a2bc1165ba2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - scBASE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Single-cell read down-sampling

## Summary

Randomly sub-samples a fixed fraction (e.g., 1%) of allele-specific reads independently within each single cell to synthesize low-coverage scRNA-Seq data, enabling precision evaluation (e.g., MSE of allelic proportion estimates) of methods like scBASE under reduced sequencing depth.

## When to use

Use when evaluating how an estimator of cell-level quantities (e.g., allelic proportions) performs under reduced per-cell sequencing depth, using full-depth single-cell read counts to create matched low-coverage replicates.

## When NOT to use

- Data are already low-coverage scRNA-Seq reads — no simulation of low depth is needed
- The analysis targets transcriptome-wide quantitative expression rather than per-cell allelic proportions, where sub-sampling destroys count fidelity without an evaluation purpose
- Input counts are UMI-based allelic transcript counts (as in Larsson et al.), where read down-sampling does not correspond to reduced molecule counts

## Inputs

- Full-depth allele-specific read count matrix (maternal/paternal counts per gene per cell) for 122 blastocyst cells from the Deng et al. CAST×B6 F1 hybrid scRNA-Seq dataset
- Per-cell sampling fraction (1%)
- Cell metadata (early, mid, late blastocyst stage labels)

## Outputs

- Down-sampled allele-specific count matrix for all 122 cells at ~1% of original depth (~147,538 reads/cell average)
- Table file (count matrix) for use as input to the partial-pooling MSE evaluation

## How to apply

Load the full-depth allele-specific read count matrix (maternal/paternal counts per gene per cell) — e.g., EMASE weighted-allocation counts from the Deng et al. CAST×B6 F1 scRNA-Seq dataset. Independently within each cell (here 122 cells spanning early, mid, and late blastocyst stages), randomly retain a fixed fraction of reads — 1%, yielding on average ~147,538 reads per cell — while preserving the allele-specific count structure per gene. Write the down-sampled count matrix for all cells to a table file suitable as input to downstream estimation (e.g., scBASE partial-pooling MSE evaluation). The rationale is to compare estimates against the full-depth gold standard at matched low coverage; note the article does not specify a randomization seed, so a fixed seed should be set for reproducibility.

## Related tools

- **scBASE** (Downstream estimator applied to the down-sampled matrix to measure precision (MSE) of partial-pooled allelic proportion estimates) — github.com/churchill-lab/scBASE
- **EMASE** (Upstream producer of the allele-specific weighted-allocation count structure that down-sampling operates on)

## Evaluation signals

- Output count matrix contains exactly all 122 cells (early, mid, late blastocyst stages)
- Per-cell down-sampled read totals equal approximately 1% of the corresponding full-depth cell totals (~147,538 reads/cell on average)
- Maternal/paternal per-gene count structure is preserved (no loss of gene-level allele-specific labels during sub-sampling)
- Full-depth counts for each cell/gene are recoverable as the reference gold standard for MSE comparison
- A fixed random seed is recorded so the 1% sampling is reproducible

## Limitations

- Down-sampling seed/randomization procedure is not specified in the article, limiting exact reproducibility
- Down-sampling mimics reduced depth but not other low-coverage artifacts (e.g., alignment changes at lower depth)
- Precision gains evaluated this way depend on cell count: partial pooling has the biggest impact when read coverage is low and the number of cells is large

## Evidence

- [discussion] 1% sampling fraction within each cell of the 122 blastocyst cells: "We randomly sampled 1% of reads in each of 122 cells at the early, mid, and late blastocyst stages"
- [discussion] Rationale: partial pooling benefits most at low coverage with many cells: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [readme] Motivation from README: single-cell ASE suffers from extremely low per-cell sequencing depth: "analyzing single-cell ASE poses unique analytical challenges due to the extremely low depth of sequencing coverage per cell"

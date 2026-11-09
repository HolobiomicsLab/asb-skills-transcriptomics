---
name: sequencing-depth-variance-simulation
description: Use when evaluating how an estimator (e.g., allelic proportion or expression
  estimates) performs at reduced sequencing depth, by simulating low-coverage data
  via random per-cell read down-sampling from full-depth counts.
when_to_use_negative:
- The input data is already low-coverage — down-sampling would discard information
  without serving an evaluation purpose
- The goal is differential-expression or burst-kinetics testing rather than estimation-precision
  (MSE) evaluation of allelic proportions
- UMI-based counts are the target (e.g., Larsson et al. data), since down-sampling
  reads rather than molecules distorts UMI count semantics
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: scBASE
  role: Downstream consumer of the down-sampled count matrix for the partial-pooling
    MSE evaluation of allelic proportion estimates
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: Upstream producer of the allele-specific counts (EM-based weighted allocation
    of multi-reads) that are down-sampled
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:86fbcb1d8a5655457dad2d34ede5f3ed730c1e3e07edb3a240ea752caf04b4a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - scBASE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sequencing-depth-variance-simulation

## Summary

Generates low-coverage scRNA-Seq data by random down-sampling of full-depth allele-specific read counts (1% of reads per cell) so that estimator precision — e.g., MSE of allelic proportions under partial pooling — can be evaluated against known full-depth values, as done in scBASE for 122 CAST×B6 blastocyst cells (~147,538 reads/cell).

## When to use

Use when evaluating how an estimator (e.g., allelic proportion or expression estimates) performs at reduced sequencing depth, by simulating low-coverage data via random per-cell read down-sampling from full-depth counts.

## When NOT to use

- The input data is already low-coverage — down-sampling would discard information without serving an evaluation purpose
- The goal is differential-expression or burst-kinetics testing rather than estimation-precision (MSE) evaluation of allelic proportions
- UMI-based counts are the target (e.g., Larsson et al. data), since down-sampling reads rather than molecules distorts UMI count semantics

## Inputs

- Full-depth allele-specific read count matrix (maternal/paternal reads per gene per cell) for the 122 blastocyst cells from Deng et al. CAST×B6 F1 scRNA-Seq data, produced by EMASE weighted-allocation counting
- Sampling fraction (1%) specifying the target low-coverage depth

## Outputs

- Down-sampled allele-specific count matrix for all 122 cells (~147,538 reads/cell average) written to a table file
- Low-coverage dataset used as input for mean-squared-error evaluation of partial-pooling allelic proportion estimates

## How to apply

Load the full-depth allele-specific read count matrix (maternal/paternal counts per gene per cell, produced upstream by EMASE weighted-allocation counting) for the cells of interest — here, 122 early/mid/late blastocyst cells from the Deng et al. CAST×B6 F1 hybrid dataset. Independently within each cell, randomly sample 1% of reads to mimic low sequencing depth, yielding ~147,538 reads per cell on average, while preserving the allele-specific count structure. Write the down-sampled count matrix for all cells to a table file for downstream use (e.g., as scBASE input for a partial-pooling MSE evaluation). Validate that all 122 cells are present and that each cell's read total equals approximately 1% of its full-depth total. Note the article does not specify a random seed or randomization procedure, so record seed choices for reproducibility.

## Related tools

- **scBASE** (Downstream consumer of the down-sampled count matrix for the partial-pooling MSE evaluation of allelic proportion estimates) — https://github.com/kbchoi-jax/scbase
- **EMASE** (Upstream producer of the allele-specific counts (EM-based weighted allocation of multi-reads) that are down-sampled)

## Examples

```
python downsample_allelic_counts.py --input full_depth_counts.tsv --cells 122 --fraction 0.01 --output downsampled_counts.tsv
```

## Evaluation signals

- Output contains all 122 cells (early, mid, and late blastocyst stages)
- Per-cell read totals in the down-sampled matrix equal approximately 1% of the corresponding full-depth counts
- Average coverage across cells is approximately 147,538 reads per cell
- Maternal/paternal allele-specific count structure (per gene per cell) is preserved from the EMASE weighted-allocation counting step
- Down-sampled estimates are compared against full-depth values via mean squared error to quantify precision loss

## Limitations

- Down-sampling seed/randomization procedure for sampling 1% of reads is not specified in the article, limiting reproducibility
- Partial pooling has the biggest impact when read coverage is low and the number of cells is large; at high depth the benefit diminishes
- Evaluation is tied to the Deng et al. CAST×B6 blastocyst data; generalization to other tissues or UMI-based protocols is not established

## Evidence

- [discussion] Down-sampling of reads (1% in 122 blastocyst cells) to evaluate estimation precision via mean squared error: "We randomly sampled 1% of reads in each of 122 cells at the early, mid, and late blastocyst stages"
- [discussion] Partial pooling impact depends on coverage and cell number: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [discussion] Missing randomization details for down-sampling: [evidence span withheld — anchor status: not_found]
- [readme] scBASE handles low sequencing depth per cell: "analyzing single-cell ASE poses unique analytical challenges due to the extremely low depth of sequencing coverage per cell"

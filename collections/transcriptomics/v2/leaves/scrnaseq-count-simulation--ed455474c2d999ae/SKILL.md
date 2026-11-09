---
name: scrnaseq-count-simulation
description: Use when benchmarking a cell-cycle or latent-factor removal method on
  scRNA-Seq data where ground-truth effect assignments (which PCs are cell-cycle,
  which genes are cell-type) must be known exactly.
when_to_use_negative:
- The goal is to remove a cell-cycle effect from a real scRNA-Seq dataset (e.g. T
  helper, mESC, glioblastoma data) rather than to benchmark a removal method against
  known ground truth
- The confounder of interest is not cell-cycle and cell-type structure (e.g. batch
  or technical replicates), since the planted effect model covers only these two components
- Cell-cycle activity is very low in the target setting (e.g. glioblastoma, where
  the paper uses a smaller cutoff C_delta = 2), so the default simulation and cutoff=3
  parameters would not reflect the analysis conditions
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: 'Benchmarked correction method: PCA on control genes, two-class bootstrap
    t-test on loading difference Delta_j with cutoff C_delta=3, subtraction of flagged
    cell-cycle directions'
  repo: cran/ccRemover
- name: scLVM
  role: Comparison latent-factor method run via its Python script with K=1 (default)
    and K=3 (scree plot) to demonstrate failure modes on the simulated data
- name: Python
  role: Used to run scLVM on the simulated matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:ba6c046966a2ea84be1bcec3f40f3671038b69ca91f9e9ddb44b34c3017c54f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Python
  - ccRemover
  - scLVM
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# scrnaseq-count-simulation

## Summary

Generates a synthetic scRNA-Seq matrix X with a planted cell-cycle effect on annotated genes and a separate cell-type effect (50 cells x 2,000 genes, 400 cell-cycle genes, six cell-type-by-cell-cycle-stage groups), enabling controlled benchmarking of correction methods such as ccRemover and scLVM against known ground truth.

## When to use

Use when benchmarking a cell-cycle or latent-factor removal method on scRNA-Seq data where ground-truth effect assignments (which PCs are cell-cycle, which genes are cell-type) must be known exactly.

## When NOT to use

- The goal is to remove a cell-cycle effect from a real scRNA-Seq dataset (e.g. T helper, mESC, glioblastoma data) rather than to benchmark a removal method against known ground truth
- The confounder of interest is not cell-cycle and cell-type structure (e.g. batch or technical replicates), since the planted effect model covers only these two components
- Cell-cycle activity is very low in the target setting (e.g. glioblastoma, where the paper uses a smaller cutoff C_delta = 2), so the default simulation and cutoff=3 parameters would not reflect the analysis conditions

## Inputs

- Simulation specification per the paper's model: 50 cells, 2,000 genes, 400 annotated cell-cycle genes, cell-cycle effect loaded on set A, cell-type effect producing six cell-type-by-cell-cycle-stage groups
- if_cc indicator vector splitting genes into cell-cycle set A and control set B
- Candidate correction methods to benchmark (ccRemover; scLVM via its Python script)

## Outputs

- Simulated log-transformed, gene-centered scRNA-Seq matrix X
- Clusterings of the original, ccRemover-corrected, and scLVM-corrected matrices
- simulation_correction_comparison.csv recording which method removed which effect

## How to apply

Generate the simulated matrix X per the paper's model: measurements for 50 cells and 2,000 genes, of which 400 genes are assigned as cell-cycle genes carrying the cell-cycle effect, plus a separate cell-type effect so that cells form six cell-type-by-cell-cycle-stage groups. Log-transform the simulated data and center it gene-by-gene, then split genes into the cell-cycle set A and control set B using the if_cc indicator vector. Validate the ground truth by clustering the original matrix (it must yield six groups by cell type and cell-cycle stage combination), then run candidate correction methods — e.g. ccRemover with cutoff C_delta = 3, max_it = 4, ntop = 10, nboot 100–200 — and scLVM with both the default K = 1 and the scree-plot-derived K = 3. Judge a correction method against the planted truth: ccRemover must flag the 2nd and 3rd principal components as cell-cycle effects, while scLVM's failure modes (K=1 removes the cell-type effect; K=3 removes both effects leaving only noise) demonstrate why the formal bootstrap t-test on Delta_j matters.

## Related tools

- **ccRemover** (Benchmarked correction method: PCA on control genes, two-class bootstrap t-test on loading difference Delta_j with cutoff C_delta=3, subtraction of flagged cell-cycle directions) — cran/ccRemover
- **scLVM** (Comparison latent-factor method run via its Python script with K=1 (default) and K=3 (scree plot) to demonstrate failure modes on the simulated data)
- **Python** (Used to run scLVM on the simulated matrix)

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Original simulated data clusters into six groups corresponding to cell type and cell-cycle stage combinations
- ccRemover identifies the 2nd and 3rd principal components as cell-cycle effects on the simulated data
- After ccRemover correction, cells cluster by cell type with different cell-cycle stages mixed within clusters
- scLVM with K=1 mistakenly removes the cell-type effect (cells cluster by cell-cycle stage); with K=3 it removes both effects, leaving only noise
- Outcomes for original, ccRemover-corrected, and scLVM-corrected matrices are recorded in simulation_correction_comparison.csv

## Limitations

- Annotation databases for cell-cycle genes are always incomplete and inaccurate; the simulation assumes a known, correct if_cc annotation
- If a cell-cycle effect influences too few control genes (below roughly 8% affected), it may not be captured by the principal components and hence missed for detection
- Supplementary sections S2 and S3 (simulation details on sparsity robustness and annotation incompleteness) are referenced but not included in the provided text
- No version numbers are reported for scLVM, ccRemover, or the R annotation packages, and no explicit GitHub URL for scLVM is given

## Evidence

- [results] Simulation model dimensions and planted effects: "We simulate data matrix X that contains measurements for 50 cells and 2,000 genes, of these genes 400 are assigned as cell-cycle genes"
- [results] Ground-truth clustering of original simulated data: "the cells are clustered into six different groups corresponding to the cell type and cell-cycle stage combinations"
- [results] ccRemover validation criterion on simulated data: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [results] scLVM failure mode that motivates the simulation benchmark: "scLVM has failed completely here by mistakenly removing the cell-type effect instead of the cell-cycle effect"
- [methods] Preprocessing of the simulated matrix before testing: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [methods] Concrete ccRemover validation call parameters: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"

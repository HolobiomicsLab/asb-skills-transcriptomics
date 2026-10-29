---
name: pseudotime-trajectory-inference
description: Use when dropout-affected scRNA-seq data has been imputed (e.g., with
  SERM vs. MAGIC) and the goal is to verify that developmental trajectories and state
  transitions are recovered, not just gene values.
when_to_use_negative:
- Input matrices have no ground-truth stage labels or known developmental ordering,
  so pseudotime recovery cannot be validated against reference trajectory structure
- Small genomic datasets where SERM's estimated distribution may diverge from the
  actual data distribution, leading to inaccurate imputation values and misleading
  trajectories
- Batch-effect-critical settings, since batch effects are not considered explicitly
  in the SERM formulation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Upstream imputation of dropout-affected matrices whose trajectories are then
    evaluated with TSCAN/Slingshot
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: TSCAN
  role: Pseudotime inference on reference, observed, and imputed matrices using the
    authors' default settings
- name: Slingshot
  role: Pseudotime inference on reference, observed, and imputed matrices using the
    authors' default settings
- name: PHATE
  role: 2D embedding visualization (2 dims, 5 NN, alpha=40, metric MDS, Euclidean
    distance, gamma=0.5) of reference, observed, and imputed trajectories
  repo: https://github.com/pkathail/magic
- name: MAGIC
  role: Competing imputation method from its official repository, run with default
    configurations for trajectory-recovery comparison
  repo: https://github.com/pkathail/magic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:c4428be5560f69c91871b5f7045eed7b2c08c75459657069727eaac55d8c5fa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - MAGIC
  - PHATE
  - Slingshot
  - TSCAN
  - self-consistent expression recovery machine (SERM)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pseudotime-trajectory-inference

## Summary

Apply TSCAN and Slingshot pseudotime inference to reference, dropout-affected observed, and imputed scRNA-seq matrices (zebrafish development, EB differentiation), then visualize with PHATE to test whether SERM-based imputation preserves continuous developmental trajectories and state transitions without manifold distortion.

## When to use

Use when dropout-affected scRNA-seq data has been imputed (e.g., with SERM vs. MAGIC) and the goal is to verify that developmental trajectories and state transitions are recovered, not just gene values.

## When NOT to use

- Input matrices have no ground-truth stage labels or known developmental ordering, so pseudotime recovery cannot be validated against reference trajectory structure
- Small genomic datasets where SERM's estimated distribution may diverge from the actual data distribution, leading to inaccurate imputation values and misleading trajectories
- Batch-effect-critical settings, since batch effects are not considered explicitly in the SERM formulation

## Inputs

- Reference (dropout-free) gene expression matrix (cells x genes) for zebrafish development and EB differentiation datasets
- Observed dropout-affected matrices at multiple sampling efficiencies
- Imputed matrices from SERM and competing methods (MAGIC, etc.)
- Ground-truth stage labels (e.g., 12 early zebrafish development stages)

## Outputs

- Pseudotime orderings per cell from TSCAN and Slingshot for reference, observed, and SERM-imputed matrices
- Pearson correlation between monocle pseudotime and ground-truth stage labels per method and sampling efficiency
- Reference-to-imputed Pearson coefficients for zebrafish and EB datasets
- 2D PHATE embeddings (ndim=2, k=5, alpha=40, metric MDS, Euclidean distance, gamma=0.5; 100 PCs zebrafish / 50 PCs EB)

## How to apply

Download the reference and observed (dropout-affected) matrices for the zebrafish (38,731 cells, 694 embryos, 12 stages) and EB differentiation datasets from the paper's Google Drive deposit, and impute the observed matrices with SERM (default adaptive histogram-equalization settings) plus a competing method such as MAGIC for contrast. Run TSCAN and Slingshot on the reference, observed, and imputed matrices with the authors' default settings, and compute the Pearson correlation between inferred pseudotime (monocle) and ground-truth stage labels for each condition and sampling efficiency. Visualize the reference, observed, and imputed data with PHATE using 2 embedding dimensions, 5 nearest neighbors, alpha = 40, metric MDS, Euclidean distance, gamma = 0.5, with 100 principal components for the zebrafish data and 50 PCs for the EB data. The rationale is that faithful imputation should reproduce the reference's continuous trajectory ordering; a correct run shows SERM-imputed pseudotime correlating better with stage labels than all competing methods (higher reference-to-imputed Pearson coefficients for both datasets), while competing methods visibly distort the manifold.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Upstream imputation of dropout-affected matrices whose trajectories are then evaluated with TSCAN/Slingshot) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **TSCAN** (Pseudotime inference on reference, observed, and imputed matrices using the authors' default settings)
- **Slingshot** (Pseudotime inference on reference, observed, and imputed matrices using the authors' default settings)
- **PHATE** (2D embedding visualization (2 dims, 5 NN, alpha=40, metric MDS, Euclidean distance, gamma=0.5) of reference, observed, and imputed trajectories) — https://github.com/pkathail/magic
- **MAGIC** (Competing imputation method from its official repository, run with default configurations for trajectory-recovery comparison) — https://github.com/pkathail/magic

## Examples

```
phateData <- phate(data_imputed, ndim=2, k=5, alpha=40, gamma=0.5, mds.method="metric", distance="euclidean")
```

## Evaluation signals

- SERM yields consistently higher Pearson correlation between monocle pseudotime and ground-truth stage labels than all competing imputation methods across sampling efficiencies on the zebrafish dataset
- SERM achieves the highest reference-to-imputed Pearson coefficients for both zebrafish and EB differentiation datasets
- PHATE visualization of SERM-imputed data reproduces the continuous developmental trajectory of Fig. 5 without distortion, whereas competing methods visibly distort the manifold
- Embedding parameters match the specified settings: 2 dimensions, 5 nearest neighbors, alpha = 40, metric MDS, Euclidean distance, gamma = 0.5, 100 PCs (zebrafish) / 50 PCs (EB)
- Pseudotime is computed on all three conditions (reference, observed, imputed) so the imputed ordering can be compared directly to the reference ordering

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so on small genomic datasets the estimated distribution may diverge from the actual distribution, producing inaccurate imputation values that would propagate into pseudotime
- Reference data for evaluation was created by choosing only cells and genes with high expressions, and dropouts were simulated rather than observed experimentally
- SERM performance assumes the ROI coverage is sufficiently large that the captured ROI distribution is representative of the entire matrix

## Evidence

- [results] Zebrafish dataset scale: "profiling 38,731 cells from 694 embryos across 12 stages of early zebrafish development"
- [results] EB dataset definition: "scRNA-seq data of human Embryonic Stem (ES) cells differentiated to embryoid bodies (EBs)"
- [readme] MAGIC as denoising comparator: [evidence span withheld — anchor status: external_doc]

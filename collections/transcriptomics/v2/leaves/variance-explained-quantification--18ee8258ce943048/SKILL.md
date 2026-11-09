---
name: variance-explained-quantification
description: Use when a JIVE (or similar joint/individual) decomposition has been
  fitted and you need the proportion of variation explained by joint, individual,
  and residual components — e.g., to verify two implementations yield equivalent decompositions.
when_to_use_negative:
- The input data have not been decomposed by JIVE — without joint and individual low-rank
  estimates there are no components whose explained variation can be partitioned
- The data are single-source with no batch/individual structure, so the joint vs.
  individual split that this skill quantifies is not meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: r.jive
  role: Original JIVE implementation; its showVarExplained function displays a barchart
    of variation explained, and its jive output is used to compute baseline variance
    proportions
- name: JIVE (scJIVE)
  role: Enhanced JIVE implementation whose joint/individual/residual variance proportions
    are computed and compared against r.jive to confirm equivalence
  repo: https://github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:4d8dab16c66b60d9679d9e9e7a592e88542db887c0376091c3e3696b7eeb79a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - JIVE (scJIVE)
  - R
  - r.jive
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# variance-explained-quantification

## Summary

Quantifies and visualizes how much total variation in a multi-source matrix decomposition (e.g., JIVE) is attributed to the joint structure, each individual (batch-specific) structure, and residual error. In this paper, the proportions are compared between the original R.JIVE implementation and the faster scJIVE implementation to confirm the decomposition is preserved.

## When to use

Use when a JIVE (or similar joint/individual) decomposition has been fitted and you need the proportion of variation explained by joint, individual, and residual components — e.g., to verify two implementations yield equivalent decompositions.

## When NOT to use

- The input data have not been decomposed by JIVE — without joint and individual low-rank estimates there are no components whose explained variation can be partitioned
- The data are single-source with no batch/individual structure, so the joint vs. individual split that this skill quantifies is not meaningful

## Inputs

- Multi-source numeric data matrices (e.g., the 200x1000 SimData pair with rank-1 joint and rank-1 individual structures, or BRCA_data with gene expression, methylation, and miRNA matrices)
- A fitted JIVE output object (returned by r.jive::jive or the scJIVE implementation)

## Outputs

- Barchart of the amount of variation explained by joint, individual, and residual components (showVarExplained)
- Summary table of variance proportions (e.g., variance_proportions.csv) comparing joint/individual/residual proportions across implementations

## How to apply

Fit the JIVE decomposition on the multi-source matrices (e.g., r.jive::jive or the scJIVE package), with each data matrix scaled by its total variation and row-orthogonality enforced between joint and individual estimates. Then use r.jive's showVarExplained function to display a barchart of the amount of variation explained by each component, and tabulate the joint/individual/residual proportions to a comparison file (e.g., variance_proportions.csv). When benchmarking implementations, compute the proportions under both the original r.jive::jive and scJIVE on identical inputs (e.g., the 200x1000 rank-1 joint / rank-1 individual SimData pair) and treat the benchmark as successful if the proportions are near-identical between implementations. As a sanity reference on the BRCA multi-omics data, JIVE estimated a final joint rank of 2 and individual ranks of 27 (gene expression), 26 (methylation), and 25 (miRNA).

## Related tools

- **r.jive** (Original JIVE implementation; its showVarExplained function displays a barchart of variation explained, and its jive output is used to compute baseline variance proportions)
- **JIVE (scJIVE)** (Enhanced JIVE implementation whose joint/individual/residual variance proportions are computed and compared against r.jive to confirm equivalence) — https://github.com/oconnell-statistics-lab/scJIVE

## Examples

```
library(r.jive); data(BRCA_data); res <- jive(BRCA_data); showVarExplained(res)
```

## Evaluation signals

- Joint + individual + residual variance proportions account for the total variation, with row-orthogonality enforced between joint and individual estimates
- Proportions of variation explained are near-identical between r.jive and scJIVE on the same input data (the task's validation criterion)
- On BRCA_data, the reported component structure matches the published ranks: joint rank 2, individual ranks 27 (gene expression), 26 (methylation), 25 (miRNA)
- A barchart of the amount of variation explained is produced via showVarExplained for visual inspection
- The proportion table is written with one row/entry per component (joint, individual, residual) per implementation

## Limitations

- JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets (~35.8 s vs ~4.1 s per run in the benchmark), so variance quantification on large scRNA-seq data should use scJIVE
- The interpretation of proportions depends on the estimated ranks; scJIVE's decomposition aims to separate true biological variability (joint) from technical batch variability (individual), so misestimated ranks distort the attributed proportions

## Evidence

- [methods] showVarExplained barchart: "The most simple function is `showVarExplained`, which displays a barchart of the amount of variation explained"
- [methods] scaling and orthogonality conventions: "each data matrix is scaled by its total variation"
- [readme] joint/individual semantics: "scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch."
- [other] benchmark equivalence finding: [evidence span withheld — anchor status: not_found]

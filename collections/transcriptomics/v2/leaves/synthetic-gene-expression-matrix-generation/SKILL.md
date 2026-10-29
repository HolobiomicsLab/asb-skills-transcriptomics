---
name: synthetic-gene-expression-matrix-generation
description: Use when a biclustering benchmark requires controlled synthetic test
  data with known implanted bicluster positions, e.g. Types I-VI (trend-preserving,
  column-constant, row-constant, shift-scale, shift, scale) planted in N(0,1) backgrounds.
when_to_use_negative:
- The goal is real-data analysis (e.g. BicatYeast, airway, GDS589) rather than controlled
  benchmark generation with known ground truth
- The downstream algorithm requires discrete/binarized input, not continuous N(0,1)
  expression values
- Only one noise-free planted pattern is needed without a background distribution
  or replicate structure
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language for seeded, reproducible generation of N(0,1) background matrices
    and implantation of Type I-VI biclusters
- name: runibic
  role: 'Downstream consumer: parallel UniBic implementation whose bicluster recovery
    is evaluated on the generated synthetic matrices'
  repo: https://github.com/athril/runibic
- name: UniBic
  role: Reference biclustering algorithm the synthetic benchmark was designed to evaluate
  repo: http://sourceforge.net/projects/unibic/files/?source=navbar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:9165fe8fc1939790fdb4d83cbe79ee80c75014ba731afa4659c8927a9b812f58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/athril/runibic
  tools:
  - R
  - UniBic
  - runibic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# synthetic-gene-expression-matrix-generation

## Summary

Generates reproducible synthetic gene expression matrices with implanted trend-preserving biclusters (Types I-VI) in N(0,1) backgrounds, providing labeled benchmarks for evaluating biclustering algorithms such as UniBic against ground truth.

## When to use

Use when a biclustering benchmark requires controlled synthetic test data with known implanted bicluster positions, e.g. Types I-VI (trend-preserving, column-constant, row-constant, shift-scale, shift, scale) planted in N(0,1) backgrounds.

## When NOT to use

- The goal is real-data analysis (e.g. BicatYeast, airway, GDS589) rather than controlled benchmark generation with known ground truth
- The downstream algorithm requires discrete/binarized input, not continuous N(0,1) expression values
- Only one noise-free planted pattern is needed without a background distribution or replicate structure

## Inputs

- Size-class specifications (150x100, 200x150, 300x200) with bicluster counts (3, 4, 5)
- Bicluster type definitions: Type I trend-preserving, Type II column-constant, Type III row-constant, Type IV shift-scale, Type V shift, Type VI scale
- Overlap-variant specification: 20x20 implanted biclusters on a 200x150 background with 0x0/3x3/6x6/9x9 overlaps
- Fixed random seeds for five independent replicates per size class

## Outputs

- 15 size-class replicate matrices (CSV) in synthetic_datasets/
- 4 overlapping-size variant matrices (CSV)
- One ground-truth bicluster row/column coordinate file per matrix

## How to apply

Fix a random seed in R for reproducibility, then draw background matrices from the standard normal distribution N(0,1) via rnorm() for each size class (150x100, 200x150, 300x200). Implant trend-preserving biclusters following the Type I-VI pattern rules — 3 biclusters in the 150x100 matrix, 4 in the 200x150, and 5 in the 300x200 — recording the ground-truth row/column coordinates of each implant. Generate five independent replicate seeds per size class, and separately build four overlapping variants with 20x20 biclusters on a 200x150 N(0,1) background using overlap sizes 0x0, 3x3, 6x6, and 9x9. Write every matrix and its ground-truth coordinate file as individual CSVs into synthetic_datasets/, since these labels are what downstream benchmark evaluation (e.g. against UniBic's LCS-based seed detection) consumes to score recovery of trend-preserving biclusters.

## Related tools

- **R** (Language for seeded, reproducible generation of N(0,1) background matrices and implantation of Type I-VI biclusters)
- **runibic** (Downstream consumer: parallel UniBic implementation whose bicluster recovery is evaluated on the generated synthetic matrices) — https://github.com/athril/runibic
- **UniBic** (Reference biclustering algorithm the synthetic benchmark was designed to evaluate) — http://sourceforge.net/projects/unibic/files/?source=navbar

## Examples

```
set.seed(42); bg <- matrix(rnorm(150*100), nrow=150)  # N(0,1) background for Type I-VI implantation
```

## Evaluation signals

- Complete file set present: 15 size-class replicate matrices + 4 overlap variants, each paired with a ground-truth coordinate file
- Each implanted region satisfies its declared Type I-VI trend pattern (e.g. column-constant or shift/scale relations hold within tolerance)
- Re-running with the same fixed seed reproduces byte-identical matrices (rnorm determinism)
- Background entries outside implants follow N(0,1) (mean ~0, variance ~1)
- Overlap variants show exactly the specified 0x0/3x3/6x6/9x9 row-column overlaps between 20x20 biclusters

## Limitations

- The paper's discussion section provides no named datasets, parameters, or implementation details for its synthetic benchmark — the Type I-VI implantation rules must be supplied externally by the task card
- Identification of general trend-preserving biclusters remains a challenging problem, so implantation patterns may not cover all bicluster forms a real analysis encounters
- UniBic outperformed competitors except BicSPAM on narrow biclusters, so narrow implanted biclusters (e.g. the 20x20 overlap variants) may bias recovery comparisons

## Evidence

- [abstract] trend-preserving biclusters from noisy data: "We found an elementary method by which biologically meaningful trend-preserving biclusters can be readily identified from noisy and complex large data."
- [discussion] synthetic and real data results: "The results on both synthetic and real data sets demonstrate that UniBic is more promising in discovery of functionally correlated expression patterns in gene expression data"
- [readme] README on trend-preserving targets: "The algorithm tries to locate trend-preserving biclusters within complex and noisy data."
- [readme] README runibic install: "devtools::install_github("athril/runibic")"

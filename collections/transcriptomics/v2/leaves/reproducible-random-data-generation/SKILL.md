---
name: reproducible-random-data-generation
description: Use when benchmarking biclustering algorithms on synthetic data with
  known ground truth — e.g. implanting trend-preserving, shift-scale, or constant
  biclusters into random N(0,1) matrices with fixed seeds for reproducibility.
when_to_use_negative:
- When the goal is biclustering real gene expression data (e.g. BicatYeast, airway,
  GDS589) rather than benchmarking on controlled synthetic ground truth
- When the downstream algorithm targets non-trend-preserving structures (e.g. additive/multiplicative
  models) not representable by the Type I-VI implant patterns
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language used to draw N(0,1) backgrounds with rnorm() under fixed seeds and
    write matrices plus ground-truth coordinates as CSV
- name: runibic
  role: Downstream parallel UniBic implementation in R that the generated synthetic
    benchmark matrices are used to evaluate
  repo: https://github.com/athril/runibic
- name: UniBic
  role: Original sequential biclustering algorithm whose performance on the synthetic
    datasets motivates the benchmark design
  repo: http://sourceforge.net/projects/unibic/files/?source=navbar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:3e5a8697cecdd5c7b8cf0d75d60758bba29d8d5c43e752b8bc0c7e54c794f7b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/athril/runibic
  tools:
  - R
  - UniBic
  - runibic
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reproducible-random-data-generation

## Summary

Generate reproducible synthetic benchmark matrices by drawing N(0,1) backgrounds in R with fixed seeds and implanting ground-truth trend-preserving biclusters (Types I-VI), enabling controlled evaluation of biclustering algorithms such as UniBic.

## When to use

Use when benchmarking biclustering algorithms on synthetic data with known ground truth — e.g. implanting trend-preserving, shift-scale, or constant biclusters into random N(0,1) matrices with fixed seeds for reproducibility.

## When NOT to use

- When the goal is biclustering real gene expression data (e.g. BicatYeast, airway, GDS589) rather than benchmarking on controlled synthetic ground truth
- When the downstream algorithm targets non-trend-preserving structures (e.g. additive/multiplicative models) not representable by the Type I-VI implant patterns

## Inputs

- Random seed specification (integer) for each replicate
- Matrix size classes (150x100, 200x150, 300x200)
- Implanted bicluster counts per size class (3, 4, 5)
- Type I-VI bicluster pattern definitions
- Overlap variant specification (20x20 biclusters on 200x150 background; 0x0/3x3/6x6/9x9 overlaps)

## Outputs

- 15 size-class replicate matrices (3 sizes x 5 seeds) as CSV files in synthetic_datasets/
- 4 overlapping-size variant matrices as CSV files
- Ground-truth bicluster row/column coordinate files for every matrix

## How to apply

In R, set a fixed random seed so all matrices are reproducible, then draw background matrices from the standard normal distribution N(0,1) using rnorm() for each size class (e.g. 150x100, 200x150, 300x200). Implant biclusters following the six Type I-VI pattern rules — type I trend-preserving, type II column-constant, type III row-constant, type IV shift-scale, type V shift, type VI scale — recording ground-truth row/column coordinates; the UniBic evaluation used 3 implanted biclusters in the 150x100 matrix, 4 in the 200x150, and 5 in the 300x200, each repeated over five independent replicate seeds. Separately build overlap-stress variants by implanting 20x20 biclusters on a 200x150 background with 0x0, 3x3, 6x6, and 9x9 overlaps. Write each matrix and its ground-truth coordinate file as CSV into a synthetic_datasets/ directory so downstream evaluation consumes a verified, complete file set.

## Related tools

- **R** (Language used to draw N(0,1) backgrounds with rnorm() under fixed seeds and write matrices plus ground-truth coordinates as CSV)
- **runibic** (Downstream parallel UniBic implementation in R that the generated synthetic benchmark matrices are used to evaluate) — https://github.com/athril/runibic
- **UniBic** (Original sequential biclustering algorithm whose performance on the synthetic datasets motivates the benchmark design) — http://sourceforge.net/projects/unibic/files/?source=navbar

## Examples

```
set.seed(42); M <- matrix(rnorm(150*100), nrow=150); # implant Type I-VI biclusters, then write.csv(M, 'synthetic_datasets/150x100_seed42.csv')
```

## Evaluation signals

- Complete file set present: 15 size-class replicate matrices + 4 overlap variants, each with a ground-truth coordinate file
- Each implanted region satisfies its declared Type I-VI trend pattern (e.g. column-constant, row-constant, shift, scale, shift-scale, or trend-preserving)
- Re-running generation with the same seed reproduces byte-identical matrices (fixed-seed reproducibility)
- Implant counts match the design: 3 biclusters in 150x100, 4 in 200x150, 5 in 300x200; overlap variants use 20x20 biclusters with 0x0/3x3/6x6/9x9 overlaps
- Background entries are drawn from N(0,1); implanted coordinates exactly match the recorded ground-truth files

## Limitations

- The six implant types cover specific trend-preserving/constant/shift-scale patterns only — identification of general trend-preserving biclusters remains a challenging problem
- UniBic's synthetic evaluation lacks named datasets/accessions and version/parameter details in some sections, so exact reproduction may require consulting the SourceForge release of the original code and data
- Fixed-seed reproducibility holds only if the same R version and RNG stream are used; different RNG implementations break byte-identical regeneration

## Evidence

- [results] synthetic datasets implant six bicluster types: [evidence span withheld — anchor status: not_found]
- [discussion] UniBic synthetic evaluation rationale: "The results on both synthetic and real data sets demonstrate that UniBic is more promising in discovery of functionally correlated expression patterns in gene expression data"
- [readme] runibic targets trend-preserving biclusters in noisy data: "This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016]
The algorithm tries to locate trend-preserving biclusters within complex and noisy data."
- [methods] UniBic source and datasets availability: "The source code as well as all datasets and results are available at: http://sourceforge.net/projects/unibic/files/?source=navbar."
- [abstract] trend-preserving biclusters from noisy data: "We found an elementary method by which biologically meaningful trend-preserving biclusters can be readily identified from noisy and complex large data."

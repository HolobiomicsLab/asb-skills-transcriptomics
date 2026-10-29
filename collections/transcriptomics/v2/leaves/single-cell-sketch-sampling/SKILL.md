---
name: single-cell-sketch-sampling
description: Use when a scRNA-seq expression matrix is too large for downstream tools
  (clustering, trajectory inference) and an evenly covering cell subsample is needed,
  e.g. hundreds of thousands to millions of cells in CSV form.
when_to_use_negative:
- The input is already a small subsample or a pre-clustered/sampled dataset (e.g.
  clustering-then-sampling output as in dropClust); sketching is designed for full
  massive datasets
- The task requires a systematic study of how the sketching distance metric interacts
  with preprocessing of downstream algorithms — the paper states this interplay was
  not systematically addressed
- The downstream algorithm's assumptions are incompatible with Sphetcher's underlying
  model — the paper only validated compatibility with Louvain community detection
  and Monocle 2
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: Sphetcher
  role: 'Core tool: spherical thresholding picks representative cells within spheres
    to produce the sketch indicator vector; Sphetcher-H adds a prior grid sampling
    step for million-cell datasets'
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository before compiling with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Downstream scRNA-seq clustering algorithm applied to the extracted sketch;
    on the umbilical cord blood sketch it recovered rare inflammatory macrophages
    (CD74, HLA-DRA, B2M, JUNB; AUROC > 0.88)
- name: Monocle 2
  role: Downstream trajectory inference tool applied to sketches, e.g. for human skeletal
    muscle myoblast differentiation
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:ed44fddf1fb44d42829728eed9344f4d0b55bccfffd73fc6a8fea6bdbb8f1ef5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Monocle 2
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-sketch-sampling

## Summary

Compute a 'spherical sketch' of a massive single-cell RNA-seq dataset with Sphetcher, which uses spherical thresholding (set cover) to pick a small representative subset of cells that evenly covers the transcriptomic space, enabling tractable downstream clustering and recovery of rare cell populations.

## When to use

Use when a scRNA-seq expression matrix is too large for downstream tools (clustering, trajectory inference) and an evenly covering cell subsample is needed, e.g. hundreds of thousands to millions of cells in CSV form.

## When NOT to use

- The input is already a small subsample or a pre-clustered/sampled dataset (e.g. clustering-then-sampling output as in dropClust); sketching is designed for full massive datasets
- The task requires a systematic study of how the sketching distance metric interacts with preprocessing of downstream algorithms — the paper states this interplay was not systematically addressed
- The downstream algorithm's assumptions are incompatible with Sphetcher's underlying model — the paper only validated compatibility with Louvain community detection and Monocle 2

## Inputs

- expression_matrix.csv: expression/PCA matrix in CSV format, rows = cells, columns = features (genes, transcripts, PCs)
- sketch_size: integer number of cells to sample (e.g. 20000 or 1% of the dataset)
- class_labels.csv (optional): column vector of prior labels (cell type, collection time point) as integers 1..K
- l_min (optional): minimum number of representatives to sample from each class

## Outputs

- sketch_indicator_output.csv: binary indicator vector of length n with exactly sketch_size ones marking selected cells
- Extracted sketched-cell expression matrix (subset of the input at indicator = 1)
- Downstream artifacts such as Louvain cluster assignments, ARI scores, or marker-gene AUROC tables (e.g. ari_scores.csv, macrophage_marker_auroc.csv, runtime_comparison.csv)

## How to apply

Clone github.com/canzarlab/Sphetcher and compile with a C++11-supporting compiler (cd src; make). Provide an expression matrix in CSV format with rows as cells and columns as features (genes, transcripts, or principal components), and choose a sketch_size (e.g. 1% of cells, such as 20296 for the 2,026,641-cell MOCA dataset). Run sphetcher to produce a binary sketch_indicator_output.csv with exactly sketch_size ones marking the selected cells; optionally supply class_labels.csv and l_min to preserve at least l_min representatives per prior cell type or time point. Optionally use the hybrid Sphetcher-H strategy (grid sampling followed by spherical thresholding/set cover) for million-cell datasets — it computed a sketch for the two-million-cell Cao et al. dataset in ~16 minutes (grid sampling 10 s, distances 600 s, set cover 400 s) versus ~30 minutes for geometric sketching. Extract the indicator=1 cells, then run downstream analysis such as Louvain community detection; note the paper mostly used Pearson correlation as the distance metric, though Euclidean distance yielded the most accurate clustering of the chen dataset by ARI.

## Related tools

- **Sphetcher** (Core tool: spherical thresholding picks representative cells within spheres to produce the sketch indicator vector; Sphetcher-H adds a prior grid sampling step for million-cell datasets) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository before compiling with a C++11 compiler) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Downstream scRNA-seq clustering algorithm applied to the extracted sketch; on the umbilical cord blood sketch it recovered rare inflammatory macrophages (CD74, HLA-DRA, B2M, JUNB; AUROC > 0.88))
- **Monocle 2** (Downstream trajectory inference tool applied to sketches, e.g. for human skeletal muscle myoblast differentiation)

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make; sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- Verify sketch_indicator_output.csv contains exactly sketch_size ones (e.g. exactly 20,000 ones for the umbilical cord blood run)
- Downstream Louvain clustering of the sketch should recover known populations, e.g. the rare inflammatory macrophage cluster with marker-gene AUROC > 0.88 on CD74, HLA-DRA, B2M, JUNB
- Compare ARI against reference cell labels across metrics: Euclidean distance should yield the highest ARI on dataset chen, with Manhattan and geometric sketching substantially lower
- On the 2M-cell MOCA dataset, Sphetcher-H total runtime should be on the order of 16 minutes (grid sampling 10 s, distances 600 s, set cover 400 s) versus ~30 minutes for geometric sketching

## Limitations

- The interplay of sketching under a given distance metric with subsequent algorithmic analysis was not systematically addressed in the study
- Benefits of leveraging prior categorical knowledge (cell types) in sketch selection were shown only at proof-of-principle level; additional experiments are required
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2, which were not tested
- No software version numbers are stated for Sphetcher or the downstream tools used

## Evidence

- [abstract] we propose algorithm Sphetcher that makes use of the thresholding technique to efficiently pick representative cells within spheres: "we propose algorithm Sphetcher that makes use of the thresholding technique to efficiently pick representative cells within spheres"
- [methods] You need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs).: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [intro] Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells.: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
- [readme] An indicator vector of n samples where 1 indicates the sample is in the sketch, 0 otherwise: "`sketch_indicator_output.csv` : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise"
- [discussion] In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"

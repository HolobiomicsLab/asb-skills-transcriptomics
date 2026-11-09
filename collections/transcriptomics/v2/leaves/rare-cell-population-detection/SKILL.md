---
name: rare-cell-population-detection
description: Use when a single-cell RNA-seq dataset is too large for downstream analysis
  (e.g., >200k cells) and a rare cell population must be recovered without uniform
  subsampling that overlooks rare states.
when_to_use_negative:
- When the dataset is small enough that clustering or trajectory tools (e.g., SC3,
  Louvain) can run directly on the full data without sketching
- When the task requires systematic evaluation of the interplay between the sketching
  distance metric and downstream preprocessing, which the authors explicitly did not
  address
- When rare-population recovery depends on prior categorical knowledge beyond proof-of-principle
  fairness sketching, since the authors note additional experiments are required
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Sphetcher
  role: Thresholding-based spherical sketching tool that selects representative cells
    covering the transcriptomic space, producing the sketch indicator vector; uses
    Pearson correlation as distance metric
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Downstream community-detection clustering of the sketched cells to reveal
    the rare inflammatory macrophage cluster
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:839e8db19d2f5ea2978f0f27157f19d38b2807c1a174b3e89dce946abc0214e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rare-cell-population-detection

## Summary

Recover rare cell populations (e.g., inflammatory macrophages) from massive scRNA-seq datasets by sketching a representative cell subset with Sphetcher's spherical thresholding, then clustering the sketch with Louvain and quantifying marker-gene enrichment by AUROC.

## When to use

Use when a single-cell RNA-seq dataset is too large for downstream analysis (e.g., >200k cells) and a rare cell population must be recovered without uniform subsampling that overlooks rare states.

## When NOT to use

- When the dataset is small enough that clustering or trajectory tools (e.g., SC3, Louvain) can run directly on the full data without sketching
- When the task requires systematic evaluation of the interplay between the sketching distance metric and downstream preprocessing, which the authors explicitly did not address
- When rare-population recovery depends on prior categorical knowledge beyond proof-of-principle fairness sketching, since the authors note additional experiments are required

## Inputs

- expression_matrix.csv: CSV matrix with rows as cells and columns as features (genes, transcripts, or principal components)
- sketch_size: integer number of cells to sample (e.g., 20000)
- class_labels.csv (optional): column vector of integer class labels (1..K) encoding prior cell type or collection time point
- l_min (optional): minimum number of representatives to sample per class

## Outputs

- sketch_indicator_output.csv: binary indicator vector of length n with exactly sketch_size ones marking sampled cells
- Louvain cluster assignments for the sketched cells
- macrophage_marker_auroc.csv: AUROC of the rare-population cluster against marker-gene labels (expected > 0.88)

## How to apply

Provide a CSV expression matrix (rows = cells, columns = features/genes/PCs) to Sphetcher, which uses thresholding to pick representative cells within spheres under Pearson correlation distance, and run it with the desired sketch_size (e.g., 20,000 from 254,941 umbilical cord blood cells) to emit sketch_indicator_output.csv containing exactly sketch_size ones. Subset the original matrix to the indicated cells, then apply Louvain community detection to cluster the sketched cells. Identify the cluster enriched for the rare population's markers (e.g., CD74, HLA-DRA, B2M, JUNB for inflammatory macrophages) and compute the AUROC of the cluster against these marker-gene labels. Validation requires an exact sketch size and marker AUROC exceeding the reported threshold (>0.88). Optionally, class_labels.csv with l_min can preserve a minimum number of representatives per prior category.

## Related tools

- **Sphetcher** (Thresholding-based spherical sketching tool that selects representative cells covering the transcriptomic space, producing the sketch indicator vector; uses Pearson correlation as distance metric) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Downstream community-detection clustering of the sketched cells to reveal the rare inflammatory macrophage cluster)

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make && sphetcher umbilical_cord_blood_pca.csv 20000 sketch_indicator_output.csv
```

## Evaluation signals

- Verify sketch_indicator_output.csv contains exactly sketch_size ones (e.g., 20,000) among n cell entries
- Confirm the Louvain clustering of the sketch recovers a cluster enriched for the marker genes CD74, HLA-DRA, B2M, and JUNB
- Check that the macrophage cluster's marker-gene AUROC exceeds 0.88 as reported in the paper
- Confirm the input expression matrix is in CSV format with cells as rows and features/PCs as columns before invoking Sphetcher
- If fairness labels are used, check that at least l_min representatives per class are included in the sketch

## Limitations

- The interplay of sketching with a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Benefits of leveraging prior categorical knowledge (e.g., cell types) in sketch selection are only demonstrated at proof-of-principle level
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2, which were not tested
- No software version numbers are stated for Sphetcher or the downstream tools used

## Evidence

- [methods] sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] Input/output format specification: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [discussion] Pearson correlation distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [intro] Rare population detection objective: "which in turn benefits downstream analyses such as clustering and allows to detect a rare population of inflammatory macrophages"
- [abstract] Representation claim: "We show that the spherical sketch computed by Sphetcher constitutes a more accurate representation of the original transcriptomic landscape"

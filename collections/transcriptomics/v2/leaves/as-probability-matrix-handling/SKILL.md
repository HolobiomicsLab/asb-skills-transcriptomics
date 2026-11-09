---
name: as-probability-matrix-handling
description: Use when starting from single-cell SMART-seq BAM or junction files (or
  Leafcutter/SJ.out.tab junctions) and you need an AS probability matrix (rows = AS
  sites, columns = cells) with missing values imputed before cell clustering on splicing
  landscapes.
when_to_use_negative:
- Input is already a finished AS probability/feature matrix — proceed to spectral
  clustering rather than re-running junction extraction and QC
- Bulk RNA-seq differential splicing with known sample groups — use tools like rMATS
  instead of SCASL's unsupervised single-cell pipeline
- The analysis goal is clustering on gene expression levels rather than splicing landscapes
  — SCASL's AS matrix layer is specifically for splicing-based cell identity definitions
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: SCASL
  role: 'Implements the full AS probability matrix workflow: junction matrix QC, AS
    grouping, normalization to AS probabilities, and iterative weighted KNN imputation'
  repo: github.com/xryanglab/SCASL
- name: Leafcutter
  role: Defines AS events from spliced junction reads that feed SCASL's AS probability
    matrix construction (installable via conda or from source added to PATH)
  repo: https://github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:314fcbe4ca1386f2322aa5c769d30159824f1ff97e13c868d57e1ba6f6f9d0d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leafcutter
  - Python
  - SCASL
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# as-probability-matrix-handling

## Summary

Construct and clean the SCASL single-cell alternative-splicing (AS) probability matrix — de novo AS event identification from junction reads, QC filtering of the junction matrix, normalization to AS probabilities, and imputation of missing (NA) values via iterative weighted KNN — so that the resulting matrix is suitable for downstream spectral clustering of cells.

## When to use

Use when starting from single-cell SMART-seq BAM or junction files (or Leafcutter/SJ.out.tab junctions) and you need an AS probability matrix (rows = AS sites, columns = cells) with missing values imputed before cell clustering on splicing landscapes.

## When NOT to use

- Input is already a finished AS probability/feature matrix — proceed to spectral clustering rather than re-running junction extraction and QC
- Bulk RNA-seq differential splicing with known sample groups — use tools like rMATS instead of SCASL's unsupervised single-cell pipeline
- The analysis goal is clustering on gene expression levels rather than splicing landscapes — SCASL's AS matrix layer is specifically for splicing-based cell identity definitions

## Inputs

- Aligned single-cell SMART-seq BAM file, or junction-read files (Leafcutter junctions or STAR SJ.out.tab)
- SCASL YAML config file (e.g., configs/srr.yaml) setting junction path, QC thresholds, knn, num_iteration, pca_dimension, num_cluster
- Demo intermediate junction files from TNBC-2 in data/junction for pipeline testing

## Outputs

- normalized_matrix: final AS probability matrix (rows = AS sites, columns = cells) with imputed NA values
- NA position indicator matrix (missing-value positions as SCASL records them)
- junc_matrix.csv / junc_mat.npy: junction reads matrix (rows = sites, columns = cells)
- filtered_matrix_start / filtered_matrix_end: QC-filtered junction matrices after AS grouping (upstream/downstream)
- site_hist.png histograms for QC threshold selection

## How to apply

Run the SCASL pipeline (`python main.py -y configs/srr.yaml`) from bam or junction input, or step-by-step using intermediate files; Leafcutter (or STAR SJ.out.tab files) provides junction reads from which AS events are identified de novo and AS probabilities are inferred from the proportions of junction reads assigned to the same 5' or 3' AS events. Apply the four QC thresholds set in configs/srr.yaml: sites_initial (min expressing cells per site in the initial junction matrix), runs_initial (min expressing sites per cell), sites_quality (min expressing cells per AS group), runs_quality (min expressing sites per cell); use the output site_hist.png histogram to help choose these cutoffs. The normalized_matrix intermediate file holds the final AS probability matrix (rows = sites, columns = cells) with NA entries where splicing coverage is absent. Missing values are then filled by SCASL's iterative weighted KNN imputation, controlled by the knn (number of adjacent cells) and num_iteration (iterations) parameters in the YAML config; an NA indicator matrix records missing positions and is later concatenated with the full AS probability matrix for clustering. To validate imputation fidelity, mask known non-NA entries at 10%/30%/90% dropout, impute, and check recovery at masked positions against the true values.

## Related tools

- **SCASL** (Implements the full AS probability matrix workflow: junction matrix QC, AS grouping, normalization to AS probabilities, and iterative weighted KNN imputation) — github.com/xryanglab/SCASL
- **Leafcutter** (Defines AS events from spliced junction reads that feed SCASL's AS probability matrix construction (installable via conda or from source added to PATH)) — https://github.com/davidaknowles/leafcutter

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- normalized_matrix has rows = AS sites and columns = cells, matching the README's intermediate-file specification
- Forced-dropout test: at 90% dropout of non-NA AS probabilities, the majority of masked values are still correctly recovered by imputation (as in Fig. S1A)
- A t-test comparing imputed vs. true values at masked positions shows no significant difference across dropout levels (10%, 30%, 90%)
- NA indicator matrix records all missing positions identically to how SCASL records naturally missing entries
- site_hist.png output is generated and used to justify the sites_initial/runs_initial/sites_quality/runs_quality cutoffs

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and AS event identification; imputation quality depends on the knn and num_iteration settings
- Coverage of genes and AS events can be very unbalanced across different single cells, making QC thresholds (min expressing cells per site / sites per cell) consequential
- The GitHub demo files (TNBC-2) are minimal and only demonstrate operation and speed; randomness of interpolation, dimensionality reduction, and unsupervised clustering can cause run-to-run differences in results

## Evidence

- [intro] AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events.: "AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events."
- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [methods] sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site"
- [readme] normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells.: "normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [readme] you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing: "you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing"

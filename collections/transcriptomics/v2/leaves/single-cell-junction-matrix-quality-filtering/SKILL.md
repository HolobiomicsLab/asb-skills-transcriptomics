---
name: single-cell-junction-matrix-quality-filtering
description: Use when a raw junction reads matrix (junc_matrix.csv / junc_mat.npy)
  has been integrated LeafCutter-style from single-cell SMART-seq BAM or SJ.out.tab
  files and must be quality-filtered before AS event grouping and probability computation.
when_to_use_negative:
- Input is already a quality-filtered AS probability matrix or a dense gene expression
  matrix rather than a raw junction reads matrix
- Bulk RNA-seq junction data intended for Leafcutter differential splicing analysis
  with known sample groups — SCASL's QC thresholds target sparse single-cell SMART-seq
  matrices
- AS analysis requiring known cell labels for differential splicing (e.g., BRIE, rMATS,
  MARVEL workflows), since this stage serves unsupervised SCASL clustering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: SCASL
  role: Implements the junction-matrix quality filtering via sites_initial/runs_initial
    and sites_quality/runs_quality parameters, plus optional log-scale QC histograms;
    the whole stage is a step in SCASL's first pipeline stage
  repo: github.com/xryanglab/SCASL
- name: Leafcutter
  role: Provides the junction-read integration scheme (and r-leafcutter conda dependency)
    used to build the raw junction reads matrix that this skill filters
  repo: github.com/davidaknowles/leafcutter
- name: STAR
  role: 'Alternative junction source: its SJ.out.tab files can replace Leafcutter-derived
    junction files as input to the filtered matrix construction'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:c285189a0f2bfd922ea8fb2ccdaa2cd72264a47f47dc69dc3b4a26658606fae1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leafcutter
  - SCASL
  - STAR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Single-cell junction matrix quality filtering

## Summary

QC stage of the SCASL pipeline that filters the single-cell junction reads matrix (sites x cells) by expression support — minimum expressing cells per site and minimum expressing sites per cell — before grouping junctions into 5'/3' AS events and computing the AS probability matrix.

## When to use

Use when a raw junction reads matrix (junc_matrix.csv / junc_mat.npy) has been integrated LeafCutter-style from single-cell SMART-seq BAM or SJ.out.tab files and must be quality-filtered before AS event grouping and probability computation.

## When NOT to use

- Input is already a quality-filtered AS probability matrix or a dense gene expression matrix rather than a raw junction reads matrix
- Bulk RNA-seq junction data intended for Leafcutter differential splicing analysis with known sample groups — SCASL's QC thresholds target sparse single-cell SMART-seq matrices
- AS analysis requiring known cell labels for differential splicing (e.g., BRIE, rMATS, MARVEL workflows), since this stage serves unsupervised SCASL clustering

## Inputs

- Raw junction reads matrix (junc_matrix.csv / junc_mat.npy) with rows as sites and columns as cells, integrated from single-cell SMART-seq BAM or STAR SJ.out.tab junction files
- YAML configuration file (configs/srr.yaml) with sites_initial, runs_initial, sites_quality, runs_quality thresholds and log_scale_histogram flag

## Outputs

- QC-filtered junction matrices grouped by splice site: filtered_matrix_start (upstream grouping) and filtered_matrix_end (downstream grouping), rows as sites, columns as cells
- Final AS probability matrix normalized_matrix (rows as sites, columns as cells) after sites_quality / runs_quality filtering
- Quality distribution histograms (site_hist.png) to guide filter threshold selection

## How to apply

Set the QC thresholds in configs/srr.yaml: sites_initial (minimum expressing cells per site) and runs_initial (minimum expressing sites per cell) for the initial junction matrix, then sites_quality (minimum expressing cells per AS group) and runs_quality (minimum expressing sites, summed over the AS group, per cell) for the downstream AS matrix. Optionally set log_scale_histogram to T to emit quality distribution histograms (site_hist.png) that guide threshold choice before committing to values. Launch the pipeline with 'python main.py -y configs/srr.yaml'; the pipeline integrates junction reads from single-cell BAM or STAR SJ.out.tab junction files, applies the initial filter, groups junctions sharing a splice site into AS groups (filtered_matrix_start / filtered_matrix_end), computes AS probabilities from proportions of junction reads assigned to each 5' or 3' AS event, and applies the AS-level quality filters to assemble the final normalized_matrix. The rationale is that single-cell splicing coverage is biased and sparse, so sites and cells with minimal support would otherwise inject noise into AS probability estimation and clustering.

## Related tools

- **SCASL** (Implements the junction-matrix quality filtering via sites_initial/runs_initial and sites_quality/runs_quality parameters, plus optional log-scale QC histograms; the whole stage is a step in SCASL's first pipeline stage) — github.com/xryanglab/SCASL
- **Leafcutter** (Provides the junction-read integration scheme (and r-leafcutter conda dependency) used to build the raw junction reads matrix that this skill filters) — github.com/davidaknowles/leafcutter
- **STAR** (Alternative junction source: its SJ.out.tab files can replace Leafcutter-derived junction files as input to the filtered matrix construction)

## Examples

```
python main.py -y configs/srr.yaml
```

## Evaluation signals

- filtered_matrix_start and filtered_matrix_end intermediate files exist and have rows corresponding to sites and columns to cells after QC
- normalized_matrix exists as the final AS probability matrix (rows = sites, columns = cells) with sites_quality and runs_quality filters applied
- Retained sites each have at least sites_initial expressing cells; retained cells each express at least runs_initial sites in the filtered matrix
- If log_scale_histogram = T, site_hist.png is generated showing the distribution of non-NA samples per site and non-NA sites per sample, consistent with the applied cutoffs

## Limitations

- Single-cell AS profiles suffer from missing data points due to dropouts and unbalanced coverage across cells, so filter thresholds strongly shape how many sites and cells survive
- QC parameter choice is dataset-dependent; the README explicitly provides the histogram figure to 'assist in the selection of filter parameters' rather than fixing universal cutoffs
- Demo data shipped in the repo is minimal (test time < 3 minutes) and only demonstrates operation, not biologically meaningful filtering behavior

## Evidence

- [readme] sites_initial defined in README: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site."
- [readme] runs_initial defined in README: "runs_initial: quality control of the initial junction matrix, minimum number of expressing sites per cell."
- [readme] histogram guides threshold choice: "log_scale_histogram: whether to output the quality distribution histogram to help determine the quality control standard, logical parameter (T or F)."
- [methods] sites_quality defined in article: "sites_quality: quality control of the AS matrix, minimum number of expressing cells per AS group"
- [intro] AS probabilities from junction read proportions: "AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events."
- [readme] normalized_matrix output description: "normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."

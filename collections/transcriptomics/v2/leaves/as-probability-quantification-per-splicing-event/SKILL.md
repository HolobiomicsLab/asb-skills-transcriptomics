---
name: as-probability-quantification-per-splicing-event
description: Use when single-cell SMART-seq BAM files or junction files (LeafCutter
  output or STAR SJ.out.tab) must be converted into a per-cell AS probability matrix
  (normalized_matrix) as stage 1 of SCASL before imputation and spectral clustering.
when_to_use_negative:
- The goal is downstream imputation of missing AS values or spectral clustering of
  cells — those are later SCASL stages operating on normalized_matrix, not this quantification
  stage
- The data are bulk RNA-seq or differential splicing with known cell labels is needed
  (tools like rMATS, BRIE, Expedition, MARVEL target those settings; SCASL targets
  unsupervised single-cell SMART-seq splicing)
- Input is already an AS probability matrix or junction-reads matrix (junc_matrix)
  — skip straight to the imputation/clustering steps
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCASL
  role: 'Implements the full stage: junction-read integration, AS-event grouping by
    shared splice site, AS probability computation, and AS-matrix QC (via main.py
    and the scasl folder py files)'
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Source of the annotation-free junction-read strategy SCASL adopts for defining
    AS events; SCASL integrates junction reads LeafCutter-style (r-leafcutter installed
    via conda)
  repo: https://github.com/davidaknowles/leafcutter
- name: STAR
  role: Its SJ.out.tab files can be used directly as the junction-file input instead
    of BAM-derived LeafCutter junctions (junction path set in configs/srr.yaml)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:2074d7f4ea87a70b3473898119435bcbae6bbbfcae898db30e8044201e7b82a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  - STAR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# as-probability-quantification-per-splicing-event

## Summary

SCASL's first pipeline stage: identify alternative splicing events de novo from junction reads in single-cell SMART-seq data (LeafCutter/FRASER-style, annotation-free) and compute per-cell AS probabilities for 3' AS and 5' AS events, yielding the final AS probability matrix (normalized_matrix).

## When to use

Use when single-cell SMART-seq BAM files or junction files (LeafCutter output or STAR SJ.out.tab) must be converted into a per-cell AS probability matrix (normalized_matrix) as stage 1 of SCASL before imputation and spectral clustering.

## When NOT to use

- The goal is downstream imputation of missing AS values or spectral clustering of cells — those are later SCASL stages operating on normalized_matrix, not this quantification stage
- The data are bulk RNA-seq or differential splicing with known cell labels is needed (tools like rMATS, BRIE, Expedition, MARVEL target those settings; SCASL targets unsupervised single-cell SMART-seq splicing)
- Input is already an AS probability matrix or junction-reads matrix (junc_matrix) — skip straight to the imputation/clustering steps

## Inputs

- Single-cell BAM files from SMART-seq scRNA-seq (any mapper, e.g. STAR)
- Junction files: LeafCutter output or STAR SJ.out.tab (directory set as 'junction' in configs/srr.yaml; auto-generated if absent)
- YAML configuration (configs/srr.yaml) with QC thresholds sites_initial, runs_initial, sites_quality, runs_quality and log_scale_histogram (T/F)

## Outputs

- junc_matrix.csv / junc_mat.npy — raw junction reads matrix, rows = sites, columns = cells
- filtered_matrix_start / filtered_matrix_end — QC-filtered, AS-grouped junction matrices (upstream vs downstream grouping), rows = sites, columns = cells
- normalized_matrix — final AS probability matrix, rows = sites, columns = cells
- site_hist.png — log-scale QC distribution histogram for choosing filter parameters

## How to apply

Configure bam (or junction) and lc directories plus the four QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality) in configs/srr.yaml and launch with 'python main.py -y configs/srr.yaml'. Integrate junction reads from the single-cell BAM/junction files LeafCutter-style into a raw junction reads matrix (junc_matrix.csv / junc_mat.npy) with rows as sites and columns as cells, then apply initial QC keeping sites expressed in at least sites_initial cells and cells expressing at least runs_initial sites (optionally emitting a log-scale histogram, log_scale_histogram T/F, to guide threshold choice). Group quality-filtered junctions sharing a common 5' or 3' splice site into AS events, producing the upstream-grouped (filtered_matrix_start) and downstream-grouped (filtered_matrix_end) matrices. Compute AS probabilities from the proportions of junction reads assigned to each 3' AS or 5' AS event, then filter the AS matrix by sites_quality (minimum expressing cells per AS group) and runs_quality (minimum expressing sites, summed over the AS group, per cell) to assemble normalized_matrix.

## Related tools

- **SCASL** (Implements the full stage: junction-read integration, AS-event grouping by shared splice site, AS probability computation, and AS-matrix QC (via main.py and the scasl folder py files)) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Source of the annotation-free junction-read strategy SCASL adopts for defining AS events; SCASL integrates junction reads LeafCutter-style (r-leafcutter installed via conda)) — https://github.com/davidaknowles/leafcutter
- **STAR** (Its SJ.out.tab files can be used directly as the junction-file input instead of BAM-derived LeafCutter junctions (junction path set in configs/srr.yaml))

## Examples

```
python main.py -y configs/srr.yaml
```

## Evaluation signals

- normalized_matrix exists as the final AS probability matrix with rows corresponding to sites and columns to cells
- QC-filtered intermediates filtered_matrix_start and filtered_matrix_end are generated, reflecting upstream (start) and downstream (end) AS grouping
- junc_matrix.csv and junc_mat.npy are produced with rows = sites and columns = cells, matching the integrated single-cell junction reads
- The four YAML QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality) were applied in the documented order: initial junction-matrix QC before AS grouping, then AS-matrix QC on the grouped event
- site_hist.png histograms (non-NA samples per site; non-NA sites per sample) were inspected to justify the chosen filter parameters

## Limitations

- Single-cell AS profiles suffer heavily from missing data points due to dropouts during scRNA-seq analysis and AS-event identification, so normalized_matrix contains NAs that this stage does not resolve (imputation is a later stage)
- Coverage of genes and AS events can be very unbalanced across different single cells, making QC threshold selection (via site_hist.png) consequential and data-dependent
- Randomness in imputation, dimensionality reduction and unsupervised clustering means repeated runs of the pipeline can differ, even when this quantification stage is deterministic
- SCASL is demonstrated on single-cell SMART-seq junction data; the GitHub demo files are minimal (TNBC-2 subset) and intended only to show operation and speed

## Evidence

- [intro] LeafCutter-style integration of junction reads: "SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data"
- [intro] AS probabilities from junction-read proportions: "AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events."
- [readme] Raw junction matrix layout: "junc_matrix.csv,junc_mat.npy: Junction reads matrix integrated from the single-cell file. In this matrix, the rows correspond to sites, while the columns represent cells."
- [readme] Final AS probability matrix layout: "normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [readme] AS-grouped QC-filtered matrices: "filtered_matrix_start,filtered_matrix_end: Junction reads matrix after quality filtering and AS grouping, the rows correspond to sites, while the columns represent cells."
- [methods] QC threshold definitions: "sites_quality: quality control of the AS matrix, minimum number of expressing cells per AS group"

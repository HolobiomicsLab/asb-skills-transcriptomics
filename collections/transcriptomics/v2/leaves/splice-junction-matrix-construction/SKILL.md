---
name: splice-junction-matrix-construction
description: Use when starting from per-cell BAM files or STAR SJ.out.tab junction
  files from single-cell SMART-seq data and you need a cells-by-sites junction matrix
  plus a 5'/3' AS probability matrix before imputation or clustering.
when_to_use_negative:
- Input is droplet-based scRNA-seq where splicing coverage is extremely sparse — the
  article notes RNA splicing profiling is more costly under current practice and SMART-seq-style
  full-length data are the demonstrated use case
- Goal is differential splicing with known cell labels (tools like rMATS, BRIE18,
  or MARVEL are designed for that); SCASL junction matrix construction targets unsupervised
  de novo AS event discovery
- You already have the AS probability matrix (e.g. the TNBC-2 demo intermediates in
  data/junction) — skip directly to imputation/clustering steps
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: 'Main pipeline: integrates per-cell junction reads into the junction matrix,
    groups junctions into AS events, and computes the AS probability matrix'
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Provides the junction-read extraction strategy SCASL emulates for annotation-free
    AS event definition; installed via conda or cloned source with PATH export
  repo: https://github.com/davidaknowles/leafcutter
- name: STAR
  role: Its SJ.out.tab files can be supplied directly as the junction input instead
    of LeafCutter output, by setting the junction path in configs/srr.yaml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:dcaafea20d48ceab2db4223521958c3393db6700f90f255b42112d03b289daa9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  - STAR
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# splice-junction-matrix-construction

## Summary

Construct a LeafCutter-style splice junction reads matrix from single-cell SMART-seq BAM/junction files and convert it into an alternative splicing (AS) probability matrix, the foundational input for SCASL's imputation and spectral clustering stages.

## When to use

Use when starting from per-cell BAM files or STAR SJ.out.tab junction files from single-cell SMART-seq data and you need a cells-by-sites junction matrix plus a 5'/3' AS probability matrix before imputation or clustering.

## When NOT to use

- Input is droplet-based scRNA-seq where splicing coverage is extremely sparse — the article notes RNA splicing profiling is more costly under current practice and SMART-seq-style full-length data are the demonstrated use case
- Goal is differential splicing with known cell labels (tools like rMATS, BRIE18, or MARVEL are designed for that); SCASL junction matrix construction targets unsupervised de novo AS event discovery
- You already have the AS probability matrix (e.g. the TNBC-2 demo intermediates in data/junction) — skip directly to imputation/clustering steps

## Inputs

- Per-cell BAM files from single-cell SMART-seq (any alignment method) or junction files (LeafCutter output or STAR SJ.out.tab)
- YAML config file (configs/srr.yaml) specifying bam, lc, junction paths and QC thresholds
- Leafcutter installation directory (conda r-leafcutter or cloned source)

## Outputs

- junc_matrix.csv / junc_mat.npy — raw junction reads matrix (rows = sites, columns = cells)
- filtered_matrix_start — upstream-grouped junction matrix after QC and AS grouping
- filtered_matrix_end — downstream-grouped junction matrix after QC and AS grouping
- normalized_matrix — final AS probability matrix (rows = sites, columns = cells)
- site_hist.png quality distribution histograms to guide threshold selection

## How to apply

Configure the bam (or junction) and lc directories plus the four QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality) in configs/srr.yaml and launch 'python main.py -y configs/srr.yaml'. Integrate junction reads across single cells LeafCutter-style into a raw junction matrix (junc_matrix.csv / junc_mat.npy) with rows as sites and columns as cells; STAR SJ.out.tab files may substitute for LeafCutter output. Apply initial QC keeping sites expressed in at least sites_initial cells and cells expressing at least runs_initial sites, optionally emitting a log-scale histogram (log_scale_histogram T/F) to guide threshold choice. Cluster quality-filtered junctions sharing a common 5' or 3' splice site into AS groups, producing the upstream-grouped (filtered_matrix_start) and downstream-grouped (filtered_matrix_end) matrices. Compute AS probabilities as proportions of junction reads assigned to the same 5' AS or 3' AS event, then filter the AS matrix by sites_quality (min expressing cells per AS group) and runs_quality (min expressing sites per cell) to yield normalized_matrix.

## Related tools

- **SCASL** (Main pipeline: integrates per-cell junction reads into the junction matrix, groups junctions into AS events, and computes the AS probability matrix) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Provides the junction-read extraction strategy SCASL emulates for annotation-free AS event definition; installed via conda or cloned source with PATH export) — https://github.com/davidaknowles/leafcutter
- **STAR** (Its SJ.out.tab files can be supplied directly as the junction input instead of LeafCutter output, by setting the junction path in configs/srr.yaml)

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- normalized_matrix exists with rows corresponding to sites and columns corresponding to cells
- QC-filtered intermediates filtered_matrix_start and filtered_matrix_end are generated alongside junc_matrix.csv / junc_mat.npy
- AS probabilities equal the proportions of junction reads assigned to each 5' AS or 3' AS scheme (values in [0,1], rows summing appropriately per AS group)
- Threshold choice is informed by the log-scale histogram outputs (site/sample non-NA count distributions) rather than arbitrary cutoffs
- On the demo data (TNBC-2 intermediates in data/junction), a full run completes in under 3 minutes with results in process_result_demo

## Limitations

- Single-cell AS profiles suffer from missing data points due to dropouts during scRNA-seq analysis and AS event identification; coverage of genes and AS events is unbalanced across cells
- Due to randomness in imputation, dimensionality reduction, and unsupervised clustering, run results may differ between executions of the demo
- Junction-based AS quantification depends on split-read coverage, making splicing profiling much more costly than gene expression quantification under current single-cell practice

## Evidence

- [intro] SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data: "SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data"
- [intro] AS probabilities are inferred from proportions of junction reads assigned to the same AS event: "AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events."
- [methods] QC threshold definitions from methods: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site"
- [readme] Normalized matrix is the final AS probability matrix: "normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [readme] Junction matrix description from README: "junc_matrix.csv,junc_mat.npy: Junction reads matrix integrated from the single-cell file. In this matrix, the rows correspond to sites, while the columns represent cells."

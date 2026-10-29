---
name: junction-read-quality-control
description: Use when building a single-cell AS probability matrix from junction reads
  (e.g., SCASL/Leafcutter output) and sparse, biased coverage must be filtered prior
  to KNN imputation and spectral clustering.
when_to_use_negative:
- Input is already an imputed, filtered AS probability matrix (normalized_matrix)
  — QC on the junction matrix has already been performed.
- Working with bulk RNA-seq splicing quantification (e.g., Leafcutter differential
  usage across bulk samples), where the sparse single-cell coverage problem motivating
  cell/site filtering does not apply.
- Only gene-expression clustering is desired (e.g., standard scRNA-seq clustering
  on a count matrix) — no junction/AS matrix is involved.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: SCASL
  role: Implements the junction and AS matrix QC filtering via the sites_initial,
    runs_initial, sites_quality, and runs_quality parameters in configs/srr.yaml
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Extracts junction reads from single-cell alignments to build the initial junction
    reads matrix that is then quality-filtered
  repo: https://github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:e3d673c08f20e9e4a3e483839085438c7ba8a5146e9a8b48c4c354d9cf02fd23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Junction-read quality control

## Summary

Quality-control filtering of the initial single-cell junction reads matrix and the derived alternative-splicing (AS) matrix in SCASL, using cell/site coverage thresholds and histogram inspection to remove sparse junction sites and low-coverage cells before imputation and clustering.

## When to use

Use when building a single-cell AS probability matrix from junction reads (e.g., SCASL/Leafcutter output) and sparse, biased coverage must be filtered prior to KNN imputation and spectral clustering.

## When NOT to use

- Input is already an imputed, filtered AS probability matrix (normalized_matrix) — QC on the junction matrix has already been performed.
- Working with bulk RNA-seq splicing quantification (e.g., Leafcutter differential usage across bulk samples), where the sparse single-cell coverage problem motivating cell/site filtering does not apply.
- Only gene-expression clustering is desired (e.g., standard scRNA-seq clustering on a count matrix) — no junction/AS matrix is involved.

## Inputs

- Integrated junction reads matrix (junc_matrix.csv, junc_mat.npy) with rows = junction sites, columns = cells, extracted from single-cell BAM or junction (SJ.out.tab) files
- QC threshold parameters in configs/srr.yaml: sites_initial, runs_initial, sites_quality, runs_quality, log_scale_histogram

## Outputs

- Quality-filtered junction matrix grouped by shared 5'/3' splice sites (filtered_matrix_start, filtered_matrix_end)
- QC distribution histograms (site_hist.png) used to choose filter parameters
- Cleaned AS matrix suitable for iterative weighted KNN imputation (yielding normalized_matrix)

## How to apply

After Leafcutter (or STAR SJ.out.tab files) yields the integrated junction reads matrix (junc_matrix.csv / junc_mat.npy), apply two rounds of filtering. First, filter the initial junction matrix with 'sites_initial' (minimum number of expressing cells per site) and 'runs_initial' (minimum number of expressing sites per cell). Then group junction reads into AS events sharing 5' and 3' splice sites (filtered_matrix_start, filtered_matrix_end) and filter the AS matrix with 'sites_quality' (minimum expressing cells per AS group) and 'runs_quality' (minimum expressing sites, summed over AS groups, per cell). Set thresholds by inspecting the QC distribution histograms (site_hist.png), which plot non-NA sample counts per site and non-NA site counts per sample; enable 'log_scale_histogram' in configs/srr.yaml to aid cutoff choice. The rationale is that single-cell AS profiles suffer greatly from missing data points due to dropouts, and unbalanced coverage would degrade the iterative weighted KNN imputation that produces the final normalized AS probability matrix.

## Related tools

- **SCASL** (Implements the junction and AS matrix QC filtering via the sites_initial, runs_initial, sites_quality, and runs_quality parameters in configs/srr.yaml) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Extracts junction reads from single-cell alignments to build the initial junction reads matrix that is then quality-filtered) — https://github.com/davidaknowles/leafcutter

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml  # sites_initial/runs_initial/sites_quality/runs_quality set in configs/srr.yaml
```

## Evaluation signals

- site_hist.png histograms show the post-filter site/sample coverage distribution and justify the chosen thresholds (horizontal axis = count of non-NA samples per site, vertical axis = number of such sites).
- filtered_matrix_start and filtered_matrix_end exist and their rows (sites) and columns (cells) respect the sites_*/runs_* minimums set in the config.
- Downstream KNN imputation converges to a normalized_matrix with reduced missingness relative to the unfiltered matrix.
- Final SCASL cluster labels (preds) reproduce the expected fine clusters for the same parameters (e.g., C3 vs C4 Treg split, activated C0 vs exhausted C6/C9 CD8 clusters in HCC tumor-associated T cells); minor run-to-run differences are acceptable due to stochastic imputation and clustering.
- Demo run on data/junction intermediate files (TNBC-2) completes in under ~3 minutes with results consistent for identical parameters.

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq analysis and AS event identification, so overly stringent thresholds may remove too many sites/cells.
- Coverage of genes and AS events can be very unbalanced across different single cells; thresholds are data-dependent and must be chosen via histogram inspection rather than fixed defaults.
- Junction extraction depends on Leafcutter; if conda installation fails, the Leafcutter source must be cloned and added to PATH, or STAR SJ.out.tab junction files can be substituted.

## Evidence

- [methods] sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site"
- [methods] runs_quality: quality control of the AS matrix, minimum number of expressing sites (sum of AS group) per cell: "runs_quality: quality control of the AS matrix, minimum number of expressing sites (sum of AS group) per cell"
- [readme] site_hist.png: This figure is used to assist in the selection of filter parameters.: "site_hist.png: This figure is used to assist in the selection of filter parameters."
- [discussion] single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events"
- [abstract] SCASL resolves the issue of biased and sparse data coverage on single-cell RNA splicing: "SCASL resolves the issue of biased and sparse data coverage on single-cell RNA splicing"

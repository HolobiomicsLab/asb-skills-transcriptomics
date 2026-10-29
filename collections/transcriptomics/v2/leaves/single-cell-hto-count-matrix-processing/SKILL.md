---
name: single-cell-hto-count-matrix-processing
description: Use when you have a cell hashing HTO count matrix (e.g. an 8-hashtag
  dataset such as Stoeckius-2018 GSE108313) and need to assign singlets, doublets
  and negatives per cell before comparing demultiplexing methods.
when_to_use_negative:
- The samples lack a distinct genetic background or are the same donors across timepoints
  and you rely solely on SNP-based demultiplexing (e.g. demuxlet, scSplit, Vireo)
  — use hashing instead, or the hybrid strategy only if hashing is available.
- The input is already a cell×sample label table or post-demultiplexing output rather
  than a raw HTO count matrix — this skill operates on raw hashtag counts in a Seurat
  object.
- The dataset contains no hashtag/HTO assay (e.g. genetic demultiplexing-only experiments)
  — HTOreader's HTOClassification requires an HTO assay in the Seurat object.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: HTOreader
  role: 'Primary R toolkit for HTO demultiplexing: HTOClassification assigns hashtag
    identities, PlotHTO checks cutoff quality'
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Loads the HTO assay as a Seurat object and provides VlnPlot for visualizing
    hashtag distributions
- name: HTOdemux
  role: Baseline demultiplexing method run from Seurat with positive quantile 0.99
    for benchmark comparison
- name: MULTI-seq
  role: Baseline demultiplexing method run with default settings on the same count
    matrix
- name: GMM_Demux
  role: Baseline demultiplexing method run with default settings on the same count
    matrix
- name: DropletUtils
  role: Baseline demultiplexing method run with default settings on the same count
    matrix
- name: BFF
  role: Baseline run as BFF_raw and BFF_cluster from the cellhashR package with default
    parameters
- name: scSplit
  role: Provides genetic-variant-based ground truth labels used to validate hashing-based
    demultiplexing assignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:536d520beac3082761f5be208f5521a35ae7f4868514476108b338ce2523931a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BFF
  - DropletUtils
  - GMM_Demux
  - HTOdemux
  - HTOreader
  - MULTI-seq
  - Seurat
  - cellhashR
  - scSplit
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-hto-count-matrix-processing

## Summary

Process single-cell HTO (hashtag oligo) count matrices for cell hashing demultiplexing by loading the HTO assay into a Seurat object and running HTOreader's HTOClassification with CLR (or log) normalization to assign per-cell hashtag identities.

## When to use

Use when you have a cell hashing HTO count matrix (e.g. an 8-hashtag dataset such as Stoeckius-2018 GSE108313) and need to assign singlets, doublets and negatives per cell before comparing demultiplexing methods.

## When NOT to use

- The samples lack a distinct genetic background or are the same donors across timepoints and you rely solely on SNP-based demultiplexing (e.g. demuxlet, scSplit, Vireo) — use hashing instead, or the hybrid strategy only if hashing is available.
- The input is already a cell×sample label table or post-demultiplexing output rather than a raw HTO count matrix — this skill operates on raw hashtag counts in a Seurat object.
- The dataset contains no hashtag/HTO assay (e.g. genetic demultiplexing-only experiments) — HTOreader's HTOClassification requires an HTO assay in the Seurat object.

## Inputs

- HTO (hashtag oligo) count matrix, e.g. from GEO accession GSE108313 Stoeckius-2018 8-hashtag dataset
- Seurat object containing an 'HTO' assay
- Optional: ground-truth cell labels (e.g. scSplit-derived) for validation

## Outputs

- Per-cell hashtag assignments (HTOid) classified as singlet, doublet or negative
- PlotHTO cutoff-quality plots (list of ggplot objects per hashtag)
- Benchmark metrics table (e.g. benchmark_metrics.csv) with singlet rate, accuracy, recall, precision and F1 per demultiplexing method

## How to apply

Install flexmix, hash and devtools, then install HTOreader from GitHub via devtools::install_github. Load the HTO assay as a Seurat object using the Seurat package. Run HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR') to obtain per-cell hashtag assignments (HTOid); alternatively use method = 'log' or supply specify_cutoff (e.g. c(2.5,2,2,2,2,2,2,2)) manually. Use PlotHTO to visualize each hashtag's distribution and check cutoff quality, adjusting the cutoff per HTO if the automated calling is poor — this matters especially with many hashtags or imbalanced sample sizes where negative signals can dominate. In benchmark contexts (e.g. Stoeckius-2018 vs scSplit ground truth), run the same count matrix through HTOdemux (positive quantile 0.99), MULTI-seq, GMM-Demux, DropletUtils, BFF_raw and BFF_cluster with defaults, then compute singlet rate, accuracy, recall, precision and F1 per method against the ground-truth labels.

## Related tools

- **HTOreader** (Primary R toolkit for HTO demultiplexing: HTOClassification assigns hashtag identities, PlotHTO checks cutoff quality) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Loads the HTO assay as a Seurat object and provides VlnPlot for visualizing hashtag distributions)
- **HTOdemux** (Baseline demultiplexing method run from Seurat with positive quantile 0.99 for benchmark comparison)
- **MULTI-seq** (Baseline demultiplexing method run with default settings on the same count matrix)
- **GMM_Demux** (Baseline demultiplexing method run with default settings on the same count matrix)
- **DropletUtils** (Baseline demultiplexing method run with default settings on the same count matrix)
- **BFF** (Baseline run as BFF_raw and BFF_cluster from the cellhashR package with default parameters)
- **scSplit** (Provides genetic-variant-based ground truth labels used to validate hashing-based demultiplexing assignments)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plots1 <- PlotHTO(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- benchmark_metrics.csv contains all five metrics (singlet rate, accuracy, recall, precision, F1) for all seven methods (HTOreader, HTOdemux, MULTI-seq, GMM-Demux, DropletUtils, BFF_raw, BFF_cluster)
- Every computed metric lies in the [0,1] range
- HTOreader yields comparable singlet rates of 80-83% to other methods on the Stoeckius-2018 benchmark when validated against scSplit ground truth
- PlotHTO ggplot outputs exist for every hashtag in the assay and cutoffs visually separate positive from negative distributions
- Per-sample/per-class confusion counts (true positives, false positives, false negatives) reconcile to the total cell count for each method

## Limitations

- Cell hashing has difficulties distinguishing true positive from background signal, unpredictable hashtag failure, and high false doublet rates when using many hashtags
- Poor staining quality of certain hashtags can decrease cell recovery by increasing the negative rate; tuning parameters only marginally improves performance on challenging datasets
- The hybrid strategy's performance is largely dependent on the quality of SNP-based clustering, which fails for plasmablasts/dying cells and requires distinct genetic backgrounds
- Manual cutoff specification may be needed when automated cutoff calling misbehaves on individual hashtags

## Evidence

- [methods] HTOClassification assigns hashtags with CLR normalization; PlotHTO checks per-hashtag cutoff quality: "`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')`"
- [readme] README documents HTOreader installation and the two main functions HTOClassification and PlotHTO: "There are two main functions, `HTOClassification` for Hashtag demultiplexing, `PlotHTO` for plotting distributions of hashtags"
- [abstract] Article abstract states HTOreader improves cutoff calling accuracy for experiments with many hashtags or imbalanced sample sizes: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [results] Task card benchmark finding reports comparable singlet rates across all methods on Stoeckius-2018: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion notes hashtag staining quality and negative-rate effects limit cell recovery: "the poor staining quality of certain hashtags that occur sporadically can further decrease cell recovery by increasing the negative rate"

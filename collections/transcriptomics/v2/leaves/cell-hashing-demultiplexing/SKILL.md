---
name: cell-hashing-demultiplexing
description: Use when a Seurat object carries an 'HTO' assay from a cell-hashing experiment
  and per-cell sample identities (singlet/doublet/negative) must be called, especially
  with many hashtags, imbalanced sample sizes, or degraded hashtag staining.
when_to_use_negative:
- Samples lack distinct genetic backgrounds and no HTO assay exists (e.g. most animal
  studies or same-donor longitudinal studies) — cell hashing cutoff-based demultiplexing
  requires hashtag-stained cells
- Input is SNP/genotype-based demultiplexing output rather than an HTO count matrix
  — that calls for the hybrid strategy (HybridDemultiplexing), not HTOClassification
  alone
- The HTO assay shows severely degraded staining for hashtags where even parameter
  tuning cannot rescue assignment; the article notes tuning only marginally improves
  performance on such challenging datasets
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: HTOreader
  role: 'Primary demultiplexing toolkit: HTOClassification performs CLR-normalized
    cutoff calling per hashtag; PlotHTO visualizes cutoff quality'
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Hosts the 'HTO' assay in a Seurat object and provides VlnPlot for inspecting
    HTO signal by assigned HTOid; HTOdemux runs via Seurat
- name: HTOdemux
  role: Baseline cell-hashing demultiplexer run with positive quantile 0.99 for benchmark
    comparison
- name: MULTI-seq
  role: Baseline demultiplexer compared on benchmark and R125 datasets
- name: GMM_Demux
  role: Baseline Gaussian-mixture demultiplexer run with defaults
- name: DropletUtils
  role: Baseline demultiplexer run with defaults on the same HTO matrix
- name: BFF / cellhashR
  role: Provides BFF_raw and BFF_cluster baseline calls run with default parameters
- name: scSplit
  role: SNP-based genetic demultiplexer supplying ground-truth cell labels for benchmark
    validation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:ce34308c100df98101057ffbc38572abb119ad052eff7794fb77aa9ddc711347
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BFF
  - BFF / cellhashR
  - DropletUtils
  - GMM_Demux
  - HTOClassification
  - HTOdemux
  - HTOreader
  - MULTI-seq
  - PlotHTO
  - Seurat
  - cellhashR
  - scSplit
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cell-hashing-demultiplexing

## Summary

Assigns each cell in a pooled multi-sample single-cell experiment to its sample of origin (singlet, doublet, or negative) based on hashtag oligo (HTO) counts, using HTOreader's improved cutoff calling with CLR normalization and comparison against methods such as HTOdemux, MULTI-seq, GMM-Demux, BFF and DropletUtils.

## When to use

Use when a Seurat object carries an 'HTO' assay from a cell-hashing experiment and per-cell sample identities (singlet/doublet/negative) must be called, especially with many hashtags, imbalanced sample sizes, or degraded hashtag staining.

## When NOT to use

- Samples lack distinct genetic backgrounds and no HTO assay exists (e.g. most animal studies or same-donor longitudinal studies) — cell hashing cutoff-based demultiplexing requires hashtag-stained cells
- Input is SNP/genotype-based demultiplexing output rather than an HTO count matrix — that calls for the hybrid strategy (HybridDemultiplexing), not HTOClassification alone
- The HTO assay shows severely degraded staining for hashtags where even parameter tuning cannot rescue assignment; the article notes tuning only marginally improves performance on such challenging datasets

## Inputs

- HTO count matrix loaded as the 'HTO' assay of a Seurat object (e.g. 8-hashtag Stoeckius-2018 matrix from GSE108313; 3-hashtag R125 matrix from Mendeley DOI 10.17632/3jdywv5jrv.3)
- Optional ground-truth sample labels (e.g. scSplit-derived) for benchmark validation
- Optional per-hashtag cutoff vector for specify_cutoff

## Outputs

- Per-cell hashtag identity assignments (singlet/doublet/negative) stored as 'HTOid' in the Seurat object
- Per-hashtag cutoff diagnostics as ggplot objects from PlotHTO (rendered with plot_grid)
- Comparison table of singlet rate, accuracy, recall, precision and F1 per method (e.g. benchmark_metrics.csv, r125_demultiplexing_comparison.csv)

## How to apply

Load the HTO assay into a Seurat object and run HTOreader's HTOClassification with method = 'CLR' under default parameters to obtain per-cell hashtag assignments stored as 'HTOid'; optionally pass specify_cutoff as a vector (e.g. c(2.5,2,2,2,2,2,2,8-style per-HTO thresholds)) when automatic cutoffs look poor, or switch to method = 'log'. Check cutoff quality per hashtag with PlotHTO (returns a list of ggplot objects rendered via plot_grid) and with a VlnPlot of HTO signal grouped by HTOid. In benchmark settings, run comparison methods (HTOdemux with positive quantile 0.99, MULTI-seq, GMM-Demux, DropletUtils, BFF_raw/BFF_cluster from cellhashR) on the same count matrix, then tabulate per-method singlet/doublet/negative rates and compute accuracy, recall, precision and F1 against ground-truth labels (e.g. scSplit-derived labels for GSE108313). The rationale is that HTOreader improves cutoff calling by avoiding dominance of negative signals, which causes methods like MULTI-seq and HTOdemux to collapse many cells into the negative group on real-world datasets (e.g. R125: 49.7% and 62.3% singlet rates vs ~94.5% for BFF_cluster).

## Related tools

- **HTOreader** (Primary demultiplexing toolkit: HTOClassification performs CLR-normalized cutoff calling per hashtag; PlotHTO visualizes cutoff quality) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Hosts the 'HTO' assay in a Seurat object and provides VlnPlot for inspecting HTO signal by assigned HTOid; HTOdemux runs via Seurat)
- **HTOdemux** (Baseline cell-hashing demultiplexer run with positive quantile 0.99 for benchmark comparison)
- **MULTI-seq** (Baseline demultiplexer compared on benchmark and R125 datasets)
- **GMM_Demux** (Baseline Gaussian-mixture demultiplexer run with defaults)
- **DropletUtils** (Baseline demultiplexer run with defaults on the same HTO matrix)
- **BFF / cellhashR** (Provides BFF_raw and BFF_cluster baseline calls run with default parameters)
- **scSplit** (SNP-based genetic demultiplexer supplying ground-truth cell labels for benchmark validation)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plots1 <- PlotHTO(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- benchmark_metrics.csv contains all five metrics (singlet rate, accuracy, recall, precision, F1) for every method, each metric within [0,1]
- On Stoeckius-2018, all methods should show comparable singlet rates of 80-83% with high accuracy/recall/F1 against scSplit ground truth
- On R125, reproduce the expected contrast: MULTI-seq ~49.7% and HTOdemux ~62.3% singlet rates (large negative-group assignments) vs ~94.5% for BFF_cluster
- PlotHTO output renders one cutoff plot per hashtag via plot_grid, allowing visual confirmation that cutoffs separate positive from negative HTO signal
- Assigned HTOid counts partition every cell into exactly one of singlet/doublet/negative across the pooled cells

## Limitations

- Existing methods fail on datasets with many hashtags or imbalanced sample sizes because negative signals dominate cutoff calling; parameter tuning only marginally improves performance
- Poor staining quality of certain hashtags decreases cell recovery by inflating the negative rate, as seen with MULTI-seq and HTOdemux on R125
- Cell hashing alone has difficulties distinguishing true positive from background signal, unpredictable hashtag failure, and high false doublet rates with many hashtags
- SNP-based demultiplexing (used for hybrid validation) is inapplicable when samples lack distinct genetic backgrounds and hybrid performance depends on SNP clustering quality

## Evidence

- [abstract] HTOreader improves cutoff calling: "we developed HTOreader, a demultiplexing tool for cell hashing that improves the accuracy of cut-off calling"
- [abstract] negative-signal dominance: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [methods] HTOClassification invocation: "`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')`"
- [readme] manual cutoff option: "Users can also specify the cutoff manually:  <br>`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"
- [readme] PlotHTO cutoff QC: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [methods] install via devtools: "devtools::install_github("WilsonImmunologyLab/HTOreader")"
- [readme] load HTO assay: "Please load your "HTO" assay using the Seurat package"
- [discussion] tuning insufficient on hard datasets: "while tuning parameters can marginally improve performance, it is not sufficient to overcome failures on challenging datasets"

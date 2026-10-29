---
name: seurat-hto-assay-processing
description: Use when a pooled multi-sample scRNA-seq experiment has a cell-hashing
  (HTO) assay stored in a Seurat object and cells must be assigned singlet/doublet/negative
  hashtag identities, especially with many hashtags or imbalanced sample sizes where
  cutoff calling dominates negative assignment.
when_to_use_negative:
- Input samples lack a hashtag/HTO assay entirely (e.g. SNP-only demultiplexing inputs
  with no cell hashing staining).
- The goal is genetic-variant (SNP-based) demultiplexing from transcriptome data —
  that requires genotype clustering methods (demuxlet, scSplit, Vireo), not HTO assay
  processing.
- The Seurat object does not contain an assay named 'HTO'; HTOClassification and PlotHTO
  require assay = 'HTO'.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HTOreader
  role: 'Primary tool: HTOClassification for hashtag demultiplexing with automatic
    cutoff calling; PlotHTO for cutoff inspection'
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Loads the 'HTO' assay as a Seurat object and renders VlnPlot of HTO signal
    grouped by HTOid
- name: HTOdemux
  role: Seurat's baseline hashtag demultiplexing method run on the same HTO matrix
    for comparison
- name: MULTI-seq
  role: Comparison method on the same HTO matrix; lowest singlet rate (49.7%) on R125
    due to negative-group over-assignment
- name: GMM_Demux
  role: Comparison method producing per-cell hashtag identity assignments on the HTO
    matrix
- name: BFF
  role: Comparison method (BFF_raw and BFF_cluster) producing per-cell hashtag assignments
- name: DropletUtils
  role: Comparison method producing per-cell hashtag identity assignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:16a38be991aa189f28b36db2b337eef3fa803e67cf3e7a297250734f7f3fc923
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BFF
  - DropletUtils
  - GMM_Demux
  - HTOClassification
  - HTOdemux
  - HTOreader
  - MULTI-seq
  - PlotHTO
  - Seurat
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# seurat-hto-assay-processing

## Summary

Prepare and process a Seurat object's HTO assay (hashtag oligo counts) for cell-hashing demultiplexing, including loading the 'HTO' assay, running HTOreader's HTOClassification with CLR or log normalization and automatic or manually specified cutoffs, and inspecting cutoff quality with PlotHTO. This is the prerequisite step for any hashtag-based singlet/doublet/negative calling in the hybrid demultiplexing workflow.

## When to use

Use when a pooled multi-sample scRNA-seq experiment has a cell-hashing (HTO) assay stored in a Seurat object and cells must be assigned singlet/doublet/negative hashtag identities, especially with many hashtags or imbalanced sample sizes where cutoff calling dominates negative assignment.

## When NOT to use

- Input samples lack a hashtag/HTO assay entirely (e.g. SNP-only demultiplexing inputs with no cell hashing staining).
- The goal is genetic-variant (SNP-based) demultiplexing from transcriptome data — that requires genotype clustering methods (demuxlet, scSplit, Vireo), not HTO assay processing.
- The Seurat object does not contain an assay named 'HTO'; HTOClassification and PlotHTO require assay = 'HTO'.

## Inputs

- Seurat object containing an 'HTO' assay (hashtag oligo count/normalized matrix)
- Optional numeric vector of manually specified per-hashtag cutoffs (specify_cutoff)
- Normalization method choice: 'CLR' (default) or 'log'

## Outputs

- Seurat object with HTOid assignment metadata (per-cell singlet/doublet/negative hashtag identities)
- List of ggplot objects from PlotHTO showing each hashtag's distribution and called cutoff
- VlnPlot of HTO signal grouped by assigned HTOid

## How to apply

Load the 'HTO' assay into a Seurat object using the Seurat package, then install HTOreader via devtools after ensuring the flexmix and hash dependencies are present. Run HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR') to demultiplex hashtags with automatic cutoff calling; if a hashtag distribution is poorly separated, either rerun with method = 'log' or supply specify_cutoff as an explicit numeric vector (e.g. c(2.5,2,2,2,2,2,2,2)) — HTOreader was designed to avoid the dominance of negative signals that causes methods like MULTI-seq and HTOdemux to collapse many cells into the negative group on degraded datasets such as R125. Visualize per-cell HTO signal with VlnPlot grouped by the assigned HTOid, and verify cutoff quality per hashtag with PlotHTO rendered via plot_grid(plotlist = plots1). The result is per-cell singlet/doublet/negative assignments that can feed downstream comparison (GMM-Demux, BFF, DropletUtils) or the SNP-based HybridDemultiplexing step.

## Related tools

- **HTOreader** (Primary tool: HTOClassification for hashtag demultiplexing with automatic cutoff calling; PlotHTO for cutoff inspection) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Loads the 'HTO' assay as a Seurat object and renders VlnPlot of HTO signal grouped by HTOid)
- **HTOdemux** (Seurat's baseline hashtag demultiplexing method run on the same HTO matrix for comparison)
- **MULTI-seq** (Comparison method on the same HTO matrix; lowest singlet rate (49.7%) on R125 due to negative-group over-assignment)
- **GMM_Demux** (Comparison method producing per-cell hashtag identity assignments on the HTO matrix)
- **BFF** (Comparison method (BFF_raw and BFF_cluster) producing per-cell hashtag assignments)
- **DropletUtils** (Comparison method producing per-cell hashtag identity assignments)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plots1 <- PlotHTO(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plot_grid(plotlist = plots1)
```

## Evaluation signals

- PlotHTO output for each hashtag shows a well-called cutoff separating positive signal from background; if not, rerun with method = 'log' or specify_cutoff.
- Negative-group rate is not dominated by misassigned cells — on the degraded R125 3-hashtag dataset, correct processing should avoid the failure mode of MULTI-seq (~49.7% singlets) and HTOdemux (~62.3% singlets) while BFF_cluster reaches ~94.5%.
- Per-method counts and rates of singlets, doublets, and negatives tabulate across the pooled cells and are written to a comparison table (r125_demultiplexing_comparison.csv).
- VlnPlot of HTO signal grouped by HTOid shows clean separation of positive versus negative cells per hashtag.

## Limitations

- Poor staining quality of certain hashtags decreases cell recovery by increasing the negative rate; parameter tuning only marginally improves performance on such degraded datasets.
- Cell hashing with many hashtags yields high false doublet rates and difficulty distinguishing true positive from background signal.
- Cutoff calling can be dominated by negative signals when there are many hashtags or imbalanced sample sizes, which is what HTOreader's automatic cutoff calling tries to mitigate.
- Cell hashing alone incurs high reagent costs when staining large numbers of cells; the hybrid SNP+hashing strategy addresses this but depends on SNP clustering quality.

## Evidence

- [readme] README HTOClassification invocation: "`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')`"
- [readme] README manual cutoff: "`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"
- [readme] README PlotHTO cutoff check: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [readme] README install dependencies: "make sure you installed these two dependencies:<br>`install.packages("flexmix")` <br> `install.packages("hash")`"
- [abstract] Abstract cutoff calling rationale: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [methods] Methods load HTO assay: "Please load your "HTO" assay using the Seurat package"
- [results] R125 failure mode finding: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion tuning limits: "while tuning parameters can marginally improve performance, it is not sufficient to overcome failures on challenging datasets"

---
name: hto-cutoff-thresholding
description: Use when a Seurat object carries an HTO assay from a cell hashing experiment
  and per-cell hashtag singlet/doublet/negative assignments require cutoff calling,
  especially with many hashtags or imbalanced sample sizes.
when_to_use_negative:
- The input has no HTO assay and demultiplexing must rely on genetic variation (use
  SNP-based tools such as demuxlet, scSplit, Vireo, Souporcell instead)
- Hashtag assignments already exist and the goal is only to merge them with SNP-based
  labels — that is the HybridDemultiplexing step, not cutoff calling
- The experiment is not a cell hashing (hashtag oligo) design, e.g. MULTI-seq lipid-tag
  barcodes with a different assay structure
edam_operation: null
tools:
- name: HTOreader
  role: Provides HTOClassification, PlotHTO, HTOcutoff and HTOIdAssign for cutoff-based
    hashtag demultiplexing
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Container for the HTO assay; the HTO assay is loaded and assigned back onto
    a Seurat object
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:3ba8198e37a464626a798e76de1ff070c862b6201302fae830fc38c76cc9b811
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BFF
  - DropletUtils
  - GMM_Demux
  - HTOdemux
  - HTOreader
  - Seurat
  - cellhashR
  - scSplit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# hto-cutoff-thresholding

## Summary

Determine per-hashtag cutoff thresholds for cell hashing demultiplexing using HTOreader's HTOClassification with CLR (or log) normalization, inspecting each HTO's distribution with PlotHTO to verify cutoff quality and optionally supplying manual cutoffs.

## When to use

Use when a Seurat object carries an HTO assay from a cell hashing experiment and per-cell hashtag singlet/doublet/negative assignments require cutoff calling, especially with many hashtags or imbalanced sample sizes.

## When NOT to use

- The input has no HTO assay and demultiplexing must rely on genetic variation (use SNP-based tools such as demuxlet, scSplit, Vireo, Souporcell instead)
- Hashtag assignments already exist and the goal is only to merge them with SNP-based labels — that is the HybridDemultiplexing step, not cutoff calling
- The experiment is not a cell hashing (hashtag oligo) design, e.g. MULTI-seq lipid-tag barcodes with a different assay structure

## Inputs

- Seurat object containing an 'HTO' assay of hashtag oligo (HTO) counts
- Optional numeric vector of per-hashtag cutoffs (specify_cutoff)
- Normalization method choice: 'CLR' (default) or 'log'

## Outputs

- Per-cell hashtag identity assignments (HTOid: singlet/doublet/negative) stored on the Seurat object
- List of per-HTO ggplot objects from PlotHTO showing hashtag distributions and called cutoffs

## How to apply

Load the HTO assay into a Seurat object (dependencies flexmix and hash installed, HTOreader via devtools::install_github('WilsonImmunologyLab/HTOreader')). Run HTOClassification with assay='HTO' and method='CLR' as the default normalization; if CLR cut-offs look poor, rerun with method='log' or supply specify_cutoff as a numeric vector of per-HTO thresholds (e.g. c(2.5,2,2,2,2,2,2,2) for an 8-hashtag panel). After assignment, call PlotHTO with the same assay/method (and optional specify_cutoff) to inspect each HTO's distribution and check the quality of the cutoff — negative and positive signal modes should be separated by the threshold, not dominated by background. Rationale: HTOreader improves cut-off calling by avoiding dominance of negative signals in experiments with many hashtags or imbalanced sample sizes, where default methods misassign cells; users can also drive lower-level functions HTOcutoff and HTOIdAssign directly when custom thresholds are needed.

## Related tools

- **HTOreader** (Provides HTOClassification, PlotHTO, HTOcutoff and HTOIdAssign for cutoff-based hashtag demultiplexing) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Container for the HTO assay; the HTO assay is loaded and assigned back onto a Seurat object)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- PlotHTO plots show each HTO's cutoff separating positive and negative signal modes, with no cutoff dominated by background signal
- The returned object contains per-cell HTOid assignments covering singlet, doublet and negative classes
- If specify_cutoff is supplied, its length equals the number of hashtags (e.g. 8 values for the 8-hashtag example) and each value is applied to the matching HTO
- VlnPlot of HTO normalized values grouped by HTOid shows coherent separation consistent with the chosen CLR/log method
- Manual cutoffs (e.g. 2.5 for one hashtag, 2 for the rest) only adjust hashtags whose default cutoff was judged poor in PlotHTO

## Limitations

- Unpredictable hashtag failure and poor staining quality of certain hashtags can occur sporadically, increasing the negative rate and decreasing cell recovery
- Distinguishing true positive from background signal is difficult, and high false doublet rates arise when using many hashtags
- Default CLR cutoffs may be inadequate, requiring manual specify_cutoff values informed by PlotHTO inspection

## Evidence

- [abstract] cutoff calling: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [methods] HTOClassification invocation: "`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')`"
- [readme] PlotHTO quality check: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [readme] manual cutoff: "Users can also specify the cutoff manually:  <br>`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"

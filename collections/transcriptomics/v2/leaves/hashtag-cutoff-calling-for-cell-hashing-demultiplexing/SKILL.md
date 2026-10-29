---
name: hashtag-cutoff-calling-for-cell-hashing-demultiplexing
description: Use when a Seurat object carries an 'HTO' assay from a cell hashing experiment
  and per-hashtag cutoffs separating true-positive from background signal are needed
  to assign singlet/doublet/negative identities, especially with many hashtags or
  imbalanced sample sizes.
when_to_use_negative:
- No 'HTO' assay is available (e.g. only SNP genotype data or gene expression) — genetic
  demultiplexing tools such as demuxlet, scSplit, or Vireo are the appropriate choice,
  and SNP-based methods additionally require distinct genetic backgrounds.
- The goal is correlating cell hashing labels with SNP clustering results (HybridDemultiplexing)
  rather than deriving per-hashtag cutoffs from hashtag distributions alone.
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: HTOreader
  role: 'Primary R toolkit: HTOClassification/HTOcutoff fit the mixture and call per-hashtag
    cutoffs, HTOIdAssign assigns cell identities, PlotHTO visualizes cutoff quality'
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: flexmix
  role: R dependency used to fit the two-component Gaussian mixture to each hashtag's
    normalized signal
- name: Seurat
  role: Hosts the input 'HTO' assay as a Seurat object and provides VlnPlot for reviewing
    assigned HTOid groups
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:0e841043cb73625c8e0693ec5cec7e9a64e5e85795d9e4e1ee8034b48fdfff69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTOreader
  - Seurat
  - flexmix
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hashtag-cutoff-calling-for-cell-hashing-demultiplexing

## Summary

Calls per-hashtag classification cutoffs that separate true-positive from background signal in cell hashing (HTO) data by fitting a two-component Gaussian mixture to normalized hashtag counts, then labels each cell singlet, doublet, or negative. Designed to improve cutoff accuracy by avoiding dominance of negative signals in experiments with many hashtags or imbalanced sample sizes.

## When to use

Use when a Seurat object carries an 'HTO' assay from a cell hashing experiment and per-hashtag cutoffs separating true-positive from background signal are needed to assign singlet/doublet/negative identities, especially with many hashtags or imbalanced sample sizes.

## When NOT to use

- No 'HTO' assay is available (e.g. only SNP genotype data or gene expression) — genetic demultiplexing tools such as demuxlet, scSplit, or Vireo are the appropriate choice, and SNP-based methods additionally require distinct genetic backgrounds.
- The goal is correlating cell hashing labels with SNP clustering results (HybridDemultiplexing) rather than deriving per-hashtag cutoffs from hashtag distributions alone.

## Inputs

- Seurat object with an 'HTO' assay containing raw hashtag oligo counts per cell (quick-start format of HTOreader)
- Normalization method choice ('CLR' default, or 'log') and optional per-hashtag specify_cutoff vector (e.g. c(2.5,2,2,2,2,2,2,2))

## Outputs

- Per-cell identity labels (singlet, doublet, negative) attached to the Seurat object, viewable via VlnPlot of hashtag features grouped by 'HTOid'
- Per-hashtag classification cutoffs — one cutoff for every hashtag, inspectable by re-plotting hashtag distributions with PlotHTO
- hto_demultiplexing_results.csv with a cutoff value for every hashtag and a label for every cell restricted to {singlet, doublet, negative}

## How to apply

Load the hashtag oligo (HTO) count matrix as a Seurat object with an 'HTO' assay and normalize counts with the centered log-ratio (CLR) transformation (method='CLR'; re-run with method='log' as a variant). When the number of hashtags N exceeds 4, subsample the negative-signal population using the empirical CLR cutoff of 1.5 (3 for log) with stride floor(N/2) so background cells do not dominate the fit — this addresses the negative-signal dominance that degrades cutoff calling in many-hashtag or imbalanced-sample-size experiments. For each hashtag, fit a two-component Gaussian mixture with FlexMix, locate the lower (negative) and upper (positive) density peaks via findMaximum/findMinimum, compute the cutoff with the rank-2 sigma weighted formula, then apply the empirical-floor correction so no cutoff falls below the global density minimum. Assign each cell a singlet, doublet, or negative label by comparing its normalized hashtag values against the per-hashtag cutoffs (HTOIdAssign logic). Inspect cutoff quality with PlotHTO (returned as a list of ggplot objects for plot_grid), and override poor automatic cutoffs with specify_cutoff, e.g. c(2.5,2,2,2,2,2,2,2).

## Related tools

- **HTOreader** (Primary R toolkit: HTOClassification/HTOcutoff fit the mixture and call per-hashtag cutoffs, HTOIdAssign assigns cell identities, PlotHTO visualizes cutoff quality) — github.com/WilsonImmunologyLab/HTOreader
- **flexmix** (R dependency used to fit the two-component Gaussian mixture to each hashtag's normalized signal)
- **Seurat** (Hosts the input 'HTO' assay as a Seurat object and provides VlnPlot for reviewing assigned HTOid groups)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- A cutoff value exists for every hashtag and a label exists for every cell, with labels restricted to the values {singlet, doublet, negative}
- PlotHTO per hashtag shows the cutoff placed between the lower (background) and upper (positive) density peaks rather than inside the dominant negative mode
- With N > 4 hashtags, the negative population was subsampled with re_sampling_cutoff 1.5 (3 for log) and stride floor(N/2) before mixture fitting
- No per-hashtag cutoff falls below the global density minimum after the empirical-floor correction; cutoffs equal specify_cutoff exactly when supplied manually
- VlnPlot of hashtag features grouped by the assigned HTOid shows clean separation of positive cells from background, as in the README example

## Limitations

- Poor staining quality of certain hashtags can occur sporadically and decrease cell recovery by increasing the negative rate; hashtag failure is unpredictable.
- Existing cutoff-calling approaches fail on challenging datasets (many hashtags, imbalanced sample sizes), and parameter tuning only marginally improves performance — HTOreader's resampling addresses this but high false doublet rates remain when using many hashtags.
- Automatic cutoffs can be poor for individual hashtags; manual specify_cutoff plus PlotHTO inspection may be required to find the best cutoff.

## Evidence

- [abstract] we developed HTOreader, a demultiplexing tool for cell hashing that improves the accuracy of cut-off calling: "we developed HTOreader, a demultiplexing tool for cell hashing that improves the accuracy of cut-off calling"
- [intro] HTOreader, a novel tool that improves the cutoff calling of individual hashtag distributions in real-world datasets that have imbalanced sample sizes or use many hashtags: "HTOreader, a novel tool that improves the cutoff calling of individual hashtag distributions in real-world datasets that have imbalanced sample sizes or use many hashtags"
- [methods] Users can use PlotHTO function to check the quality of the cutoff of each HTO: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [readme] pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR'): "pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')"
- [readme] or use a different normalization method (CLR or log): "or use a different normalization method (CLR or log)"

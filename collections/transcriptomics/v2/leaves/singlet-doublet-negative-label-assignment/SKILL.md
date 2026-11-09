---
name: singlet-doublet-negative-label-assignment
description: Use when you have normalized per-hashtag counts plus per-hashtag cutoffs
  and must classify each barcode as singlet, doublet, or negative to demultiplex a
  cell hashing experiment.
when_to_use_negative:
- Samples lack a distinct genetic background is not the issue here — rather, do not
  use this alone to cross-validate sample identity; HTOreader's paper notes potential
  errors are best caught by the hybrid SNP-plus-hashing strategy (HybridDemultiplexing)
- Input is raw UMI counts without an 'HTO' assay loaded in a Seurat object, or the
  data are transcriptome/SNP genotypes rather than hashtag oligo counts
- The hashtag distributions are so degenerate (e.g. sporadic poor staining of a hashtag)
  that mixture fitting yields unreliable cutoffs without manual specify_cutoff inspection
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: HTOreader
  role: Provides HTOClassification, HTOcutoff, and HTOIdAssign for Gaussian-mixture-based
    cutoff calling and singlet/doublet/negative assignment from normalized HTO counts
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: flexmix
  role: R dependency used to fit the two-component Gaussian mixture to each hashtag's
    normalized counts
- name: Seurat
  role: Hosts the Seurat object with the 'HTO' assay and provides CLR normalization
    and VlnPlot visualization of per-hashtag distributions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:7eae4d4ac5b17776df11c0b7ee36148f0e500e44d7516802d91f84538ae7b59f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTOreader
  - Seurat
  - flexmix
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# singlet-doublet-negative-label-assignment

## Summary

Assigns each cell in a cell-hashing (HTO) experiment a singlet, doublet, or negative label by comparing its normalized hashtag values against per-hashtag cutoffs computed by HTOreader's Gaussian-mixture cutoff calling.

## When to use

Use when you have normalized per-hashtag counts plus per-hashtag cutoffs and must classify each barcode as singlet, doublet, or negative to demultiplex a cell hashing experiment.

## When NOT to use

- Samples lack a distinct genetic background is not the issue here — rather, do not use this alone to cross-validate sample identity; HTOreader's paper notes potential errors are best caught by the hybrid SNP-plus-hashing strategy (HybridDemultiplexing)
- Input is raw UMI counts without an 'HTO' assay loaded in a Seurat object, or the data are transcriptome/SNP genotypes rather than hashtag oligo counts
- The hashtag distributions are so degenerate (e.g. sporadic poor staining of a hashtag) that mixture fitting yields unreliable cutoffs without manual specify_cutoff inspection

## Inputs

- Seurat object with an 'HTO' assay containing raw per-hashtag counts
- Per-hashtag classification cutoffs (computed by HTOcutoff or supplied manually via specify_cutoff)
- Normalization method choice: 'CLR' or 'log'

## Outputs

- Per-cell demultiplexing label restricted to {singlet, doublet, negative} (e.g. hto_demultiplexing_results.csv, HTOid column in the Seurat object)
- Per-hashtag cutoff values stored alongside per-cell labels
- Per-hashtag distribution plots (PlotHTO ggplot objects) for cutoff quality inspection

## How to apply

First run HTOreader's cutoff calling: normalize the HTO counts (CLR via Seurat's HTODemux-style transformation, or log as a variant), fit a two-component Gaussian mixture per hashtag with FlexMix, locate the negative and positive density peaks with findMaximum/findMinimum, and compute each cutoff with the rank-2 sigma weighting plus the empirical-floor correction. If the number of hashtags N exceeds 4, subsample the negative-signal population using the empirical CLR cutoff of 1.5 (3 for the log method) with stride floor(N/2) to prevent negative-signal dominance. Then apply HTOIdAssign-style logic: for each cell, compare its normalized value for every hashtag against that hashtag's cutoff to decide whether the cell exceeds exactly one hashtag (singlet), more than one (doublet), or none (negative). Manual cutoffs can be supplied via specify_cutoff, and cutoff quality should be checked qualitatively with PlotHTO, which returns a list of ggplot objects of the hashtag distributions.

## Related tools

- **HTOreader** (Provides HTOClassification, HTOcutoff, and HTOIdAssign for Gaussian-mixture-based cutoff calling and singlet/doublet/negative assignment from normalized HTO counts) — github.com/WilsonImmunologyLab/HTOreader
- **flexmix** (R dependency used to fit the two-component Gaussian mixture to each hashtag's normalized counts)
- **Seurat** (Hosts the Seurat object with the 'HTO' assay and provides CLR normalization and VlnPlot visualization of per-hashtag distributions)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- hto_demultiplexing_results.csv exists with a cutoff value for every hashtag and a label for every cell
- Every per-cell label is restricted to the values {singlet, doublet, negative}
- PlotHTO returns a list of ggplot objects, one per hashtag, whose visualized cutoffs plausibly separate the negative peak from the positive peak in each hashtag distribution
- VlnPlot of the HTO assay features grouped by 'HTOid' shows coherent separation of singlet, doublet, and negative populations
- No per-hashtag cutoff falls below the global density minimum after the empirical-floor correction

## Limitations

- Cell hashing has difficulties distinguishing true positive from background signal, unpredictable hashtag failure, and high false doublet rates when using many hashtags
- Poor staining quality of certain hashtags can decrease cell recovery by increasing the negative rate
- With imbalanced sample sizes or many hashtags, negative-signal dominance can skew cutoffs, which HTOreader mitigates by negative-population subsampling; existing methods may still fail on challenging datasets and parameter tuning only marginally improves performance

## Evidence

- [abstract] HTOreader improves cut-off calling accuracy by avoiding dominance of negative signals in experiments with many hashtags or imbalanced sample sizes: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [readme] Seurat object with HTO assay input: "The `pbmc.hashtag` in the following example is a Seurat object. Please load your "HTO" assay using the Seurat package"
- [readme] Manual cutoff specification in HTOClassification: "pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))"
- [methods] PlotHTO for cutoff quality check: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [methods] HTOIdAssign function role: "- `HTOIdAssign(data, cutoff)`"

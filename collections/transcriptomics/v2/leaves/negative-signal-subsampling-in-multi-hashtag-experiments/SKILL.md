---
name: negative-signal-subsampling-in-multi-hashtag-experiments
description: Use when demultiplexing a cell hashing experiment with more than 4 hashtags
  or imbalanced sample sizes, where negative-signal dominance degrades cutoff accuracy
  in HTOreader's HTOcutoff step.
when_to_use_negative:
- Experiments with 4 or fewer hashtags, where the subsampling trigger (N > 4) does
  not apply and negative signals are less likely to dominate
- Datasets without an 'HTO' assay loadable as a Seurat object (e.g. raw FASTQ or non-hashing
  samples)
- Samples being demultiplexed by SNP/genetic-variation methods alone (e.g. demuxlet,
  scSplit, Vireo), which do not use hashtag distributions
tools:
- name: HTOreader
  role: Implements the subsampling and cutoff-calling logic via HTOcutoff with the
    re_sampling_cutoff parameter
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: flexmix
  role: Fits the two-component Gaussian mixture to the subsampled normalized hashtag
    signal
- name: Seurat
  role: Loads the 'HTO' assay as a Seurat object that HTOClassification/HTOcutoff
    operate on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:c6e994c96ebc738d37818aae5ebbb22ef28389a9b1dd451a92b4e72a6fbbb208
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTOreader
  - Seurat
  - flexmix
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# negative-signal-subsampling-in-multi-hashtag-experiments

## Summary

A cutoff-calling refinement in HTOreader that subsamples the negative-signal population when a cell hashing experiment uses many hashtags (N > 4), using an empirical CLR cutoff of 1.5 (3 for the log method) and a stride of floor(N/2) so background cells do not dominate the Gaussian mixture fit and skew per-hashtag cutoffs.

## When to use

Use when demultiplexing a cell hashing experiment with more than 4 hashtags or imbalanced sample sizes, where negative-signal dominance degrades cutoff accuracy in HTOreader's HTOcutoff step.

## When NOT to use

- Experiments with 4 or fewer hashtags, where the subsampling trigger (N > 4) does not apply and negative signals are less likely to dominate
- Datasets without an 'HTO' assay loadable as a Seurat object (e.g. raw FASTQ or non-hashing samples)
- Samples being demultiplexed by SNP/genetic-variation methods alone (e.g. demuxlet, scSplit, Vireo), which do not use hashtag distributions

## Inputs

- HTO count matrix as a Seurat object with an 'HTO' assay
- Number of hashtags N (determines stride = floor(N/2))
- Normalization method choice: 'CLR' (default) or 'log'
- re_sampling_cutoff parameter value (1.5 for CLR, 3 for log)

## Outputs

- Per-hashtag cutoff values after negative-signal subsampling
- Subsampled negative population used for Gaussian mixture fitting
- Per-cell labels (singlet, doublet, negative) downstream via HTOIdAssign

## How to apply

After loading the HTO count matrix as a Seurat object with an 'HTO' assay, normalize with centered log-ratio (CLR) transformation (or the log method as a variant). When the number of hashtags N exceeds 4, subsample the negative-signal population before mixture-model fitting: cells below the empirical CLR cutoff of 1.5 (3 when using the log method) are subsampled with a stride of floor(N/2) to prevent negative-signal dominance. The subsampled normalized counts are then fit to two Gaussian components per hashtag with FlexMix, and the weighted rank-2 sigma formula with an empirical-floor correction (no cutoff below the global density minimum) yields the per-hashtag cutoff. This is exposed via the HTOcutoff function's re_sampling_cutoff parameter, and cutoff quality can be inspected with PlotHTO.

## Related tools

- **HTOreader** (Implements the subsampling and cutoff-calling logic via HTOcutoff with the re_sampling_cutoff parameter) — github.com/WilsonImmunologyLab/HTOreader
- **flexmix** (Fits the two-component Gaussian mixture to the subsampled normalized hashtag signal)
- **Seurat** (Loads the 'HTO' assay as a Seurat object that HTOClassification/HTOcutoff operate on)

## Examples

```
HTOcutoff(object = pbmc.hashtag, assay = 'HTO', method = 'CLR', re_sampling_cutoff = 1.5)
```

## Evaluation signals

- Subsampling only activates when the number of hashtags N > 4; for N <= 4, verify cutoffs are fit on the full negative population
- The stride equals floor(N/2) and the empirical subsampling threshold is 1.5 for CLR normalization (3 for log); check the HTOcutoff call includes the matching re_sampling_cutoff
- No resulting cutoff may fall below the global density minimum (empirical-floor correction invariant)
- PlotHTO on each hashtag should show the fitted cutoff separating the lower (negative) and upper (positive) Gaussian density peaks
- Downstream HTOIdAssign labels must be restricted to {singlet, doublet, negative} for every cell

## Limitations

- Subsampling addresses negative-signal dominance only; unpredictable hashtag failure or poor staining quality can still raise the negative rate and reduce cell recovery
- The empirical cutoffs (1.5 CLR / 3 log) are dataset-level thresholds; some hashtags may still require manual cutoffs via specify_cutoff in HTOClassification or PlotHTO

## Evidence

- [abstract] improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [methods] HTOcutoff signature with re_sampling_cutoff parameter: "`HTOcutoff(object, assay, method, min_limit, re_sampling_cutoff)`"
- [readme] README on manual cutoff specification and normalization variants: "Users can also specify the cutoff manually:  <br>`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"

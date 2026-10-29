---
name: gaussian-mixture-fitting-of-hashtag-distributions
description: Use when demultiplexing cell hashing experiments and per-hashtag cutoffs
  must distinguish true positive from background signal, especially with many hashtags
  or imbalanced sample sizes.
when_to_use_negative:
- The goal is genetic-variant-based demultiplexing and samples lack a distinct genetic
  background (e.g. most animal studies or same-donor longitudinal studies) — cell
  hashing cutoff calling is unrelated to SNP clustering needs
- Cutoffs are already trusted and supplied externally — use HTOClassification's specify_cutoff
  argument instead of refitting the mixture model
- Input is not a cell hashing HTO count matrix (e.g. transcriptome gene expression
  matrix or SNP genotype calls)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: HTOreader
  role: 'Implements the full cutoff-calling pipeline: HTOcutoff (mixture-based cutoff
    determination), HTOIdAssign (cell label assignment), and PlotHTO (cutoff quality
    inspection)'
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: flexmix
  role: Required dependency used to fit the two-component Gaussian mixture to each
    normalized hashtag distribution
- name: Seurat
  role: Loads the 'HTO' assay as a Seurat object and provides VlnPlot for visualizing
    demultiplexed HTOid labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:718da007a13efde6927e11f4ac02e937b34970fe6a1d34cc59adfd1e65262bf5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTOreader
  - Seurat
  - flexmix
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gaussian-mixture-fitting-of-hashtag-distributions

## Summary

Fits a two-component Gaussian mixture (background vs true positive) to each normalized hashtag oligo (HTO) distribution in cell hashing data to determine per-hashtag classification cutoffs, as implemented in HTOreader.

## When to use

Use when demultiplexing cell hashing experiments and per-hashtag cutoffs must distinguish true positive from background signal, especially with many hashtags or imbalanced sample sizes.

## When NOT to use

- The goal is genetic-variant-based demultiplexing and samples lack a distinct genetic background (e.g. most animal studies or same-donor longitudinal studies) — cell hashing cutoff calling is unrelated to SNP clustering needs
- Cutoffs are already trusted and supplied externally — use HTOClassification's specify_cutoff argument instead of refitting the mixture model
- Input is not a cell hashing HTO count matrix (e.g. transcriptome gene expression matrix or SNP genotype calls)

## Inputs

- Seurat object with an 'HTO' assay containing raw hashtag oligo counts (e.g. Stoeckius-2018 cell hashing matrix)
- Normalization method choice: 'CLR' (default) or 'log'
- Optional manually specified cutoffs (specify_cutoff vector)

## Outputs

- Per-hashtag classification cutoff values
- Per-cell labels restricted to {singlet, doublet, negative}
- List of ggplot objects from PlotHTO showing each hashtag distribution with its fitted cutoff

## How to apply

Load the HTO count matrix as a Seurat object with an 'HTO' assay and normalize counts with centered log-ratio (CLR) transformation, or the log method as a variant. When the number of hashtags N exceeds 4, subsample the negative-signal population using the empirical CLR cutoff of 1.5 (3 for log) with stride floor(N/2) to prevent negative-signal dominance. For each hashtag, fit a two-component Gaussian mixture to the normalized signal with FlexMix, locating the lower (negative) and upper (positive) density peaks via maximum/minimum finding (findMaximum/findMinimum). Compute the per-hashtag cutoff with the weighted formula using rank-2 sigma weighting, then apply the empirical-floor correction so no cutoff falls below the global density minimum. Assign each cell a singlet, doublet, or negative label by comparing normalized values against the cutoffs (HTOIdAssign logic), and inspect cutoff quality qualitatively by re-plotting hashtag distributions with PlotHTO.

## Related tools

- **HTOreader** (Implements the full cutoff-calling pipeline: HTOcutoff (mixture-based cutoff determination), HTOIdAssign (cell label assignment), and PlotHTO (cutoff quality inspection)) — github.com/WilsonImmunologyLab/HTOreader
- **flexmix** (Required dependency used to fit the two-component Gaussian mixture to each normalized hashtag distribution)
- **Seurat** (Loads the 'HTO' assay as a Seurat object and provides VlnPlot for visualizing demultiplexed HTOid labels)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plots1 <- PlotHTO(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- hto_demultiplexing_results.csv contains a cutoff value for every hashtag in the assay
- Every cell carries a label restricted to exactly {singlet, doublet, negative}
- PlotHTO output shows each fitted cutoff positioned between the lower (negative) and upper (positive) density peaks of the two-Gaussian fit
- No computed cutoff falls below the global density minimum after the empirical-floor correction
- Re-running with method='log' (cutoff 3 instead of CLR's 1.5 for negative-signal subsampling) yields consistent singlet/doublet/negative assignments

## Limitations

- Existing cell hashing demultiplexing approaches can fail on challenging datasets with poor hashtag staining quality, high false doublet rates with many hashtags, and difficulty distinguishing true positive from background signal
- Parameter tuning only marginally improves performance when the underlying cutoff calling fails on certain datasets
- Poor staining quality of sporadic hashtags can decrease cell recovery by increasing the negative rate

## Evidence

- [abstract] HTOreader improves cut-off calling accuracy by avoiding dominance of negative signals: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [methods] make sure you installed these two dependencies: "make sure you installed these two dependencies:<br>`install.packages("flexmix")` <br> `install.packages("hash")`"
- [methods] HTOcutoff function signature: "- `HTOcutoff(object, assay, method, min_limit, re_sampling_cutoff)`"
- [readme] PlotHTO cutoff quality check: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [readme] manual cutoff specification: "`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"

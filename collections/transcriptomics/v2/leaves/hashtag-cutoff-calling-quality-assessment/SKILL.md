---
name: hashtag-cutoff-calling-quality-assessment
description: Use when a Seurat object with an 'HTO' assay has been demultiplexed (or
  is about to be) with HTOreader and you need to verify each hashtag's cutoff, especially
  with imbalanced sample sizes, many hashtags, or suspected poor hashtag staining.
when_to_use_negative:
- When the samples have distinct genetic backgrounds and SNP-based demultiplexing
  alone is sufficient — the hybrid strategy, not cutoff inspection, is the article's
  recommended accuracy booster
- When the input is not an HTO assay in a Seurat object (e.g. genetic variant data
  such as VCF/BAM for demuxlet/Vireo) — PlotHTO/HTOClassification do not apply
- When only global singlet/doublet rate comparisons between methods are needed without
  diagnosing cutoff placement
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HTOreader
  role: Provides PlotHTO for per-hashtag cutoff inspection and HTOClassification for
    automatic cutoff calling and demultiplexing
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Hosts the 'HTO' assay as a Seurat object and provides VlnPlot for visualizing
    HTO signal grouped by assigned HTOid
- name: HTOdemux
  role: Comparison demultiplexing method (Seurat) whose per-cell assignments can be
    contrasted with HTOreader cutoff-based calls
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:522657180ba93e9512295d2b3f9ee5195c55d570089be8609a6dd6e2af8b8f2a
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
  - PlotHTO
  - Seurat
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hashtag-cutoff-calling-quality-assessment

## Summary

Assess and, if needed, manually refine HTOreader's automatic per-hashtag cutoffs by plotting each hashtag's signal distribution with PlotHTO, so that singlet/doublet/negative calls from HTOClassification are not corrupted by poorly called cutoffs (e.g. degraded hashtag staining as in the R125 dataset).

## When to use

Use when a Seurat object with an 'HTO' assay has been demultiplexed (or is about to be) with HTOreader and you need to verify each hashtag's cutoff, especially with imbalanced sample sizes, many hashtags, or suspected poor hashtag staining.

## When NOT to use

- When the samples have distinct genetic backgrounds and SNP-based demultiplexing alone is sufficient — the hybrid strategy, not cutoff inspection, is the article's recommended accuracy booster
- When the input is not an HTO assay in a Seurat object (e.g. genetic variant data such as VCF/BAM for demuxlet/Vireo) — PlotHTO/HTOClassification do not apply
- When only global singlet/doublet rate comparisons between methods are needed without diagnosing cutoff placement

## Inputs

- Seurat object containing an 'HTO' assay (per-cell hashtag UMI counts), loaded with the Seurat package
- Normalization method choice for HTO signals ('CLR' or 'log')
- Optional vector of manually specified per-hashtag cutoffs passed via specify_cutoff

## Outputs

- Per-hashtag cutoff quality plots (list of ggplot objects from PlotHTO, combinable with plot_grid)
- Per-cell HTOid assignments (singlet/doublet/negative) from HTOClassification stored on the Seurat object
- VlnPlot of HTO signal grouped by assigned HTOid for visual confirmation

## How to apply

Load the HTO assay into a Seurat object and run HTOreader's HTOClassification (e.g. method = 'CLR'; 'log' is the alternative) to obtain automatic cutoffs and HTOid assignments. Then run PlotHTO on the same assay with the same normalization method; it returns a list of ggplot objects, one per hashtag, that you combine with plot_grid to inspect where each cutoff sits relative to the positive and background (negative) signal modes. Judge cutoff quality by whether the cutoff separates true positive signal from background — poorly stained hashtags (as with degraded staining in the R125 3-hashtag dataset) can push cells into the negative group and inflate negative rates. If a cutoff is misplaced, re-run HTOClassification or PlotHTO with specify_cutoff set to a vector of per-hashtag values (e.g. c(2.5,2,2,2,2,2,2,2)). This step matters because HTOreader was designed to improve cutoff calling by avoiding dominance of negative signals in experiments with many hashtags or imbalanced sample sizes.

## Related tools

- **HTOreader** (Provides PlotHTO for per-hashtag cutoff inspection and HTOClassification for automatic cutoff calling and demultiplexing) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Hosts the 'HTO' assay as a Seurat object and provides VlnPlot for visualizing HTO signal grouped by assigned HTOid)
- **HTOdemux** (Comparison demultiplexing method (Seurat) whose per-cell assignments can be contrasted with HTOreader cutoff-based calls)

## Examples

```
plots1 <- PlotHTO(pbmc.hashtag, assay = 'HTO', method = 'CLR'); plot_grid(plotlist = plots1)
```

## Evaluation signals

- PlotHTO returns one ggplot object per hashtag; every hashtag's cutoff should visually separate the positive peak from the negative/background peak
- After accepting cutoffs, the negative-cell fraction should not be spuriously inflated — on the R125 degraded-staining dataset, HTOdemux and MULTI-seq misassigned large fractions of cells to the negative group (singlet rates 62.3% and 49.7%) while BFF_cluster reached ~94.5%, so compare against such baselines
- HTOid assignments should partition all cells into singlets, doublets, and negatives with no unlabeled cells
- Changing the normalization method (CLR vs log) or supplying specify_cutoff should produce visible cutoff shifts in the PlotHTO output, confirming the parameters were honored
- VlnPlot of HTO signals grouped by HTOid should show clear separation of signal intensities between assigned hashtags

## Limitations

- Poor staining quality of certain hashtags, occurring sporadically, can decrease cell recovery by increasing the negative rate; parameter tuning only marginally improves performance on challenging datasets
- Automatic cutoff calling can be biased when negative signals dominate, e.g. in experiments with many hashtags or imbalanced sample sizes
- Cutoffs are assay- and normalization-dependent (CLR vs log); a cutoff valid under one method is not transferable to the other

## Evidence

- [readme] check cutoff quality: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [readme] plot_grid combination: "this function returns a list of ggplot objects, use plot_grid function to plot them"
- [readme] manual cutoffs: "Users can also specify the cutoff manually:  <br>`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"
- [abstract] cutoff improvement rationale: "improves the accuracy of cut-off calling by avoiding the dominance of negative signals in experiments with many hashtags or imbalanced sample sizes"
- [discussion] staining failure mode: "the poor staining quality of certain hashtags that occur sporadically can further decrease cell recovery by increasing the negative rate"

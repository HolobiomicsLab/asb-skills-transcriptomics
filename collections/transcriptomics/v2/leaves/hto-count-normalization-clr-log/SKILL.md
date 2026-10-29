---
name: hto-count-normalization-clr-log
description: Use when a pooled multi-sample single-cell experiment has raw HTO (hashtag)
  counts loaded as a Seurat object and cells must be normalized (CLR or log) and demultiplexed
  into singlet/doublet/negative hashtag identities.
when_to_use_negative:
- The cells must be demultiplexed by genotype/SNP clustering (demuxlet, scSplit, Vireo,
  Souporcell) rather than by hashtag counts — SNP-based demultiplexing does not operate
  on the HTO assay.
- The hashtag data are not stored as an 'HTO' assay in a Seurat object, so HTOClassification's
  expected input structure is absent.
- The goal is to assign cell identities without hashtags at all (e.g. samples lacking
  distinct hashtag staining), where neither CLR nor log normalization of HTO counts
  can be applied.
tools:
- name: HTOreader
  role: Provides HTOClassification, which performs CLR or log normalization of the
    raw HTO counts, calls per-hashtag cutoffs (automatic or via specify_cutoff), and
    assigns per-cell HTOid singlet/doublet/negative labels.
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Used to load and store the 'HTO' assay as a Seurat object and to render VlnPlot
    of HTO features grouped by the assigned HTOid.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:7959c98ed39e4338527a74b86ba22a3a8a1fe33b1674e8bec631e24f7f74c2f2
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# hto-count-normalization-clr-log

## Summary

Normalize raw hashtag oligo (HTO) counts stored as a Seurat 'HTO' assay using either CLR or log normalization within HTOreader's HTOClassification, then assign per-cell singlet/doublet/negative identities via automatic or manually specified cutoffs. This normalization choice directly drives the quality of cutoff calling for hashtag-based demultiplexing.

## When to use

Use when a pooled multi-sample single-cell experiment has raw HTO (hashtag) counts loaded as a Seurat object and cells must be normalized (CLR or log) and demultiplexed into singlet/doublet/negative hashtag identities.

## When NOT to use

- The cells must be demultiplexed by genotype/SNP clustering (demuxlet, scSplit, Vireo, Souporcell) rather than by hashtag counts — SNP-based demultiplexing does not operate on the HTO assay.
- The hashtag data are not stored as an 'HTO' assay in a Seurat object, so HTOClassification's expected input structure is absent.
- The goal is to assign cell identities without hashtags at all (e.g. samples lacking distinct hashtag staining), where neither CLR nor log normalization of HTO counts can be applied.

## Inputs

- Raw HTO (hashtag oligo) count matrix stored as an 'HTO' assay in a Seurat object (e.g. pbmc.hashtag)
- Optional: user-specified per-hashtag cutoff vector (specify_cutoff, e.g. c(2.5,2,2,2,2,2,2,2))

## Outputs

- Per-cell hashtag identity assignments (HTOid: singlet/doublet/negative) added to the Seurat object
- Automatic per-hashtag cutoff values
- ggplot objects from PlotHTO (per-hashtag distributions with cutoffs) and a VlnPlot of HTO signal grouped by HTOid

## How to apply

Load the HTO assay as a Seurat object using the Seurat package, ensuring HTOreader's dependencies (flexmix, hash) are installed via install.packages before devtools::install_github('WilsonImmunologyLab/HTOreader'). Run HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR') to normalize the raw HTO counts with centered log-ratio transformation and obtain automatic per-hashtag cutoffs and per-cell HTOid assignments; alternatively pass method = 'log' to use log normalization if CLR cutoffs look poor. If the automatic cutoff call is unsatisfactory, supply a per-hashtag cutoff vector via specify_cutoff (e.g. c(2.5,2,2,2,2,2,2,2)) — the vector length must match the number of hashtags. Validate the normalization and cutoffs by rendering VlnPlot of the HTO features grouped by HTOid and PlotHTO per-hashtag distribution plots (combined with plot_grid) to confirm cutoffs separate negative background from true positive signal.

## Related tools

- **HTOreader** (Provides HTOClassification, which performs CLR or log normalization of the raw HTO counts, calls per-hashtag cutoffs (automatic or via specify_cutoff), and assigns per-cell HTOid singlet/doublet/negative labels.) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Used to load and store the 'HTO' assay as a Seurat object and to render VlnPlot of HTO features grouped by the assigned HTOid.)

## Examples

```
pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR')
```

## Evaluation signals

- HTOClassification returns a non-empty HTOid column covering all cells in the 'HTO' assay, with values restricted to singlet, doublet, and negative classes.
- VlnPlot of the rownames of the HTO assay's data slot grouped by 'HTOid' shows visually separated hashtag signal distributions across assigned identities.
- PlotHTO (optionally with plot_grid(plotlist = plots1)) shows each called cutoff positioned between the negative background and positive hashtag distribution; poor cutoffs should prompt retry with method = 'log' or manual specify_cutoff.
- The specify_cutoff vector length equals the number of hashtags (e.g. 8 values for 8 hashtags, such as c(2.5,2,2,2,2,2,2,2)).
- Switching normalization from method = 'CLR' to method = 'log' and re-running should produce consistent or inspectable differences in assignments, with cutoff quality re-checked via PlotHTO.

## Limitations

- Automatic cutoff calling can misassign a large fraction of cells into the negative group on datasets with degraded hashtag staining quality (on the real-world R125 3-hashtag dataset, existing methods like MULTI-seq and HTOdemux dropped to ~49.7% and ~62.3% singlet rates), so manual specify_cutoff or the log normalization method may be needed.
- Poor staining quality of certain hashtags can decrease cell recovery by increasing the negative rate, affecting cutoff placement regardless of the normalization method.
- HTOreader requires the flexmix and hash packages to be installed beforehand, and installation is via devtools::install_github rather than CRAN.

## Evidence

- [readme] The `pbmc.hashtag` in the following example is a Seurat object. Please load your "HTO" assay using the Seurat package: "Please load your "HTO" assay using the Seurat package (More details in our example)."
- [readme] CLR normalization via HTOClassification: "There are two main functions, `HTOClassification` for Hashtag demultiplexing, `PlotHTO` for plotting distributions of hashtags"
- [readme] Alternative log normalization method: "or use a different normalization method (CLR or log):"
- [readme] Manual cutoff specification: "Users can also specify the cutoff manually:  <br>`pbmc.hashtag <- HTOClassification(pbmc.hashtag, assay = 'HTO', method = 'CLR', specify_cutoff = c(2.5,2,2,2,2,2,2,2))`"
- [readme] Cutoff quality checking via PlotHTO: "Users can use PlotHTO function to check the quality of the cutoff of each HTO"
- [methods] Dependency installation requirement: "make sure you installed these two dependencies:<br>`install.packages("flexmix")` <br> `install.packages("hash")`"
- [other] Negative-group misassignment on degraded staining: [evidence span withheld — anchor status: not_found]

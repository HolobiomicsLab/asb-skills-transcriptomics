---
name: outlyingness-score-visualization
description: Use when SCISSOR has returned a ScissorOutput object with GSC$OS and
  LSC$OS outlyingness scores and you must identify and report which samples exceed
  the global/local shape-change cutoffs for a gene such as TP53.
when_to_use_negative:
- The Scissor() detection step has not yet been run — there are no outlyingness scores
  or cutoffs to visualize yet
- Samples with severe RNA degradation were not excluded first (e.g., the 70 severely
  degraded cases out of 522 TCGA HNSC), since degraded profiles would dominate the
  outlyingness plots
- The goal is mutation calling or junction-based isoform quantification rather than
  coverage-shape outlier screening, for which tools like DEXSeq, MISO, or rMATS are
  designed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: Provides plot_ScissorOS for visualizing GSC and LSC outlyingness scores against
    significance cutoffs from the ScissorOutput object
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:4ec30e533c49dc196c1ac9c5dd922e8c9184b0a7ef0109ff560535a8d89d475d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - build_gaf
  - get_Ranges
  - plot_ScissorOS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# outlyingness-score-visualization

## Summary

Visualizes per-sample outlyingness scores from SCISSOR's global (GSC) and local (LSC) shape-change detection, using plot_ScissorOS with sample labels, against the derived significance cutoffs (e.g., 1e-04), to identify which samples deviate from the gene's coverage shape.

## When to use

Use when SCISSOR has returned a ScissorOutput object with GSC$OS and LSC$OS outlyingness scores and you must identify and report which samples exceed the global/local shape-change cutoffs for a gene such as TP53.

## When NOT to use

- The Scissor() detection step has not yet been run — there are no outlyingness scores or cutoffs to visualize yet
- Samples with severe RNA degradation were not excluded first (e.g., the 70 severely degraded cases out of 522 TCGA HNSC), since degraded profiles would dominate the outlyingness plots
- The goal is mutation calling or junction-based isoform quantification rather than coverage-shape outlier screening, for which tools like DEXSeq, MISO, or rMATS are designed

## Inputs

- ScissorOutput object from the Scissor() function, containing $SC (identified shape changes), $GSC$OS and $LSC$OS outlyingness scores, and cutoff values
- Significance cutoff level (e.g., 1e-04) used to flag outlier samples

## Outputs

- Plots of global and local outlyingness scores per sample with outlier labels (textSC=TRUE)
- List of outlier sample IDs per procedure (e.g., written to tp53_scissor_outliers.csv)

## How to apply

After running the all-in-one Scissor function (which performs log-transformation, base-level normalization, global shape-change detection in low-dimensional space, and local shape-change detection in residual space), extract the outlyingness scores ScissorOutput$GSC$OS and ScissorOutput$LSC$OS together with their cutoff values. Call plot_ScissorOS with colmat=NULL and textSC=TRUE so samples exceeding the cutoff are labeled in the plot, separately for the global and local procedures. Compare plotted outlier samples against ScissorOutput$SC to confirm the identified shape changes; for TP53 in the 452 non-degraded TCGA HNSC samples at a 1e-04 significance cutoff, expect 26 GSCs and 24 LSCs. Write the identified outlier sample IDs per procedure to a file (e.g., tp53_scissor_outliers.csv) for downstream validation.

## Related tools

- **SCISSOR** (Provides plot_ScissorOS for visualizing GSC and LSC outlyingness scores against significance cutoffs from the ScissorOutput object) — https://github.com/hyochoi/SCISSOR

## Examples

```
plot_ScissorOS(ScissorOutput, colmat = NULL, textSC = TRUE)
```

## Evaluation signals

- At a 1e-04 significance level on the 452 non-degraded TCGA HNSC samples, the TP53 analysis should recover 26 global shape changes (GSCs) and 24 local shape changes (LSCs)
- Samples labeled in the plot_ScissorOS output (textSC=TRUE) must match the shape-change IDs in ScissorOutput$SC and the scores above the cutoff in $GSC$OS / $LSC$OS
- The written outlier CSV must contain one distinct set of sample IDs per procedure (global vs local) consistent with the plotted cutoffs
- Scores for non-outlier samples should fall below the reported cutoff values, confirming correct cutoff extraction

## Limitations

- LSC detection results depend on the window size choice — different window sizes may give alternative results
- GSC detection loses information about the order of base pairs
- Tuning parameters (log normalization shift, normality cutoff) may require optimization for RNA-seq protocols different from TCGA's

## Evidence

- [results] SCISSOR implements a two-step procedure, global and local, taking advantage of low-dimensional transformations and sparsity: "SCISSOR implements a two-step procedure, global and local, taking advantage of low-dimensional transformations and sparsity"
- [results] 452 samples TP53 counts: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [other] workflow extraction of outlyingness scores: [evidence span withheld — anchor status: not_found]
- [readme] SCISSOR purpose: "`SCISSOR` aims for unsupervised screening of a range of structural alterations in RNA-seq data."

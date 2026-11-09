---
name: replicate-quality-validation-pca-correlation
description: Use when RiboSeq samples have passed footprint QC (read-length ~28 nt,
  metagene periodicity, CDS enrichment) and you must decide which replicates are consistent
  before running DESeq2 differential expression.
when_to_use_negative:
- Input is still raw FastQ or unaligned BAM data — the qualitative footprint QC (read-length,
  periodicity, CDS enrichment) must come first
- Only one replicate per condition is available, since pairwise correlation and PCA
  require multiple samples to compare
- Data are not ribosome-profiling counts (e.g. RNA-seq feature counts analyzed outside
  the RiboDoc pipeline)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3512
tools:
- name: RiboDoc
  role: Runs the PCA and pairwise Spearman correlation analysis as statistical validations
    of replicate reproducibility within its Docker/Snakemake pipeline
  repo: https://github.com/equipeGST/RiboDoc
- name: riboWaltz
  role: Upstream P-site offset and periodicity diagnostics whose outputs feed the
    RiboDoc QC report alongside the statistical validations
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:4c9324e355f985aba89e531bcf3c58085aa4063953c5122b640b902cc47a7a44
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - BEDTools
  - DESeq2
  - RiboDoc
  - riboWaltz
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Replicate quality validation via PCA and Spearman correlation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical validation step in RiboDoc that assesses whether ribosome profiling replicates are reproducible enough for downstream differential-expression analysis, using pairwise Spearman correlation coefficients and a PCA of per-sample count vectors.

## When to use

Use when RiboSeq samples have passed footprint QC (read-length ~28 nt, metagene periodicity, CDS enrichment) and you must decide which replicates are consistent before running DESeq2 differential expression.

## When NOT to use

- Input is still raw FastQ or unaligned BAM data — the qualitative footprint QC (read-length, periodicity, CDS enrichment) must come first
- Only one replicate per condition is available, since pairwise correlation and PCA require multiple samples to compare
- Data are not ribosome-profiling counts (e.g. RNA-seq feature counts analyzed outside the RiboDoc pipeline)

## Inputs

- Merged ribosome profiling count matrix produced by htseq-count (e.g. count_matrix_by_transcript_IDs.csv)
- Per-sample alignment/count files grouped by condition and replicate (e.g. Wild_Type.56, Mutant.42)
- config.yaml with user-specified conditions (reference_condition and comparison conditions)

## Outputs

- Pairwise sample-by-sample Spearman correlation coefficient matrix/plot
- PCA plot of sample/replicate reproducibility
- Panels embedded in the QC/final report (e.g. Yeast_RiboSeq.Final_report.html)

## How to apply

After HTSeq-count produces the merged count matrix (e.g. count_matrix_by_transcript_IDs.csv), optionally standardize per-sample count vectors and run a PCA plus a pairwise sample-by-sample comparison in which Spearman's correlation coefficients are displayed between all replicates. RiboDoc assembles these two statistical panels alongside the qualitative QC (read-length distribution, metagene periodicity, CDS overrepresentation) so the user can judge replicate suitability. If a replicate clusters apart or shows low Spearman correlation with its condition siblings, treat it as low-quality, remove it, and rerun RiboDoc performing only the steps for which recalculation is required. Only replicates that pass this validation should be carried into the DESeq2 negative-binomial GLM (estimateSizeFactors, estimateDispersions, nbinomWaldTest) with the configured p-val and logFC thresholds.

## Related tools

- **RiboDoc** (Runs the PCA and pairwise Spearman correlation analysis as statistical validations of replicate reproducibility within its Docker/Snakemake pipeline) — https://github.com/equipeGST/RiboDoc
- **riboWaltz** (Upstream P-site offset and periodicity diagnostics whose outputs feed the RiboDoc QC report alongside the statistical validations) — https://github.com/LabTranslationalArchitectomics/riboWaltz

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- The report contains both a PCA plot and a pairwise Spearman correlation coefficient matrix covering every replicate
- Replicates of the same condition cluster together in the PCA and show high Spearman correlation with each other
- Samples flagged as outliers by these panels are removed and RiboDoc is rerun with only the required steps recalculated
- Final differential calls respect the config.yaml p-val (e.g. 0.01) and logFC thresholds applied only to validated replicates

## Limitations

- These are statistical validations, not footprint-quality controls: they assess replicate agreement but do not prove reads are active ribosome footprints (that requires periodicity and CDS enrichment)
- Low-complexity or poorly aligning datasets (e.g. the GSE173856 example that does not align enough material for riboWaltz) may compromise the statistical panels
- No explicit correlation cutoff is specified in the article; the accept/reject decision is left to the user's inspection of the plots

## Evidence

- [results] Pairwise sample-by-sample Spearman correlation analysis: [evidence span withheld — anchor status: not_found]

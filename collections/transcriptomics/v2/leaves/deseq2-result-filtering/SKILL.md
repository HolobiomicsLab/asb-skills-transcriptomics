---
name: deseq2-result-filtering
description: Use when a raw DESeq2 results data.frame (e.g., 32883-row Schlegel_2022_Ttr_D1_30mkg
  output) must be cleaned of NA rows and thresholded on log2FoldChange, padj, and
  baseMean before seed-match counting or deseq_fc_ecdf.
when_to_use_negative:
- Input is already a filtered, seed-match-annotated results frame — re-filtering would
  silently discard rows used as ECDF background gene sets
- The differential expression table lacks gene_id, log2FoldChange, or padj columns,
  so SeedMatchR()'s res argument cannot map seed counts onto it
- The analysis targets non-DESeq2 quantification outputs (e.g., raw count matrices)
  rather than a DESeq2 results object
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SeedMatchR
  role: Provides filter_res() to filter DESeq2 results by log2FoldChange, padj, baseMean
    and remove NA entries prior to seed-match annotation
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Produces the differential expression results data.frame that filter_res consumes
    and that SeedMatchR extends with seed-match counts
  repo: https://github.com/tacazares/SeedMatchR
- name: R
  role: Runtime environment in which filter_res() and the downstream SeedMatchR workflow
    execute
  repo: https://github.com/tacazares/SeedMatchR
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:cb63ec758fd8f834537dcffaebcf93f8a0c0e3f1bfd9c5689b110a5075bf0b83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  - ensembldb
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# deseq2-result-filtering

## Summary

Filter DESeq2 differential expression results (e.g., the Schlegel_2022_Ttr_* tables from GSE184929) with SeedMatchR::filter_res before seed-match annotation, dropping NA entries and applying log2FoldChange/padj/baseMean cutoffs so downstream ECDF/KS analyses operate on a clean results frame.

## When to use

Use when a raw DESeq2 results data.frame (e.g., 32883-row Schlegel_2022_Ttr_D1_30mkg output) must be cleaned of NA rows and thresholded on log2FoldChange, padj, and baseMean before seed-match counting or deseq_fc_ecdf.

## When NOT to use

- Input is already a filtered, seed-match-annotated results frame — re-filtering would silently discard rows used as ECDF background gene sets
- The differential expression table lacks gene_id, log2FoldChange, or padj columns, so SeedMatchR()'s res argument cannot map seed counts onto it
- The analysis targets non-DESeq2 quantification outputs (e.g., raw count matrices) rather than a DESeq2 results object

## Inputs

- DESeq2 results data.frame (e.g., Schlegel_2022_Ttr_D1_30mkg, Schlegel_2022_Ttr_D4_30mkg, Schlegel_2022_Ttr_D1_10mkg) with columns including gene_id, baseMean, log2FoldChange, padj
- Cutoff parameters: fdr_cutoff (e.g., 1) and fc_cutoff (e.g., 0)

## Outputs

- Filtered DESeq2 results data.frame (e.g., 13582 rows x 8 columns for the Ttr D1 30 mg/kg example) with NA entries removed and ready for SeedMatchR() seed-match annotation

## How to apply

Load the DESeq2 results frame (e.g., sirna.data$Schlegel_2022_Ttr_D1_30mkg from SeedMatchR::load_example_data('sirna')) and pass it to filter_res() with fdr_cutoff=1 and fc_cutoff=0, which filters on log2FoldChange, padj, and baseMean and removes NA entries (32883 rows reduced to 13582 for the D1 30 mg/kg example). Retain the columns SeedMatchR requires for DESeq2 mode: gene_id, log2FoldChange, and padj. Rationale: seed-match counts are appended per gene_id onto this filtered frame, and the ECDF/KS comparison of log2(FC) distributions needs complete padj/baseMean values; filtering first avoids spurious NA-driven failures in vcountPattern aggregation and the KS test.

## Related tools

- **SeedMatchR** (Provides filter_res() to filter DESeq2 results by log2FoldChange, padj, baseMean and remove NA entries prior to seed-match annotation) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Produces the differential expression results data.frame that filter_res consumes and that SeedMatchR extends with seed-match counts) — https://github.com/tacazares/SeedMatchR
- **R** (Runtime environment in which filter_res() and the downstream SeedMatchR workflow execute) — https://github.com/tacazares/SeedMatchR

## Examples

```
res = filter_res(res, fdr_cutoff=1, fc_cutoff=0)
```

## Evaluation signals

- Row count drops from 32883 to 13582 for Schlegel_2022_Ttr_D1_30mkg after filter_res(fdr_cutoff=1, fc_cutoff=0)
- No NA values remain in log2FoldChange, padj, or baseMean columns of the filtered frame
- Required columns gene_id, log2FoldChange, and padj are present, satisfying SeedMatchR()'s res-argument contract
- Every retained gene_id is mappable to the annodb$seqs feature names so SeedMatchR() can append mer7m8/mer8 count columns without dropped genes

## Limitations

- filter_res with fc_cutoff=0 and fdr_cutoff=1 removes only NA rows and rows with baseMean/FC below threshold; stricter cutoffs will shrink gene sets and change KS-test power
- rnor6-based workflows require canonical=FALSE in load_annotations because ensembldb::TxIsCanonicalFilter is unsupported, which affects which transcript sequences the filtered genes map to
- SeedMatchR (version 2.0.0 in the fork) requires R >= 4.3.0 and AnnotationHub >= 3.8.0 when using rnor7 annotations

## Evidence

- [methods] The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries.: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [abstract] SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches.: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [readme] The input must contain the columns gene_id, log2FoldChange, and padj.: "The input must contain the columns `gene_id`,
`log2FoldChange`, and `padj`."
- [readme] filter_res called with fdr_cutoff=1, fc_cutoff=0 in the example workflow: "res = filter_res(res, fdr_cutoff=1, fc_cutoff=0)"

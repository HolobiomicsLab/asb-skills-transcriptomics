---
name: differential-expression-results-filtering
description: Use when a DESeq2 results table (with gene_id, log2FoldChange, padj,
  baseMean) contains NA rows or unfiltered fold changes and must be cleaned before
  annotating siRNA seed matches or running deseq_fc_ecdf comparisons.
when_to_use_negative:
- Input is a raw count matrix or a summary object that has not yet been processed
  by DESeq2 into a results data frame with gene_id, log2FoldChange, and padj columns
- You only need transcriptome-wide seed match locations (res.format = 'granges' or
  full_search) and are not integrating differential expression results
- The results table has already been filtered and contains no NA values — re-filtering
  adds no information and could drop rows twice
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SeedMatchR
  role: Provides filter_res() for filtering DESeq2 results by log2FoldChange, padj,
    and baseMean, and provides the downstream SeedMatchR()/deseq_fc_ecdf() steps that
    consume the filtered table
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Produces the differential expression results data frame (log2FoldChange, padj,
    baseMean) that filter_res() cleans
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:d7d4c5d837e1f67ea64a7f90695ebfc1ad3b38ff938172a5113d9a9e8900f43f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# differential-expression-results-filtering

## Summary

Filters a DESeq2 results data frame with SeedMatchR::filter_res() on log2FoldChange, padj, and baseMean while removing NA entries, so that downstream seed-match annotation and ECDF/KS comparisons (e.g., mer7m8 targets vs background in the Ttr D1 30 mg/kg dataset) operate on a clean, comparable gene set.

## When to use

Use when a DESeq2 results table (with gene_id, log2FoldChange, padj, baseMean) contains NA rows or unfiltered fold changes and must be cleaned before annotating siRNA seed matches or running deseq_fc_ecdf comparisons.

## When NOT to use

- Input is a raw count matrix or a summary object that has not yet been processed by DESeq2 into a results data frame with gene_id, log2FoldChange, and padj columns
- You only need transcriptome-wide seed match locations (res.format = 'granges' or full_search) and are not integrating differential expression results
- The results table has already been filtered and contains no NA values — re-filtering adds no information and could drop rows twice

## Inputs

- DESeq2 results data frame with columns gene_id, log2FoldChange, padj, baseMean (e.g., sirna.data$Schlegel_2022_Ttr_D1_30mkg)
- Filtering parameters: fdr_cutoff (e.g., 1) and fc_cutoff (e.g., 0)

## Outputs

- Filtered DESeq2 results data frame (e.g., 32883 -> 13582 rows for Ttr D1 30 mg/kg) with NA entries removed, ready for SeedMatchR seed-match annotation and deseq_fc_ecdf testing

## How to apply

Run SeedMatchR::filter_res() on the DESeq2 results data frame with fdr_cutoff=1 and fc_cutoff=0, which filters on log2FoldChange, padj, and baseMean and removes NA entries — in the Schlegel_2022_Ttr_D1_30mkg example this reduces the table from 32883 rows to 13582 rows. The input must contain the columns gene_id, log2FoldChange, and padj for SeedMatchR to map seed matches to differential expression data. Filtering should happen after loading the DESeq2 results (load_example_data('sirna')) and before counting seed matches with SeedMatchR(), since the seed-match counts (mer7m8, mer8) are appended as columns to the filtered table. Rationale: the ECDF-based cumulative shift test (deseq_fc_ecdf with a one-sided KS test, alternative='greater') compares log2(FC) distributions between gene sets (e.g., mer7m8 >= 1 & mer8 == 0 vs mer7m8 == 0 & mer8 == 0), so NA-containing or unfiltered rows would bias or break the distribution comparison.

## Related tools

- **SeedMatchR** (Provides filter_res() for filtering DESeq2 results by log2FoldChange, padj, and baseMean, and provides the downstream SeedMatchR()/deseq_fc_ecdf() steps that consume the filtered table) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Produces the differential expression results data frame (log2FoldChange, padj, baseMean) that filter_res() cleans)

## Examples

```
res = filter_res(res, fdr_cutoff=1, fc_cutoff=0)
```

## Evaluation signals

- Row count of the filtered table matches expectation for the given cutoffs (32883 -> 13582 rows for Schlegel_2022_Ttr_D1_30mkg with fdr_cutoff=1, fc_cutoff=0)
- No NA values remain in log2FoldChange, padj, or baseMean columns of the filtered data frame
- Filtered table retains the required gene_id, log2FoldChange, and padj columns
- Downstream SeedMatchR() call succeeds in appending a mer7m8 count column to the filtered table (e.g., Ttr gene ENSRNOG00000016275 with log2FC ~ -8.16 and mer7m8 = 1)
- deseq_fc_ecdf gene sets (mer7m8 >= 1 & mer8 == 0 vs mer7m8 == 0 & mer8 == 0 background) are non-empty and yield a finite KS D statistic and p-value

## Limitations

- filter_res() only filters on log2FoldChange, padj, baseMean and NA removal — no support for additional custom column filters
- If input sequence names are transcript IDs rather than gene IDs, tx.id.col must be set to TRUE for matches to be summarized to gene level; gene-level filtering assumes gene_id keys
- The SeedMatchR function requires specific input columns (gene_id, log2FoldChange, padj); other result formats will not map correctly

## Evidence

- [readme] The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries.: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [readme] Input must contain required DESeq2 columns: [evidence span withheld — anchor status: external_doc]
- [abstract] SeedMatchR extends DESeq2 by annotating results with predicted seed matches: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [methods] ECDF + KS testing rationale for filtered gene sets: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test."

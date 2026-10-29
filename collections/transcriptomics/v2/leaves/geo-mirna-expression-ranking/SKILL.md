---
name: geo-mirna-expression-ranking
description: Use when you need to identify the most abundantly expressed endogenous
  miRNAs from a GEO expression accession (e.g., GSE172269 rat liver) so their predicted
  targets can be tested for cumulative expression shifts against a background in DESeq2
  results.
when_to_use_negative:
- You already have a specific siRNA guide sequence (e.g., the 23 bp D1 Ttr guide 'UUAUAGAGCAAGAACACUGUUUU')
  — use SeedMatchR seed matching (get_seed/SeedMatchR/full_search) rather than miRNA
  expression-derived target lists
- The differential expression results and miRNA expression data come from different
  species or tissues, so the target-to-gene mapping via rnor6 annotations would be
  invalid
- You have raw RNA-seq reads rather than a processed expression/DESeq2 results table
  — deseq_fc_ecdf and filter_res require an already-quantified results data frame
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3308
tools:
- name: SeedMatchR
  role: Provides load_annotations (rnor6, canonical=FALSE, min.feature.width=8, longest.utr=TRUE)
    for transcript-to-gene mapping, filter_res for filtering DESeq2 results, and deseq_fc_ecdf
    for the KS/ECDF comparison of miRNA-target vs background log2FC distributions
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Produces the differential expression results data frame (log2FoldChange, padj,
    baseMean) on which the miRNA-target versus background ECDF analysis is run
- name: AnnotationHub
  role: Supplies the rat Rnor_v6.0 (rnor6) GTF and DNA 2bit resources (AH93578, AH95846)
    behind SeedMatchR::load_annotations used to map miRDB target transcripts to gene
    IDs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:11f6426864f78365718c1d746cdd9498fe7680d3b39ff5871b5afc98833eabb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# geo-mirna-expression-ranking

## Summary

Ranks miRNAs from a GEO-deposited expression dataset (e.g., rat liver miRNA expression from GSE172269) by abundance to select the top-expressed miRNAs, whose miRDB-predicted high-confidence targets then serve as gene lists for SeedMatchR ECDF/KS testing of miRNA pathway perturbation.

## When to use

Use when you need to identify the most abundantly expressed endogenous miRNAs from a GEO expression accession (e.g., GSE172269 rat liver) so their predicted targets can be tested for cumulative expression shifts against a background in DESeq2 results.

## When NOT to use

- You already have a specific siRNA guide sequence (e.g., the 23 bp D1 Ttr guide 'UUAUAGAGCAAGAACACUGUUUU') — use SeedMatchR seed matching (get_seed/SeedMatchR/full_search) rather than miRNA expression-derived target lists
- The differential expression results and miRNA expression data come from different species or tissues, so the target-to-gene mapping via rnor6 annotations would be invalid
- You have raw RNA-seq reads rather than a processed expression/DESeq2 results table — deseq_fc_ecdf and filter_res require an already-quantified results data frame

## Inputs

- GEO miRNA expression matrix from a public accession (e.g., GSE172269 rat liver miRNA expression)
- DESeq2 results data frame with gene_id, log2FoldChange, and padj columns (e.g., Schlegel_2022_Ttr_D1_10mkg)
- miRDB target predictions for the selected miRNAs
- SeedMatchR species annotations (rnor6 gene/transcript mappings from AnnotationHub)

## Outputs

- Ranked list of top-10 most expressed miRNAs
- High-confidence miRDB target gene list (score >= 90) mapped to rat gene IDs, plus background gene list
- KS test statistic and p-value exported to mirna_ecdf_ks_results.csv
- ECDF plot (Fig. 1F style) saved as mirna_target_ecdf.png with Background and miRNA-target curves

## How to apply

Download the miRNA expression data from the GEO accession (e.g., GSE172269) and rank miRNAs by expression, keeping the top 10 most expressed (rat liver in the reference workflow). Fetch miRDB target predictions for those miRNAs and retain only high-confidence targets with miRDB score >= 90, then map predicted target transcripts to rat gene IDs using SeedMatchR::load_annotations (reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE) so transcript-to-gene mappings are consistent with the annotation set. Load the matched differential expression results (e.g., Schlegel_2022_Ttr_D1_10mkg from get_example_data/load_example_data), filter with SeedMatchR::filter_res on log2FoldChange, padj, and baseMean with NA removal, and split genes into miRNA-target and background (non-target) lists. Run SeedMatchR::deseq_fc_ecdf with stats_test='KS' and alternative='greater' to test whether the log2(Fold Change) distribution of predicted miRNA targets is positively shifted relative to background.

## Related tools

- **SeedMatchR** (Provides load_annotations (rnor6, canonical=FALSE, min.feature.width=8, longest.utr=TRUE) for transcript-to-gene mapping, filter_res for filtering DESeq2 results, and deseq_fc_ecdf for the KS/ECDF comparison of miRNA-target vs background log2FC distributions) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Produces the differential expression results data frame (log2FoldChange, padj, baseMean) on which the miRNA-target versus background ECDF analysis is run)
- **AnnotationHub** (Supplies the rat Rnor_v6.0 (rnor6) GTF and DNA 2bit resources (AH93578, AH95846) behind SeedMatchR::load_annotations used to map miRDB target transcripts to gene IDs)

## Examples

```
ecdf.results = deseq_fc_ecdf(res, list("Background" = background.list, "miRNA.targets" = target.list), stats_test='KS', alternative='greater')
```

## Evaluation signals

- The top-10 miRNA list is ranked by expression in the GSE172269 liver data and matches the top of the ranked abundance table
- Every retained miRDB target has score >= 90 and maps to a valid rat gene ID via the rnor6 annotations
- The KS test (alternative='greater') exported to mirna_ecdf_ks_results.csv indicates a statistically significant positive shift of the miRNA-target log2FC distribution relative to background
- mirna_target_ecdf.png renders with Background and miRNA-target ECDF curves clearly separated (Fig. 1F style)
- Filtering with filter_res reduces the raw DESeq2 table (e.g., 32883 rows for Schlegel_2022 results) and removes NA entries before the ECDF test

## Limitations

- miRDB predictions are in-silico; the score >= 90 cutoff is used to limit false positives but the target set remains predicted rather than experimentally validated
- rnor6 and rnor7 cannot be used with ensembldb::TxIsCanonicalFilter or TxSupportLevelFilter — canonical must be FALSE and transcript.support should not be passed for rat transcriptomes
- Selecting only the top-10 most expressed miRNAs assumes abundant miRNAs dominate the observed perturbation of endogenous miRNA pathway targets; lower-abundance miRNAs are excluded
- rnor7 requires R 4.3.0 or greater with AnnotationHub >= 3.8.0

## Evidence

- [other] Retrieve rat liver miRNA expression data from GEO accession GSE172269 and rank miRNAs by expression to select the top 10 most expressed liver miRNAs.: [evidence span withheld — anchor status: not_found]
- [other] Fetch miRDB target predictions for those 10 miRNAs, retaining only high-confidence targets with miRDB score >= 90, and map predicted target transcripts to rat gene IDs.: [evidence span withheld — anchor status: not_found]
- [readme] Many factors that perturb gene expression, like miRNA, show cumulative changes in their targets gene expression.: "Many factors that perturb gene expression, like miRNA, show cumulative changes in their targets gene expression."
- [methods] The function deseq_fc_ecdf is designed to work directly with a DESeq2 results data frame.: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [readme] rnor6 and mm10 cannot be used with the ensembldb::TxIsCanonicalFilter therefore the canonical argument for SeedMatchR::load_annotations must be set to FALSE.: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter` therefore the `canonical` argument for `SeedMatchR::load_annotations` must be set to `FALSE`."

---
name: mirna-target-prediction-integration
description: Use when you have filtered DESeq2 results (e.g., Schlegel_2022_Ttr_D1_10mkg)
  plus a set of predicted miRNA targets and want to test whether the target log2FC
  distribution shifts positively relative to background genes.
when_to_use_negative:
- The goal is to quantify seed matches in transcripts for a specific siRNA guide sequence
  (e.g., 'mer7m8' counting with SeedMatchR()/full_search()) rather than test cumulative
  target-gene shifts via ECDF.
- The input is raw counts rather than a DESeq2 results data frame — deseq_fc_ecdf
  is designed to work directly with a DESeq2 results data frame and will not accept
  other formats.
- Working with rnor7 annotations where transcript.support filtering or TxSupportLevelFilter
  would be attempted; rnor6/rnor7 cannot use ensembldb::TxSupportLevelFilter, and
  rnor6 additionally cannot use TxIsCanonicalFilter.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SeedMatchR
  role: Provides load_annotations, filter_res, and deseq_fc_ecdf for comparing log2(Fold
    Change) distributions of miRNA-target vs background gene sets with an ECDF/KS
    test
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Source of the differential expression results (log2FoldChange, padj, baseMean
    per gene_id) consumed by deseq_fc_ecdf
- name: AnnotationHub
  role: Supplies the rnor6 2bit/ensembldb resources (AH93578 / AH95846) that load_annotations
    uses to build GTF mappings and 3UTR sequences for transcript-to-gene mapping
- name: R
  role: Runtime environment for the SeedMatchR workflow (R >= 4.3.0 required for rnor7/AnnotationHub
    >= 3.8.0)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:f2af6d3ac47f200301a4545ae6141e35e4b2671b6cf1f0ccf06a47c73bc605cd
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mirna-target-prediction-integration

## Summary

Integrates high-confidence miRNA target predictions (miRDB score ≥ 90) with DESeq2 differential expression results and SeedMatchR's ECDF/KS analysis to test whether a perturbation (e.g., siRNA treatment) measurably shifts endogenous miRNA pathway target gene expression.

## When to use

Use when you have filtered DESeq2 results (e.g., Schlegel_2022_Ttr_D1_10mkg) plus a set of predicted miRNA targets and want to test whether the target log2FC distribution shifts positively relative to background genes.

## When NOT to use

- The goal is to quantify seed matches in transcripts for a specific siRNA guide sequence (e.g., 'mer7m8' counting with SeedMatchR()/full_search()) rather than test cumulative target-gene shifts via ECDF.
- The input is raw counts rather than a DESeq2 results data frame — deseq_fc_ecdf is designed to work directly with a DESeq2 results data frame and will not accept other formats.
- Working with rnor7 annotations where transcript.support filtering or TxSupportLevelFilter would be attempted; rnor6/rnor7 cannot use ensembldb::TxSupportLevelFilter, and rnor6 additionally cannot use TxIsCanonicalFilter.

## Inputs

- DESeq2 results data frame (e.g., Schlegel_2022_Ttr_D1_10mkg) with columns gene_id, log2FoldChange, padj, baseMean
- Rat (rnor6) gene/transcript annotations loaded via SeedMatchR::load_annotations
- miRNA expression matrix from GEO accession GSE172269 (rat liver)
- miRDB target predictions for the top 10 expressed liver miRNAs (score >= 90)

## Outputs

- KS test statistic and p-value table exported as mirna_ecdf_ks_results.csv
- ECDF plot of log2FC distributions for Background vs miRNA-target gene sets (mirna_target_ecdf.png, Fig. 1F style)

## How to apply

Load rat annotations via SeedMatchR::load_annotations('rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE) to map transcript-level predictions to gene IDs (rnor6 cannot use ensembldb::TxIsCanonicalFilter, so canonical must be FALSE). Download the Schlegel 2022 DESeq2 results with SeedMatchR::get_example_data / load_example_data (dataset Schlegel_2022_Ttr_D1_10mkg) and filter with filter_res() on log2FoldChange, padj, baseMean, removing NAs. Retrieve rat liver miRNA expression from GEO accession GSE172269, rank miRNAs by expression, and keep the top 10 most expressed liver miRNAs; fetch miRDB target predictions for these, retaining only high-confidence targets with score >= 90, and map predicted target transcripts to rat gene IDs. Run SeedMatchR::deseq_fc_ecdf on the filtered DESeq2 results with two gene lists — predicted miRNA targets and non-target background genes — using stats_test='KS' with alternative='greater', since miRNA-like perturbations manifest as cumulative shifts in target gene expression. Export the KS statistic and p-value (e.g., mirna_ecdf_ks_results.csv) and save the ECDF plot comparing the Background and miRNA-target curves.

## Related tools

- **SeedMatchR** (Provides load_annotations, filter_res, and deseq_fc_ecdf for comparing log2(Fold Change) distributions of miRNA-target vs background gene sets with an ECDF/KS test) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Source of the differential expression results (log2FoldChange, padj, baseMean per gene_id) consumed by deseq_fc_ecdf)
- **AnnotationHub** (Supplies the rnor6 2bit/ensembldb resources (AH93578 / AH95846) that load_annotations uses to build GTF mappings and 3UTR sequences for transcript-to-gene mapping)
- **R** (Runtime environment for the SeedMatchR workflow (R >= 4.3.0 required for rnor7/AnnotationHub >= 3.8.0))

## Examples

```
ecdf.results = deseq_fc_ecdf(res, list("Background"=background.list, "miRNA.targets"=target.list), stats_test="KS", alternative="greater"); write.csv(ecdf.results$stat, "mirna_ecdf_ks_results.csv")
```

## Evaluation signals

- KS test with alternative='greater' shows a statistically significant positive shift of the miRNA-target log2FC distribution relative to background
- ECDF plot renders with Background and miRNA-target curves clearly separated
- Filtered results retain expected schema: gene_id, log2FoldChange, padj, baseMean with no NA entries (e.g., 32883 rows reduced after filter_res)
- miRDB predictions restricted to high-confidence score >= 90 and successfully mapped to rat gene IDs via rnor6 annotations
- KS statistic and p-value exported to mirna_ecdf_ks_results.csv and the ECDF figure saved as mirna_target_ecdf.png

## Limitations

- rnor6 and mm10 cannot be used with ensembldb::TxIsCanonicalFilter, so the canonical argument of load_annotations must be FALSE; rnor6/rnor7 also cannot use TxSupportLevelFilter
- rnor7 requires R >= 4.3.0 and AnnotationHub >= 3.8.0; loading annotations takes several minutes and should be cached as Rdata for repeated use
- The miRNA target-shift signal is a perturbation indicator, not proof of seed-mediated off-targeting; interpretation depends on the confidence of miRDB predictions (score >= 90) and the miRNA expression ranking

## Evidence

- [methods] deseq_fc_ecdf input schema: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [methods] filter_res usage: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [intro] rnor6 canonical filter restriction: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter` therefore the `canonical` argument for `SeedMatchR::load_annotations` must be set to `FALSE`."
- [readme] miRNA cumulative-change rationale and ECDF/KS framing: "Many factors that perturb gene expression, like miRNA, show cumulative changes in their targets gene expression. Cumulative changes in the profile of genes expression can be visualized and tested"
- [readme] load_annotations rat parameters: "annodb = load_annotations(reference.name = "rnor6", canonical = FALSE, min.feature.width = 8, longest.utr = T)"

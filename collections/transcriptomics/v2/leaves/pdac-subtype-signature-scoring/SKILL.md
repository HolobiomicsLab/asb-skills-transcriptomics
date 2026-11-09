---
name: pdac-subtype-signature-scoring
description: Use when comparing curated PDAC subtype signatures across matched/paired
  samples (primary vs metastatic vs cell line) or platforms (RNAseq vs microarray)
  to quantify signature retention or loss.
when_to_use_negative:
- When the goal is assigning individual tumors to a subtype rather than scoring signature
  retention/loss across paired samples — use the PurIST single-sample classifier instead
- 'When the input is single-cell data: pdacR datasets are bulk transcriptomics and
  the companion scOh atlas (rmoffitt/scOh) is a separate resource without a gene-set
  scoring web interface'
- 'When trying to merge or batch-correct across many platforms: the article notes
  the inability to robustly merge datasets across platforms and compositions, so comparisons
  are done within each platform separately'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: R package providing the parsed PACA-AU expression datasets, sampInfo metadata,
    and curated gene signature lists used for signature scoring
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Language in which the z-scoring, mean-centered gene set scoring, and paired-sample
    comparison are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:3b360e568fb0d4eb66b5cc2f60aba4ac96c5e96142bc6c89737b2cd7934e40e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - pdacR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pdac-subtype-signature-scoring

## Summary

Scores PDAC subtype gene signatures (e.g., Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) by per-sample z-scoring of signature genes and computing mean-centered gene set scores, to test whether signatures are retained or lost across paired primary tumors, metastases, and derived cell lines in the pdacR-curated PACA-AU datasets.

## When to use

Use when comparing curated PDAC subtype signatures across matched/paired samples (primary vs metastatic vs cell line) or platforms (RNAseq vs microarray) to quantify signature retention or loss.

## When NOT to use

- When the goal is assigning individual tumors to a subtype rather than scoring signature retention/loss across paired samples — use the PurIST single-sample classifier instead
- When the input is single-cell data: pdacR datasets are bulk transcriptomics and the companion scOh atlas (rmoffitt/scOh) is a separate resource without a gene-set scoring web interface
- When trying to merge or batch-correct across many platforms: the article notes the inability to robustly merge datasets across platforms and compositions, so comparisons are done within each platform separately

## Inputs

- Parsed ICGC PACA-AU RNAseq expression matrix (genes x samples) with sampInfo metadata (.rds objects in pdacR Data/)
- Parsed ICGC PACA-AU microarray expression matrix with sampInfo metadata (.rds objects in pdacR Data/)
- pdacR gene_lists object (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine signatures from R/parse_gene_lists.R)
- Sample pairing labels from sampInfo (primary, metastatic, cell line), excluding ICGC_PARSE_Nones-flagged samples

## Outputs

- icgc_gene_set_scores.csv: per-sample table (samples x 3 signatures) with class labels (primary/metastatic/cell line) and platform (RNAseq/microarray)
- Cross-platform comparison of mean-centered signature scores across paired sample classes

## How to apply

Load the parsed ICGC PACA-AU RNAseq and microarray expression matrices (genes x samples) with their matching sampInfo from the pdacR Data/ directory, which holds the parsed datasets and the gene signatures object populated by R/parse_gene_lists.R. Identify paired samples (primary-primary, primary-metastatic, primary-cell line) within and between the two PACA-AU platforms using sampInfo metadata, excluding samples flagged by ICGC_PARSE_Nones. For each sample, z-score the genes belonging to each signature (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) and compute the signature score as the mean of the z-scored signature genes (mean-centered gene set score). Compare scores across paired sample classes on both platforms to determine whether signatures drop in derived cell lines and metastases relative to matched primaries, then write the per-sample score table (samples x 3 signatures with class labels and platform) to icgc_gene_set_scores.csv. The mean-of-z-scores approach is chosen because it gives a simple, platform-comparable single-sample score without needing cross-platform batch correction.

## Related tools

- **pdacR** (R package providing the parsed PACA-AU expression datasets, sampInfo metadata, and curated gene signature lists used for signature scoring) — https://github.com/rmoffitt/pdacR.git
- **R** (Language in which the z-scoring, mean-centered gene set scoring, and paired-sample comparison are implemented)

## Examples

```
devtools::install_github("rmoffitt/pdacR")
```

## Evaluation signals

- Each of the three signatures (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) shows a clear reduction in mean-centered score in derived cell lines and metastatic samples versus their paired primary tumors
- The reduction pattern is reproduced consistently on both PACA-AU platforms (RNAseq and microarray), consistent with the reported Fig. 2c result
- The output CSV contains a score for all three signatures for every analyzed sample, with correct class labels and platform annotations
- Samples flagged by ICGC_PARSE_Nones are excluded from the per-sample score table
- Scores are mean-centered (mean of z-scored signature genes), so within-sample scores center near zero across the transcriptome

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signature scores may reflect admixed TME signal (e.g., immune content of the Immunogenic signature)
- Datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding complexity; scores are compared within rather than across platforms
- The microarray and RNAseq PACA-AU pairs must be matched via sampInfo metadata; missing or mis-annotated sample labels will break pairing

## Evidence

- [results] In paired PACA-AU samples, the Bailey ADEX, Bailey Immunogenic, and Collisson exocrine gene signatures are lost in cell lines derived from primary PDAC tumors that express these signatures: [evidence span withheld — anchor status: not_found]
- [supplementary] For each dataset, z-score signature genes per sample and compute mean-centered gene set scores (mean of the z-scored signature genes) for each of the three signatures.: [evidence span withheld — anchor status: not_found]
- [intro] We developed an R package, termed pdacR, for public use: "we have created an R package, termed pdacR, for public use"
- [readme] Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object.: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
- [discussion] Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"

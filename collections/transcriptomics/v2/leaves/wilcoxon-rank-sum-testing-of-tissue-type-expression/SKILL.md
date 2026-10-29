---
name: wilcoxon-rank-sum-testing-of-tissue-type-expression
description: Use when you have bulk RNA-seq or microarray samples from multiple pancreatic
  compartments (normal pancreas, primary PDAC, PDX, CAF lines, metastasis) and need
  to test which tissue types drive a curated gene signature's expression.
when_to_use_negative:
- Input is single-cell data (e.g., the rmoffitt/scOh atlas) — this skill is defined
  for bulk transcriptomics, where the article notes 'we are limited in our ability
  to fully separate cellular populations'; use the single-cell classifier instead
- Samples must be merged across platforms (bulk RNA-seq + microarrays) into one matrix
  — the article explicitly found data aggregation 'introduced more problems than it
  solved'
- The goal is survival or therapy-response prediction rather than two-group expression
  comparison — use PurIST subtyping and the survival/Cox tabs instead
edam_operation: http://edamontology.org/operation_3793
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: Supplies the parsed Moffitt 2015 microarray and RNAseq expression matrices,
    sampInfo tissue-of-origin tracks, and curated gene signature lists used as test
    inputs
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Language in which the signature scoring (z-score + mean) and Wilcoxon rank-sum
    tests are implemented
- name: devtools
  role: Used to install pdacR from GitHub to access the datasets and signature objects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:52a0f802f60e1bae34e199db85be7a50c97d98673e3aa3f02d85f486cdab368d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_3793
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# wilcoxon-rank-sum-testing-of-tissue-type-expression

## Summary

Compares gene-signature scores (e.g., ADEX, Exocrine, Immunogenic) between tissue/cell-type groups in bulk PDAC expression datasets using Wilcoxon rank-sum tests to determine whether a signature is tissue-of-origin intrinsic or tumor-cell intrinsic, as in the Moffitt 2015 normal-pancreas-vs-PDAC analysis.

## When to use

Use when you have bulk RNA-seq or microarray samples from multiple pancreatic compartments (normal pancreas, primary PDAC, PDX, CAF lines, metastasis) and need to test which tissue types drive a curated gene signature's expression.

## When NOT to use

- Input is single-cell data (e.g., the rmoffitt/scOh atlas) — this skill is defined for bulk transcriptomics, where the article notes 'we are limited in our ability to fully separate cellular populations'; use the single-cell classifier instead
- Samples must be merged across platforms (bulk RNA-seq + microarrays) into one matrix — the article explicitly found data aggregation 'introduced more problems than it solved'
- The goal is survival or therapy-response prediction rather than two-group expression comparison — use PurIST subtyping and the survival/Cox tabs instead

## Inputs

- Parsed Moffitt 2015 microarray and RNAseq expression datasets (.RData objects from pdacR data/, exposed as $ex genes-by-samples matrices)
- Sample-level metadata (sampInfo) annotating tissue of origin: primary PDAC, PDX, CAF line, cell line, normal pancreas, metastasis
- Curated gene signature lists (ADEX, Exocrine, Immunogenic) as gene SYMBOLs from R/parse_gene_lists.R

## Outputs

- Per-sample signature scores with Wilcoxon statistics and p-values written to adex_exocrine_immunogenic_scores.csv
- Grouped boxplots of signature scores by tissue type
- Significance calls (p < 0.0001) comparing normal pancreas vs PDAC and primary PDAC vs CAF lines in each platform

## How to apply

Load the parsed .RData expression datasets and sampInfo metadata from the pdacR data/ directory, keeping microarray and RNAseq datasets separate because cross-platform aggregation was found to 'introduce more problems than it solved'. Load the curated signature lists from R/parse_gene_lists.R (ADEX, Exocrine, Immunogenic), verifying identifiers are gene SYMBOLs that match the $ex genes-by-samples matrix. Z-score each gene with mean-centering/scaling, then compute each per-sample signature score as the mean scaled expression of that signature's genes. Group samples by their tissue-of-origin sample track and run Wilcoxon rank-sum tests per signature per dataset: normal pancreas vs PDAC tumors, and primary PDAC vs CAF lines. Accept the result only if normal pancreata and primaries both show significantly higher scores at p < 0.0001 in BOTH the microarray and RNAseq datasets.

## Related tools

- **pdacR** (Supplies the parsed Moffitt 2015 microarray and RNAseq expression matrices, sampInfo tissue-of-origin tracks, and curated gene signature lists used as test inputs) — github.com/rmoffitt/pdacR
- **R** (Language in which the signature scoring (z-score + mean) and Wilcoxon rank-sum tests are implemented)
- **devtools** (Used to install pdacR from GitHub to access the datasets and signature objects)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); scores <- colMeans(t(scale(ex[adex_genes, ]))); wilcox.test(scores ~ tissue, subset = tissue %in% c("normal_pancreas", "primary_PDAC"))
```

## Evaluation signals

- Normal pancreata show significantly higher ADEX/Exocrine/Immunogenic scores than PDACs at p < 0.0001 in both the microarray and RNAseq datasets
- Primary PDACs show higher scores than CAF lines at p < 0.0001 in both platforms
- Signature gene identifiers are gene SYMBOLs that match rownames of the $ex matrix — no unmatched genes silently dropped
- Per-sample scores equal the mean of z-scored genes (per-gene centering check), and microarray and RNAseq scores are computed and reported separately, never pooled
- adex_exocrine_immunogenic_scores.csv contains every sample with its tissue-of-origin annotation, including non-tumor compartments (PDX, CAF lines, normal pancreas, metastasis)

## Limitations

- Bulk analyses cannot fully separate cellular populations, so elevated non-tumor signature scores in tumors may reflect stromal/exocrine/immune admixture rather than expression origin
- Datasets cannot be robustly merged across platforms (RNA-seq vs microarray); batch correction/aggregation was removed because it 'introduced more problems than it solved'
- Non-adenocarcinoma or low-purity samples (e.g., 32 of 182 TCGA-PAAD samples are non-PDAC) must be filtered via sample tracks or they introduce non-specificity into signature interpretation
- Validation is platform-dependent — a conclusion holds only if the p < 0.0001 directionality replicates in both the microarray and RNAseq Moffitt datasets

## Evidence

- [readme] pdacR houses parsed datasets and signature objects: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object. These datasets are used both in the app and in the [associated publication Torre-Healy et al. 2023]"
- [methods] installation command for the R package: "To install the package, simply run
```
devtools::install_github("rmoffitt/pdacR")
```"
- [discussion] bulk data limitation for cell-type separation: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
- [readme] sample filtering is required for accurate analysis: "Checking a box in this list will remove all samples with that label from all analyses. This is important to do to ensure accruate analyses."
- [discussion] package purpose rationale: "we developed an R package termed pdacR that houses carefully organized and annotated public PDAC transcriptomics data for ease of access and heightened transparency"

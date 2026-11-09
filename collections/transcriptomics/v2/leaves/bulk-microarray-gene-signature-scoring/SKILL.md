---
name: bulk-microarray-gene-signature-scoring
description: Use when you need to compute a gene-signature score per sample from a
  bulk microarray expression matrix (e.g., the 309 FFPE primary PDAC samples in Puleo
  2018) and correlate it against sample-level metadata such as predicted mean VAF
  to test purity dependence.
when_to_use_negative:
- Input is single-cell RNA-seq data or a scRNA atlas (e.g., rmoffitt/scOh Seurat objects)
  — signature-to-purity correlation of this form applies to bulk transcriptomics
- The gene signature list cannot be matched at the SYMBOL level to the expression
  matrix rows (mismatched identifier spaces)
- The dataset has not been filtered to histologically confirmed PDAC samples (e.g.,
  using TCGA-PAAD's 'Decision' = 'whitelist' filter), since non-adenocarcinoma cases
  introduce non-specificity
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: Supplies parse_Puleo_Array to load the parsed Puleo 2018 microarray dataset
    and the curated gene signature lists (gene_lists) and PurIST classifier objects
    from its Data/ directory
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Runtime environment in which the signature scoring and cor.test(method = 'spearman')
    correlation are performed
  repo: https://www.nature.com/articles/s42003-023-04461-6
- name: ggplot2
  role: Renders the scatter plot of VAF versus exocrine signature score with a fitted
    trend line (Fig. 2b reproduction)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:e9e616bb8ccfe79a8509ecede4b5ead866a32d5117f3725e72c4c8d0c7d9681b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bulk-microarray-gene-signature-scoring

## Summary

Scores a curated gene signature (e.g., Collisson Exocrine) as the mean expression of its member genes across bulk microarray samples, then tests association of the score with sample-level metadata such as predicted VAF used as a tumor-purity proxy. This operationalizes the pdacR curation ethos: signature-to-purity relationships must be checked before signatures are reused across datasets.

## When to use

Use when you need to compute a gene-signature score per sample from a bulk microarray expression matrix (e.g., the 309 FFPE primary PDAC samples in Puleo 2018) and correlate it against sample-level metadata such as predicted mean VAF to test purity dependence.

## When NOT to use

- Input is single-cell RNA-seq data or a scRNA atlas (e.g., rmoffitt/scOh Seurat objects) — signature-to-purity correlation of this form applies to bulk transcriptomics
- The gene signature list cannot be matched at the SYMBOL level to the expression matrix rows (mismatched identifier spaces)
- The dataset has not been filtered to histologically confirmed PDAC samples (e.g., using TCGA-PAAD's 'Decision' = 'whitelist' filter), since non-adenocarcinoma cases introduce non-specificity

## Inputs

- Parsed bulk microarray expression matrix ($ex) from a pdacR dataset object (e.g., Puleo 2018)
- Curated gene signature lists (gene_lists) shipped in pdacR's Data/ directory, matched at the SYMBOL level
- Sample-level metadata ($sampInfo) containing predicted mean variant allele frequency (VAF) per sample

## Outputs

- Per-sample gene-signature score (mean expression of signature genes)
- Spearman correlation estimate and p-value (expected: rho = -0.221, p = 0.0003 for Collisson Exocrine vs. VAF in Puleo 2018)
- ggplot2 scatter plot of VAF versus signature score with fitted trend line (reproducing Fig. 2b / Supplementary Fig. 1A)

## How to apply

Load a parsed dataset via pdacR's parse_Puleo_Array and the curated gene signature lists (gene_lists) from the package's Data/ directory. Extract the signature's gene symbols by matching at the SYMBOL level, subset the $ex expression matrix to the sample cohort of interest (309 FFPE primary PDAC for Puleo 2018), and score each sample as the mean expression of the signature's genes. Pull the per-sample predicted mean VAF from $sampInfo as the tumor cellularity proxy and run cor.test(method = "spearman") between signature score and VAF. Render a scatter plot of VAF versus signature score with a fitted ggplot2 trend line for comparison against published figures (Fig. 2b / Supplementary Fig. 1A). Sample filtering matters: pdacR's authors stress that unchecked samples (e.g., the 32 non-PDAC cases among TCGA-PAAD's 182) can bias signature analyses, so confirm the cohort is restricted to the annotated whitelist before scoring.

## Related tools

- **pdacR** (Supplies parse_Puleo_Array to load the parsed Puleo 2018 microarray dataset and the curated gene signature lists (gene_lists) and PurIST classifier objects from its Data/ directory) — github.com/rmoffitt/pdacR
- **R** (Runtime environment in which the signature scoring and cor.test(method = 'spearman') correlation are performed) — https://www.nature.com/articles/s42003-023-04461-6
- **ggplot2** (Renders the scatter plot of VAF versus exocrine signature score with a fitted trend line (Fig. 2b reproduction))

## Examples

```
ct <- cor.test(exocrine_score, sampInfo$pred_vaf, method = "spearman")
```

## Evaluation signals

- Spearman rho for Collisson Exocrine score vs. predicted mean VAF in Puleo 2018 equals approximately -0.221 with p = 0.0003
- Signature score is computed as the mean expression of the signature's genes from the $ex matrix, restricted to the 309 FFPE primary PDAC samples
- Scatter plot with fitted trend line visually reproduces Fig. 2b and cross-checks against Supplementary Fig. 1A
- Gene-to-probe matching succeeds at the SYMBOL level with no unmatched signature genes dropped silently
- Cohort metadata confirms no non-PDAC or low-quality samples remain in the scored set (cf. TCGA-PAAD 182 -> 150 whitelist-style filtering)

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signature scores confound tumor cells with fibroblast, endocrine, exocrine, and immune signals
- Predicted mean VAF is only a proxy for tumor cellularity, not a direct purity measurement
- Datasets span multiple platforms (bulk RNA-seq and microarrays), so scores are not robustly comparable across datasets without aggregation/batch correction, which pdacR's authors removed because it 'introduced more problems than it solved'

## Evidence

- [abstract] Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases.: "Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases."
- [methods] Score each sample's exocrine signature as the mean expression of its genes from the $ex matrix: [evidence span withheld — anchor status: not_found]
- [methods] Compute the Spearman rank correlation between the exocrine signature score and predicted mean VAF using cor.test(method = "spearman"): [evidence span withheld — anchor status: not_found]
- [readme] These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots.: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [discussion] Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"

---
name: geo-transcriptomic-clinical-data-harmonization
description: Use when a GEO accession (e.g. GSE78220) provides a raw/processed transcriptome
  matrix plus separate clinical or treatment-response metadata that must be aligned
  per sample before survival or responder-vs-non-responder analysis.
when_to_use_negative:
- The input is already a harmonized expression-plus-clinical table with gene-symbol
  rows and sample columns
- The GEO series lacks paired per-sample clinical/response metadata, so no survival
  or responder stratification can be performed
- Analysis targets single-cell data rather than bulk transcriptome matrices (e.g.
  GSE81861/GSE75688 used only for DCNet validation)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Language used (with GEOparse) to download the GEO series and reshape/join
    the expression and clinical tables
  repo: https://github.com/xindd/DCNet
- name: DCNet
  role: Downstream consumer of the harmonized matrix, which expects each column to
    be a sample and each row a human gene symbol
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:26b8d40e12fde4b29b7298dee68f629286387d222c7ab4289848913c999dbbf8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# geo-transcriptomic-clinical-data-harmonization

## Summary

Harmonizes GEO-deposited bulk transcriptome matrices with their paired clinical/response metadata into a sample-by-gene analytical table, as done for the 38 pre-treatment melanoma biopsies of GSE78220 used in the DCNet immunotherapy-response analyses.

## When to use

Use when a GEO accession (e.g. GSE78220) provides a raw/processed transcriptome matrix plus separate clinical or treatment-response metadata that must be aligned per sample before survival or responder-vs-non-responder analysis.

## When NOT to use

- The input is already a harmonized expression-plus-clinical table with gene-symbol rows and sample columns
- The GEO series lacks paired per-sample clinical/response metadata, so no survival or responder stratification can be performed
- Analysis targets single-cell data rather than bulk transcriptome matrices (e.g. GSE81861/GSE75688 used only for DCNet validation)

## Inputs

- GEO accession series (e.g. GSE78220) containing bulk RNA-seq/transcriptome expression data
- Corresponding clinical metadata: overall survival times and anti-PD-1 responder/non-responder labels per biopsy specimen

## Outputs

- Sample-by-gene expression matrix (rows = human gene symbols, columns = samples) with log2(x+1)-transformed values
- Per-sample clinical annotation table (survival time, response status) aligned to expression sample IDs

## How to apply

Download the GEO series (e.g. GSE78220, transcriptome samples and corresponding clinical data of 38 melanoma biopsy specimens) with GEOparse in Python. Reshape the expression matrix so rows are human gene symbols and columns are samples, matching the DCNet input convention, and apply the recommended log2(x+1) transformation to expression values. Join each sample column to its clinical record (overall survival, anti-PD-1 responder/non-responder status) via sample IDs, dropping samples lacking paired clinical annotations. The harmonized matrix is then ready for downstream steps such as per-sample risk-score computation over 33 prognostic genes, median-split Kaplan-Meier/log-rank stratification, and Wilcoxon comparison of risk scores between responders and non-responders.

## Related tools

- **Python** (Language used (with GEOparse) to download the GEO series and reshape/join the expression and clinical tables) — https://github.com/xindd/DCNet
- **DCNet** (Downstream consumer of the harmonized matrix, which expects each column to be a sample and each row a human gene symbol) — https://github.com/xindd/DCNet

## Examples

```
import GEOparse; gse = GEOparse.get_GEO('GSE78220'); expr = gse.pivot_samples('VALUE'); expr.columns = clinical.index; expr = np.log2(expr + 1)
```

## Evaluation signals

- Matrix orientation check: rows are human gene symbols, columns are the 38 melanoma biopsy specimens
- Expression values are log2(x+1) transformed before any downstream risk-score or deconvolution step
- Every sample column with a risk score has a matching clinical record (survival time and responder status); unpaired samples are excluded
- Resulting risk_scores.csv contains one per-sample score computed as sum over 33 prognostic genes of log10(hazard ratio) x transformed expression
- Median-split log-rank test yields p<0.05 and the responder-vs-non-responder Wilcoxon comparison reproduces p≈0.00051 (Figure 5c,d)

## Limitations

- The cohort is small (38 pre-treatment melanoma biopsy specimens in GSE78220), so clinical group sizes after stratification are limited
- No gene-symbol mapping or batch-correction guidance is given; samples whose IDs cannot be matched between expression matrix and clinical file are silently lost
- The log2(x+1) transformation is only 'recommended' by the authors; deviation may break comparability with the published hazard-ratio-based risk scores

## Evidence

- [other] Download the GSE78220 transcriptome matrix and corresponding clinical/response metadata for the 38 pre-treatment melanoma biopsy specimens from GEO using GEOparse.: [evidence span withheld — anchor status: not_found]

---
name: pan-cancer-gene-expression-correlation-analysis
description: Use when bulk RNA-seq expression of a predefined malignant-vs-non-malignant
  gene signature (e.g. TMGs/NMGs) must be correlated with ESTIMATE immune/stromal
  scores and tumor purity and tested for tumor-vs-normal elevation and survival impact
  in TCGA.
when_to_use_negative:
- Input is single-cell transcriptome data needing malignant vs non-malignant cell
  annotation — use the PreCanCell classifier itself rather than this bulk-correlation
  skill
- No paired normal controls exist (normal sample size < 5 per cancer type), so tumor-vs-normal
  t-tests and FDR adjustment cannot be meaningfully evaluated
- The gene set is not predefined and must be discovered by differential expression
  — this skill only tests a fixed TMG/NMG signature downstream
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ESTIMATE
  role: Computes per-sample immune score, stromal score, and tumor purity from bulk
    TCGA expression for the correlation step
- name: R survival
  role: Provides survfit() for Kaplan-Meier modeling and log-rank tests of OS/DSS
    after median-split TMG expression
- name: PreCanCell
  role: Upstream source of the 73 TMGs and 186 NMGs whose pan-cancer bulk-expression
    behavior is being correlated and tested
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:26b4577559dd2de9dea8ae47a14c02571308ec791e18a5591fba64474f42c469
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - ESTIMATE
  - PreCanCell
  - R
  - R survival
  - Spearman correlation
  - Student's t-test
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pan-cancer-gene-expression-correlation-analysis

## Summary

Reproduce pan-cancer TCGA analyses linking a predefined gene set (the 73 tumor marker genes and 186 normal marker genes from PreCanCell) to immune scores, tumor purity, tumor-vs-normal expression, and overall/disease-specific survival, validating the gene set's clinical and molecular relevance.

## When to use

Use when bulk RNA-seq expression of a predefined malignant-vs-non-malignant gene signature (e.g. TMGs/NMGs) must be correlated with ESTIMATE immune/stromal scores and tumor purity and tested for tumor-vs-normal elevation and survival impact in TCGA.

## When NOT to use

- Input is single-cell transcriptome data needing malignant vs non-malignant cell annotation — use the PreCanCell classifier itself rather than this bulk-correlation skill
- No paired normal controls exist (normal sample size < 5 per cancer type), so tumor-vs-normal t-tests and FDR adjustment cannot be meaningfully evaluated
- The gene set is not predefined and must be discovered by differential expression — this skill only tests a fixed TMG/NMG signature downstream

## Inputs

- RSEM-normalized TCGA pan-cancer RNA-seq expression matrix for 33 cancer types (from GDC portal)
- TCGA clinical data with overall survival (OS) and disease-specific survival (DSS) endpoints
- Predefined gene-set membership: 73 tumor marker genes (TMGs) and 186 normal marker genes (NMGs) from the PreCanCell DEG analysis
- TCGA normal (adjacent) sample annotations for tumor-vs-normal comparison

## Outputs

- Per-sample mean TMG and NMG expression values (log2(x+1)-transformed)
- ESTIMATE immune score, stromal score, and tumor purity per TCGA sample
- Spearman correlation table (rho and P-values) of TMG/NMG expression vs immune score, stromal score, and tumor purity
- Tumor-vs-normal Student's t-test P-values with Benjamini-Hochberg FDR adjustment, per cancer type and pan-cancer
- Kaplan-Meier OS and DSS curves with log-rank P-values for low- vs high-TMG-expression classes

## How to apply

Download RSEM-normalized RNA-seq expression and clinical data for the 33 TCGA cancer types from the GDC portal and log2(x+1)-transform expression values. Compute the per-sample mean expression of the 73 TMGs and 186 NMGs, and estimate immune score, stromal score, and tumor purity with the ESTIMATE algorithm. Compute Spearman correlations (rho and P-values) between gene-set expression and immune score, stromal score, and tumor purity, expecting strong positive NMG-immune correlations (rho >= 0.89) and strong negative NMG-purity correlations (rho < -0.70). Compare TMG expression between tumors and normal controls with two-tailed Student's t-tests adjusted by the Benjamini-Hochberg FDR method (expect pan-cancer P = 2.22e-39, and FDR < 0.05 in 25 of the 30 cancer types with at least five normal samples). Finally, split samples at the median TMG expression into low/high classes, fit Kaplan-Meier models with survfit() in the R survival package, and evaluate OS and DSS separation by log-rank test (expect OS P ~ 0 and DSS P = 3.1e-15).

## Related tools

- **ESTIMATE** (Computes per-sample immune score, stromal score, and tumor purity from bulk TCGA expression for the correlation step)
- **R survival** (Provides survfit() for Kaplan-Meier modeling and log-rank tests of OS/DSS after median-split TMG expression)
- **PreCanCell** (Upstream source of the 73 TMGs and 186 NMGs whose pan-cancer bulk-expression behavior is being correlated and tested) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
library(survival); fit <- survfit(Surv(time, status) ~ tmg_level, data = tcga); survdiff(Surv(time, status) ~ tmg_level, data = tcga)
```

## Evaluation signals

- NMG mean expression vs ESTIMATE immune score yields Spearman rho >= 0.89
- NMG mean expression vs ESTIMATE tumor purity yields Spearman rho < -0.70
- Pan-cancer TMG tumor-vs-normal elevation reproduces P = 2.22e-39 with BH-FDR < 0.05 in 25 of 30 cancer types having >= 5 normal samples
- Kaplan-Meier log-rank separation of high- vs low-TMG-expression classes reproduces OS P ~ 0 and DSS P = 3.1e-15
- Expression matrix is log2(x+1)-transformed before any correlation or t-test, and covariate effects (immune score, stromal score, purity) are checked for TMG/NMG consistency with TMG (not NMG) being the tumor-specific signal

## Limitations

- TMG-vs-normal comparisons are only meaningful for cancer types with normal sample size not less than five (25 of 30 tested types reached FDR < 0.05; the remaining TCGA types lack adequate normal controls)
- The TMG/NMG gene sets derive from only five cancer-type single-cell training datasets (RCC, HNSCC, melanoma, LUAD, breast cancer), so the signature may underrepresent cancer biology of other tumor types
- Immune score, stromal score, and tumor purity are computational ESTIMATE inferences from bulk expression, not experimentally measured values, so correlations reflect estimation behavior as well as biology

## Evidence

- [methods] Data acquisition and TCGA download: "we downloaded RNA-seq gene expression profiling (RSEM normalized) and clinical data for 33 TCGA cancer types"
- [methods] Survival analysis implementation: "We implemented survival analyses with the function “survfit()” in the R package “survival.”"
- [results] Expected pan-cancer TMG elevation: "TMGs displayed significantly higher expression levels in TCGA pan-cancer (P = 2.22 × 10−39) and in 25 of the 30 individual cancer types with normal sample’s size not less than five (FDR < 0.05)"
- [results] Gene-set definition: "The DEGs included 73 TMGs and 186 NMGs."

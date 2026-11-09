---
name: ic50-based-compound-efficacy-labeling
description: Use when a drug retrieval evaluation needs binary efficacy ground truth
  per compound — e.g., computing ROC AUC and PR AUC for GSFM reversal scores against
  CMap/XSum/ZhangScore/RS baselines.
when_to_use_negative:
- A compound has no IC50 record in ChEMBL v27 (e.g., very new or unstudied agents),
  so no reliable efficacy label can be assigned and it must be excluded from the labeled
  retrieval benchmark
- The analysis needs graded potency estimates rather than the binary <10 vs >10 μmol/L
  efficacy split, or compares candidates across cell lines where the 10 μmol/L cutoff
  is not pharmacologically justified
edam_topics:
- http://edamontology.org/topic_0202
tools:
- name: GSFM
  role: Framework whose RS_GSFM drug retrieval scores are evaluated against the IC50-derived
    efficacy labels
  repo: https://github.com/LXY1201/GSFM
- name: pROC
  role: Computes ROC AUC distinguishing IC50-labeled effective from ineffective compounds
    by retrieval rank
- name: ROCR
  role: Computes precision-recall AUC for the IC50-labeled compound retrieval rankings
- name: ChEMBL database (version 27)
  role: Source of per-compound IC50 efficacy data used to derive the binary labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:66651a4732d013e615c3093fb5b80866bd0cbf395e4794438967e3981b116c23
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LXY1201/GSFM
  repo_url: https://github.com/LXY1201/GSFM
  tools:
  - ChEMBL database (version 27)
  - Docker
  - GSFM
  - R
  - ROCR
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ic50-based-compound-efficacy-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Labels compounds as effective or ineffective using median IC50 values from the ChEMBL database (version 27) with a 10 μmol/L cutoff, providing the ground-truth classes needed to benchmark drug retrieval methods (CMap, XSum, ZhangScore, RS) against the RS_GSFM score via ROC and PR AUC.

## When to use

Use when a drug retrieval evaluation needs binary efficacy ground truth per compound — e.g., computing ROC AUC and PR AUC for GSFM reversal scores against CMap/XSum/ZhangScore/RS baselines.

## When NOT to use

- A compound has no IC50 record in ChEMBL v27 (e.g., very new or unstudied agents), so no reliable efficacy label can be assigned and it must be excluded from the labeled retrieval benchmark
- The analysis needs graded potency estimates rather than the binary <10 vs >10 μmol/L efficacy split, or compares candidates across cell lines where the 10 μmol/L cutoff is not pharmacologically justified

## Inputs

- ChEMBL database (version 27) IC50 efficacy records per compound
- Per-compound median IC50 table derived from ChEMBL records
- Compound identifiers matched to the drug-induced LINCS gene expression / GSFM active matrix

## Outputs

- Binary efficacy label vector per compound (effective: IC50 < 10 μmol/L; ineffective: IC50 > 10 μmol/L)
- Per-fold ROC AUC and PR AUC values (pROC/ROCR) averaged across 10 folds
- Benchmark table (benchmark_table.csv) comparing GSFM with CMap, XSum, ZhangScore, RS, Gene, and Viper per cancer setting

## How to apply

Source efficacy data for each compound as IC50 from the ChEMBL database (version 27) and collapse replicate measurements to a per-compound median IC50. Apply the threshold effective if median IC50 < 10 μmol/L and ineffective if median IC50 > 10 μmol/L, yielding a binary label vector aligned with the compound identifiers in the drug-induced GSFM active matrix. Feed these labels into the 10-fold cross-validation retrieval framework, where each fold holds out one compound subset so every compound is validated exactly once. In each fold, compute ROC AUC and precision-recall AUC with pROC 1.17.0.1 and ROCR 1.0.11 by distinguishing effective from ineffective compounds by retrieval rank, then average per-fold AUCs across folds for each method and cancer setting (BRCA/MCF7, LUAD/A549, CRPC/PC3). The rationale is that GSFM reversal scores correlate positively with IC50, so the 10 μmol/L split lets higher ROC values indicate superior retrieval of efficacious compounds.

## Related tools

- **GSFM** (Framework whose RS_GSFM drug retrieval scores are evaluated against the IC50-derived efficacy labels) — https://github.com/LXY1201/GSFM
- **pROC** (Computes ROC AUC distinguishing IC50-labeled effective from ineffective compounds by retrieval rank)
- **ROCR** (Computes precision-recall AUC for the IC50-labeled compound retrieval rankings)
- **ChEMBL database (version 27)** (Source of per-compound IC50 efficacy data used to derive the binary labels)

## Examples

```
Rscript -e 'lbl <- ifelse(chembl_ic50$median_ic50_umol < 10, "effective", "ineffective"); roc <- pROC::roc(lbl, retrieval_rank)'
```

## Evaluation signals

- Every compound in the drug-induced GSFM active matrix carries exactly one label, and labels are consistent with the median ChEMBL IC50 relative to the 10 μmol/L threshold
- The 10-fold split ensures each compound is validated exactly once across folds (fold union equals the full compound set)
- Per-fold ROC and PR AUCs from pROC/ROCR are finite and the cross-fold average is reported per method per cancer setting
- benchmark_table.csv contains ROC and PR AUC values for every method (GSFM, CMap, XSum, ZhangScore, RS, Gene, Viper) and every setting (BRCA, LUAD, CRPC)
- The GSFM-based score attains higher ROC AUC than baselines: ROC = 0.840/PR = 0.832 (BRCA), ROC = 0.825/PR = 0.894 (LUAD), ROC = 0.910/PR = 0.951 (CRPC)

## Limitations

- Labeling depends entirely on ChEMBL v27 IC50 coverage; compounds without IC50 data cannot be classified as effective or ineffective
- The binary 10 μmol/L cutoff collapses potency information into two classes, and the broader GSFM workflow may not be applicable when appropriate cell-line/disease context is missing, e.g., 'our method may not be applicable to certain rare diseases when normal samples are unavailable'

## Evidence

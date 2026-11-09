---
name: knockdown-perturbation-differential-testing
description: 'Use when candidate RBPs (e.g., from CRIT NMF clusters) need experimental
  validation via public shRNA/siRNA knockdown RNA-seq data: detect circRNAs per dataset,
  compute per-sample mCRI, and run group t tests knockdown vs control.'
when_to_use_negative:
- The input is not perturbation RNA-seq but observational expression data (e.g., TCGA/MiOncoCirc
  correlation or NMF feature matrices) — use the CRIT feature-collecting scripts instead
- Only one circRNA-detecting tool can be run, since the validation requires concordance
  between CIRI2 and find_circ calls
- No knockdown and matched control samples exist for the RBP of interest (only 22
  of the CRIT candidates had suitable knockdown RNA-seq data)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: CIRI2
  role: circRNA detection algorithm applied to raw alignments of each knockdown dataset,
    first of the two complementary tools whose calls must concur
- name: find_circ
  role: second, complementary circRNA-detecting algorithm run on the same raw alignments;
    concordance with CIRI2 is required to call an RBP validated
- name: R
  role: environment used for computing per-sample mCRI, running group t tests between
    knockdown and control conditions, and writing mcri_validation_results.csv
- name: survival
  role: R package from the CRIT pipeline; in this skill the R environment hosts the
    statistical group t tests of mCRI between conditions
- name: CRIT
  role: upstream NMF-based pipeline that supplies the candidate RBPs (22 putative
    regulators) that this knockdown validation skill tests
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:98d87f72563e73f2f0e8a5686a06eb342ed46de1610ca7322efe56da08b60b82
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CIRI2
  - CRIT
  - R
  - find_circ
  - survival
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# knockdown-perturbation-differential-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation skill that tests whether candidate circRNA-regulating RBPs affect global circRNA expression upon shRNA/siRNA knockdown, by reanalyzing public knockdown RNA-seq datasets with two circRNA-detecting tools (CIRI2 and find_circ) and computing the normalized modified circRNA index (mCRI) per knockdown/control pair.

## When to use

Use when candidate RBPs (e.g., from CRIT NMF clusters) need experimental validation via public shRNA/siRNA knockdown RNA-seq data: detect circRNAs per dataset, compute per-sample mCRI, and run group t tests knockdown vs control.

## When NOT to use

- The input is not perturbation RNA-seq but observational expression data (e.g., TCGA/MiOncoCirc correlation or NMF feature matrices) — use the CRIT feature-collecting scripts instead
- Only one circRNA-detecting tool can be run, since the validation requires concordance between CIRI2 and find_circ calls
- No knockdown and matched control samples exist for the RBP of interest (only 22 of the CRIT candidates had suitable knockdown RNA-seq data)

## Inputs

- Raw FASTQ files of shRNA/siRNA knockdown RNA-seq datasets (e.g., 27 datasets from GEO/ENCODE for 22 candidate RBPs, per Table S5)
- List of candidate RBP symbols (e.g., the 22 CRIT candidate RBPs)
- Positive control gene (QKI) and negative control gene (SIGLEC15) identifiers

## Outputs

- Per-sample mCRI table (normalized modified circRNA index per knockdown/control pair)
- Significance summary with group t test results per RBP (e.g., mcri_validation_results.csv)
- Dual-tool concordance list of RBPs supported by both CIRI2 and find_circ

## How to apply

First obtain the raw FASTQ files of the shRNA/siRNA knockdown RNA-seq datasets for the candidate RBPs from GEO/ENCODE (in the CRIT study, 27 datasets covering 22 candidate RBPs, listed in Table S5). Detect circRNAs in each dataset independently with two complementary algorithms, CIRI2 and find_circ, from the raw alignments, and require concordance between the two tools' calls. Compute the normalized modified circRNA index (mCRI) per sample, quantifying global circRNA abundance relative to linear RNA expression for each knockdown/control pair. Run group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP, including a positive control (QKI) and a negative control (SIGLEC15) to calibrate significance. Write the per-sample mCRI table and significance summary to a results file (e.g., mcri_validation_results.csv) and report which RBPs are supported by both circRNA-detecting tools.

## Related tools

- **CIRI2** (circRNA detection algorithm applied to raw alignments of each knockdown dataset, first of the two complementary tools whose calls must concur)
- **find_circ** (second, complementary circRNA-detecting algorithm run on the same raw alignments; concordance with CIRI2 is required to call an RBP validated)
- **R** (environment used for computing per-sample mCRI, running group t tests between knockdown and control conditions, and writing mcri_validation_results.csv)
- **survival** (R package from the CRIT pipeline; in this skill the R environment hosts the statistical group t tests of mCRI between conditions)
- **CRIT** (upstream NMF-based pipeline that supplies the candidate RBPs (22 putative regulators) that this knockdown validation skill tests) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
Rscript RBP_analysis.R
```

## Evaluation signals

- 13 of the 22 candidate RBPs show a statistically significant mCRI change upon knockdown (group t test)
- QKI, the positive control, is significant after knockdown
- SIGLEC15, the negative control, is non-significant after knockdown
- The dual-tool concordance list (RBPs supported by both CIRI2 and find_circ) matches exactly SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1
- mcri_validation_results.csv contains both the per-sample mCRI table and the per-RBP significance summary

## Limitations

- Only 22 of the CRIT candidate RBPs had suitable knockdown RNA-seq data available; RBPs without public shRNA/siRNA datasets cannot be validated this way
- Significance calls depend on concordance between two circRNA-detecting tools; RBPs supported by only one tool are not counted as validated (only 13 of 22 showed evidence from at least one tool, 5 by both)
- Validation is computational (global mCRI change), not a direct biochemical demonstration of individual circRNA regulation

## Evidence

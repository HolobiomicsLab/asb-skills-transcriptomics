---
name: geo-encode-dataset-retrieval-and-reprocessing
description: Use when candidate circRNA regulators (e.g. CRIT-predicted RBPs) need
  experimental validation via reanalysis of public knockdown RNA-seq FASTQ files from
  GEO/ENCODE, with global circRNA abundance compared between knockdown and control
  conditions.
when_to_use_negative:
- The goal is de novo discovery of circRNA regulators from MiOncoCirc/TCGA features
  (rho_m, Zp, Zs, Zg, C value) via the CRIT NMF pipeline — that is a separate candidate-prediction
  skill, not knockdown reanalysis.
- Only circRNA-RBP interaction evidence is needed (e.g. CLIP database overlap or catRAPID/RNAct
  binding scores) rather than expression-level validation from knockdown perturbation
  data.
- No raw FASTQ data (or suitable knockdown series) exist for the RBP of interest —
  in the source study, only 22 of the putative regulators had suitable knockdown RNA-seq
  data.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIRI2
  role: Detects circRNAs independently from raw alignments of each knockdown RNA-seq
    dataset; its calls must concord with find_circ before an RBP is declared validated
- name: find_circ
  role: Second, complementary circRNA-detecting algorithm run on the same datasets
    to require dual-tool concordance
- name: R
  role: Runs group t tests comparing per-sample mCRI between knockdown and control
    conditions for each RBP and writes mcri_validation_results.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:d49f9f3950d4cf50fb2435494ba3f171fd68a47bf3497f1e70e5cd383e0ff564
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CIRI2
  - CRIT
  - R
  - find_circ
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# geo-encode-dataset-retrieval-and-reprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves publicly available shRNA/siRNA knockdown RNA-seq datasets from GEO/ENCODE and reprocesses them from raw FASTQ to detect circRNAs (CIRI2, find_circ) and compute the normalized modified circRNA index (mCRI), enabling experimental validation of CRIT-predicted circRNA regulators such as SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1.

## When to use

Use when candidate circRNA regulators (e.g. CRIT-predicted RBPs) need experimental validation via reanalysis of public knockdown RNA-seq FASTQ files from GEO/ENCODE, with global circRNA abundance compared between knockdown and control conditions.

## When NOT to use

- The goal is de novo discovery of circRNA regulators from MiOncoCirc/TCGA features (rho_m, Zp, Zs, Zg, C value) via the CRIT NMF pipeline — that is a separate candidate-prediction skill, not knockdown reanalysis.
- Only circRNA-RBP interaction evidence is needed (e.g. CLIP database overlap or catRAPID/RNAct binding scores) rather than expression-level validation from knockdown perturbation data.
- No raw FASTQ data (or suitable knockdown series) exist for the RBP of interest — in the source study, only 22 of the putative regulators had suitable knockdown RNA-seq data.

## Inputs

- Public shRNA/siRNA knockdown RNA-seq raw FASTQ files from GEO/ENCODE (27 datasets listed in Table S5, covering 22 candidate RBPs)
- Human reference genome and gene annotation for circRNA detection alignments
- Knockdown vs control sample metadata (which samples are KD vs control) per dataset

## Outputs

- Per-sample mCRI table (normalized modified circRNA index per knockdown/control pair)
- mcri_validation_results.csv containing the per-sample mCRI table and group t-test significance summary
- Dual-tool concordance list of RBPs supported by both CIRI2 and find_circ

## How to apply

Download raw FASTQ files of the shRNA/siRNA knockdown RNA-seq datasets listed in Table S5 from GEO/ENCODE for each candidate RBP. Detect circRNAs in each dataset independently with two complementary algorithms, CIRI2 and find_circ, from the raw alignments to the reference genome. Compute the normalized modified circRNA index (mCRI) per sample, quantifying global circRNA abundance relative to linear RNA expression for each knockdown/control pair. Run group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP, including QKI as a positive control and SIGLEC15 as a negative control. Require concordance between CIRI2 and find_circ calls before declaring an RBP supported, and write the per-sample mCRI table plus significance summary to mcri_validation_results.csv.

## Related tools

- **CIRI2** (Detects circRNAs independently from raw alignments of each knockdown RNA-seq dataset; its calls must concord with find_circ before an RBP is declared validated)
- **find_circ** (Second, complementary circRNA-detecting algorithm run on the same datasets to require dual-tool concordance)
- **R** (Runs group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP and writes mcri_validation_results.csv)

## Evaluation signals

- 13 of the 22 RBPs show a statistically significant mCRI change upon knockdown (group t test).
- QKI is significant in the mCRI test (positive control).
- SIGLEC15 is non-significant (negative control).
- The dual-tool concordance list matches SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1 exactly.
- mcri_validation_results.csv exists and contains both the per-sample mCRI table and the significance summary.

## Limitations

- Only RBPs with suitable public shRNA/siRNA knockdown RNA-seq data can be validated; in the source study, 22 of the putative regulators had suitable data and 13 showed experimental evidence from at least one circRNA-detecting tool.
- Validation requires concordance between CIRI2 and find_circ; support from only one tool (8 RBPs) was considered insufficient.
- Results depend on the quality and depth of third-party GEO/ENCODE datasets and on correct assignment of knockdown vs control samples in heterogeneous public studies.

## Evidence

- [other] Among the 22 putative regulators with suitable knockdown RNA-seq data, 13 RBPs showed experimental evidence from at least one circRNA-detecting tool: [evidence span withheld — anchor status: not_found]
- [other] Compute the normalized modified circRNA index (mCRI) per sample, quantifying global circRNA abundance relative to linear RNA expression for each knockdown/control pair: [evidence span withheld — anchor status: not_found]

---
name: circrna-detection-from-rnaseq-alignments
description: Use when raw FASTQ/alignment data from knockdown (shRNA/siRNA) or condition-controlled
  RNA-seq experiments must be mined for back-splice circRNAs to test whether knocking
  down a candidate RBP changes global circRNA expression.
when_to_use_negative:
- Input is already a circRNA count/feature table and no raw alignments are available,
  so CIRI2/find_circ cannot be re-run from the alignments
- The goal is RBP-circRNA binding-site prediction or NMF-based regulator discovery
  (CRIT's features-collecting and cluster-identify stages) rather than validating
  global circRNA expression change upon knockdown
- No suitable public knockdown RNA-seq data exist for the RBP of interest — in the
  source study only 13 of 22 candidate RBPs had suitable knockdown data
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIRI2
  role: First complementary algorithm used to detect circRNAs independently from the
    raw alignments of each knockdown dataset
- name: find_circ
  role: Second complementary algorithm used to detect circRNAs independently from
    the raw alignments; concordance with CIRI2 is required for validation
- name: R
  role: Used to compute per-sample mCRI values, run group t tests between knockdown
    and control conditions, and write mcri_validation_results.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:ce34aaec4300a610d6c9ad7bb1ed3922651ed2a3ba1e872b9f2139ab7446b9e0
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
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# circRNA-detection-from-rnaseq-alignments

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect circRNAs directly from raw shRNA/siRNA knockdown RNA-seq alignments using two complementary algorithms (CIRI2 and find_circ), then quantify global circRNA burden per sample via the normalized modified circRNA index (mCRI) to test whether an RBP regulates circRNA expression.

## When to use

Use when raw FASTQ/alignment data from knockdown (shRNA/siRNA) or condition-controlled RNA-seq experiments must be mined for back-splice circRNAs to test whether knocking down a candidate RBP changes global circRNA expression.

## When NOT to use

- Input is already a circRNA count/feature table and no raw alignments are available, so CIRI2/find_circ cannot be re-run from the alignments
- The goal is RBP-circRNA binding-site prediction or NMF-based regulator discovery (CRIT's features-collecting and cluster-identify stages) rather than validating global circRNA expression change upon knockdown
- No suitable public knockdown RNA-seq data exist for the RBP of interest — in the source study only 13 of 22 candidate RBPs had suitable knockdown data

## Inputs

- Raw FASTQ files of shRNA/siRNA knockdown RNA-seq datasets from GEO/ENCODE (e.g., Table S5 datasets)
- Sample metadata pairing each knockdown sample with its matched control
- Reference genome alignment/annotation needed by CIRI2 and find_circ (e.g., GTF, CircBase-style circRNA annotation)

## Outputs

- Per-dataset circRNA call sets from CIRI2 and from find_circ
- Per-sample normalized modified circRNA index (mCRI) table for each knockdown/control pair
- Group t-test significance summary per RBP (p-values, positive/negative control outcomes)
- Dual-tool concordance list of RBPs supported by both CIRI2 and find_circ
- mcri_validation_results.csv (per-sample mCRI table plus significance summary)

## How to apply

Download the raw FASTQ files of the relevant shRNA/siRNA knockdown RNA-seq datasets (e.g., the 27 GEO/ENCODE datasets in Table S5 covering 22 candidate RBPs) and detect circRNAs in each dataset independently with two complementary algorithms, CIRI2 and find_circ, from the raw alignments. For each knockdown/control pair, compute the normalized modified circRNA index (mCRI) per sample, quantifying global circRNA abundance relative to linear RNA expression. Run group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP, including a known regulator (QKI) as positive control and a non-regulator (SIGLEC15) as negative control. Require concordance between CIRI2 and find_circ calls so that only RBPs supported by both circRNA-detecting tools are accepted. Write the per-sample mCRI table and significance summary to mcri_validation_results.csv.

## Related tools

- **CIRI2** (First complementary algorithm used to detect circRNAs independently from the raw alignments of each knockdown dataset)
- **find_circ** (Second complementary algorithm used to detect circRNAs independently from the raw alignments; concordance with CIRI2 is required for validation)
- **R** (Used to compute per-sample mCRI values, run group t tests between knockdown and control conditions, and write mcri_validation_results.csv)

## Evaluation signals

- 13 of the 22 RBPs show a statistically significant mCRI change upon knockdown (group t test)
- QKI is significant (positive control)
- SIGLEC15 is non-significant (negative control)
- The dual-tool concordance list matches SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1 (supported by both CIRI2 and find_circ)
- mcri_validation_results.csv contains a per-sample mCRI row for every knockdown/control pair in the analyzed datasets

## Limitations

- Only RBPs with suitable public shRNA/siRNA knockdown RNA-seq data can be assessed; in the source study 9 of the 22 candidate RBPs lacked suitable data
- CircRNA detection sensitivity depends on the algorithm; the design therefore requires concordance between CIRI2 and find_circ, and single-tool-only calls are treated as weaker evidence
- mCRI measures global circRNA abundance change, so it cannot resolve which specific circRNAs an RBP regulates

## Evidence

- [other] knockdown workflow (source card): [evidence span withheld — anchor status: not_found]
- [other] mCRI definition (source card): [evidence span withheld — anchor status: not_found]
- [abstract] article abstract: "When analyzing RNA sequencing (RNA-seq) datasets after short hairpin RNA (shRNA)/small interfering RNA (siRNA) knockdown, we found several novel RBPs that can affect global circRNA expression"
- [readme] README circRNA release file: "v0.1.release.txt : All circRNAs (filtered at 2 reads) in 2,000+ cancer samples in hg38"

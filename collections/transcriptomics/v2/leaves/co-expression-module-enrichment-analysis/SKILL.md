---
name: co-expression-module-enrichment-analysis
description: Use when validating a set of predicted RBP candidates by asking whether
  they are co-expressed with known (gold-standard) circRNA regulators, using the Harmonizome
  MSigDB Cancer Gene Co-expression Modules resource and permutation-based enrichment
  testing.
when_to_use_negative:
- The validation question concerns physical protein-protein interactions or shared
  transcription factor (TRANSFAC) modulation instead of co-expression — these are
  separate Harmonizome-based analyses with their own backgrounds and expected results.
- The candidate list did not come from CRIT's NMF clustering (e.g., an arbitrary RBP
  set), so the candidate–background design and the 57/73 or 5-candidate expectations
  no longer apply.
- The study system is not human cancer samples with Harmonizome coverage — the MSigDB
  Cancer Gene Co-expression Modules are cancer-focused and human-specific.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CRIT
  role: Upstream pipeline whose NMF clustering produces the candidate regulator list
    that is then tested for co-expression with gold regulators via Harmonizome evidence
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Implementation language for loading inputs (readr, data.table), performing
    the 10,000 random-sampling permutation tests, and writing the association result
    table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:60e56acea1c42268b06f1fef97486356569961fafdf4517739a4fe7f5168e3a4
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
  - CRIT
  - R
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# co-expression-module-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether CRIT-predicted candidate circRNA regulators are co-expressed with gold-standard circRNA regulators using the Harmonizome MSigDB Cancer Gene Co-expression Modules dataset, with significance assessed by 10,000 random-sampling permutations against both gene and RBP backgrounds.

## When to use

Use when validating a set of predicted RBP candidates by asking whether they are co-expressed with known (gold-standard) circRNA regulators, using the Harmonizome MSigDB Cancer Gene Co-expression Modules resource and permutation-based enrichment testing.

## When NOT to use

- The validation question concerns physical protein-protein interactions or shared transcription factor (TRANSFAC) modulation instead of co-expression — these are separate Harmonizome-based analyses with their own backgrounds and expected results.
- The candidate list did not come from CRIT's NMF clustering (e.g., an arbitrary RBP set), so the candidate–background design and the 57/73 or 5-candidate expectations no longer apply.
- The study system is not human cancer samples with Harmonizome coverage — the MSigDB Cancer Gene Co-expression Modules are cancer-focused and human-specific.

## Inputs

- Candidate regulator list from CRIT NMF clustering (e.g., NMFgroup_XXclusterYYrun.txt, chained output of the CRIT pipeline)
- gold_regulators.xlsx (gold-standard circRNA regulator list, placed in ./DATA/input/)
- Harmonizome MSigDB Cancer Gene Co-expression Modules dataset (downloaded to ./DATA/input/Harmonizome)

## Outputs

- Co-expression association result table reporting candidate counts, observed overlaps, and permutation-based p-values for candidate–gold regulator co-expression

## How to apply

Load the CRIT NMF-derived candidate regulator list (chained from the NMF.R clustering step) and the gold_regulators.xlsx file into R using readr/data.table. Download the Harmonizome MSigDB Cancer Gene Co-expression Modules dataset (https://maayanlab.cloud/Harmonizome/dataset/MSigDB+Cancer+Gene+Co-expression+Modules) into ./DATA/input/Harmonizome. For each candidate, query the co-expression modules for shared module membership with gold regulators, and assess enrichment by comparing the observed overlap against 10,000 random-sampling permutations drawn from two distinct backgrounds: the whole-gene background and the RBP-only background, because candidates are RBPs and the appropriate null differs between these universes. Select candidates significant at p < 0.0001 (the paper's expectation is 5 candidates: RBBP6, RPL17, RPS12, SRSF5, THRAP3). Write the co-expression association result table, reporting candidate counts, observed overlaps, and permutation-based p-values, to the output directory alongside the physical-interaction and shared-TF association tables.

## Related tools

- **CRIT** (Upstream pipeline whose NMF clustering produces the candidate regulator list that is then tested for co-expression with gold regulators via Harmonizome evidence) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Implementation language for loading inputs (readr, data.table), performing the 10,000 random-sampling permutation tests, and writing the association result table)

## Examples

```
Rscript RBP_analysis.R
```

## Evaluation signals

- Exactly 5 candidates should pass p < 0.0001 against the co-expression background: RBBP6, RPL17, RPS12, SRSF5, THRAP3.
- Permutation testing must use 10,000 random-sampling iterations, run against both the gene background and the RBP background, not a single null.
- Input paths must match the CRIT README layout: gold_regulators.xlsx in ./DATA/input/ and Harmonizome files in ./DATA/input/Harmonizome.
- Output table must contain, for each candidate, the candidate counts, observed overlap with gold regulators, and permutation-based p-values.
- The co-expression result should be reported as one of three Harmonizome-based association tables (with physical-interaction and shared-TF tables), not merged with them.

## Limitations

- Significance depends on the choice of background (gene vs. RBP); the same overlap can be significant under one background and not the other, as seen in the shared-TF analysis where the gene background gave p = 0.0001 but the RBP background gave p = 0.265.
- Co-expression evidence is correlative and cancer-module-based; it supports, but does not prove, a candidate's role in circRNA regulation.
- Permutation p-values are bounded by the number of random samples (10,000), so the minimum attainable p-value is on the order of 1e-4.

## Evidence

- [other] Co-expression analysis with gold regulators via Harmonizome modules: [evidence span withheld — anchor status: not_found]

---
name: transcription-factor-target-enrichment-analysis
description: Use when you have a list of predicted circRNA/RBP candidate regulators
  and a gold-standard regulator list, and need to test shared transcription factor
  modulation (or related Harmonizome associations) to validate the candidates.
when_to_use_negative:
- Input is a raw feature matrix (Zp, Zs, Zg, rho_m, C value) that has not yet been
  clustered by NMF.R — candidate regulators do not exist yet
- You need de novo TF motif discovery or TF binding site prediction from sequence;
  this skill only tests overlap against the curated TRANSFAC target sets
- The candidate set is not circRNA/RBP-related — the RBP-background permutation (p
  = 0.265) shows enrichment is context-dependent for circRNA regulators
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0099
tools:
- name: CRIT
  role: Upstream pipeline whose NMF clustering produces the candidate regulators being
    validated against Harmonizome TF-target evidence
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Implementation language for reading inputs (readr/data.table), running the
    10,000 permutation tests, and writing result tables
- name: Harmonizome
  role: Source of the TRANSFAC Curated Transcription Factor Targets (and companion
    PPI / co-expression) datasets used as evidence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:504d7d584c3aa872ba5b7ea94a9e1e8df1d1f31ee3d962be0942df005aaaf06a
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
  - Harmonizome
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# transcription-factor-target-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Harmonizome-based validation step from the CRIT pipeline that tests whether NMF-predicted circRNA regulator candidates are modulated by the same transcription factors as gold-standard regulators, using the TRANSFAC Curated Transcription Factor Targets dataset with 10,000 random-sampling permutation controls.

## When to use

Use when you have a list of predicted circRNA/RBP candidate regulators and a gold-standard regulator list, and need to test shared transcription factor modulation (or related Harmonizome associations) to validate the candidates.

## When NOT to use

- Input is a raw feature matrix (Zp, Zs, Zg, rho_m, C value) that has not yet been clustered by NMF.R — candidate regulators do not exist yet
- You need de novo TF motif discovery or TF binding site prediction from sequence; this skill only tests overlap against the curated TRANSFAC target sets
- The candidate set is not circRNA/RBP-related — the RBP-background permutation (p = 0.265) shows enrichment is context-dependent for circRNA regulators

## Inputs

- Candidate regulator list from CRIT NMF clustering (chained output, e.g. NMFgroup_XXclusterYYrun.txt)
- gold_regulators.xlsx (gold-standard circRNA regulators collected from previous research)
- Harmonizome TRANSFAC Curated Transcription Factor Targets dataset files (./DATA/input/Harmonizome)

## Outputs

- Shared-TF association result table reporting candidate counts, observed overlaps, and permutation-based p-values/FDR (14 significant candidates expected)
- Optionally, companion PPI and co-expression association tables (57/73 candidates at FDR < 0.05; 5 co-expression candidates at p < 0.0001)

## How to apply

Load the candidate regulator list from the CRIT NMF clustering output and gold_regulators.xlsx into R using readr/data.table. Query the Harmonizome TRANSFAC Curated Transcription Factor Targets dataset (./DATA/input/Harmonizome) to identify candidates regulated by the same TFs as gold regulators. Assess significance by comparing the observed TF-sharing overlap against 10,000 random-sampling permutations drawn from two backgrounds: the gene background (expected p = 0.0001) and the RBP-only background (expected p = 0.265, showing the effect is not simply an RBP property). Expect 14 candidates to pass this shared-TF test in the CRIT cancer analysis; write a result table reporting candidate counts, observed overlaps, and permutation-based p-values/FDR to the output directory. The same permutation framework extends to the companion Harmonizome checks (Pathway Commons PPI with FDR < 0.05 yielding 57/73 candidates; MSigDB Cancer Gene Co-expression Modules at p < 0.0001 yielding RBBP6, RPL17, RPS12, SRSF5, THRAP3).

## Related tools

- **CRIT** (Upstream pipeline whose NMF clustering produces the candidate regulators being validated against Harmonizome TF-target evidence) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Implementation language for reading inputs (readr/data.table), running the 10,000 permutation tests, and writing result tables)
- **Harmonizome** (Source of the TRANSFAC Curated Transcription Factor Targets (and companion PPI / co-expression) datasets used as evidence)

## Examples

```
Rscript RBP_analysis.R
```

## Evaluation signals

- Shared-TF enrichment against the gene background reaches p = 0.0001 with 10,000 permutations; against the RBP background it is non-significant (p = 0.265)
- 14 of the 73 newly predicted candidates pass the shared-TF test
- Companion check: 57/73 candidates show enriched physical interactions with gold regulators at FDR < 0.05
- Companion check: co-expression analysis (MSigDB Cancer Gene Co-expression Modules) recovers exactly RBBP6, RPL17, RPS12, SRSF5, THRAP3 at p < 0.0001
- Three result tables are written to the output directory, each reporting candidate counts, observed overlaps, and permutation-based p-values/FDR

## Limitations

- This is associative evidence from curated databases (TRANSFAC/Harmonizome), not experimental validation of TF-mediated regulation
- The enrichment is significant against the gene background but not the RBP background (p = 0.265), so it does not distinguish these candidates from RBPs generally
- Path parameters must be specified in CRIT_settings.R before sourcing; incorrect ./DATA/input/Harmonizome file placement will break the analysis

## Evidence

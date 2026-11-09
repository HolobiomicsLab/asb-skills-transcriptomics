---
name: drug-combination-coverage-computation
description: Use when computing percent cell-line coverage of proteasome-inhibitor
  backbone + secondary drug (secDrug) combinations over GDSC1000-derived resistant
  B-cell cancer cell lines to rank combination candidates.
when_to_use_negative:
- The task is pre-clinical/functional validation of a combination in xenograft, mouse,
  or ex vivo models rather than in silico coverage prediction.
- The pipeline must test more than two secDrugs in combination — the article states
  model systems for testing more than two secDrugs are not yet developed.
- The input lacks drug IC50 response data per cell line, so S/R/N grouping via AssignGroup
  cannot be computed.
edam_topics:
- http://edamontology.org/topic_0208
tools:
- name: R
  role: Implementation language of the universally applicable R package for predicting
    secondary therapies in chemotherapy-resistant cancers
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Optimization-regularization greedy set-cover algorithm that selects secDrugs
    added to the PI backbone before coverage computation
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: R package exporting AssignGroup(x, l) for S/R/N response grouping and CombinationDrugs(dir,
    file, dest) for top-28 regimen generation and coverage computation
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:42c026d1629cd19f5d6ab318d90b1d89f1a8964cc3877b536305ddaa6389ace1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# drug-combination-coverage-computation

## Summary

A set-cover-based regimen computation from the secDrug pipeline that categorizes proteasome-inhibitor (PI) response groups (S/R/N) with AssignGroup and uses CombinationDrugs to generate top PI-backbone + secDrug regimens with percent cell-line coverage in GDSC1000-derived B-cell cancer panels.

## When to use

Use when computing percent cell-line coverage of proteasome-inhibitor backbone + secondary drug (secDrug) combinations over GDSC1000-derived resistant B-cell cancer cell lines to rank combination candidates.

## When NOT to use

- The task is pre-clinical/functional validation of a combination in xenograft, mouse, or ex vivo models rather than in silico coverage prediction.
- The pipeline must test more than two secDrugs in combination — the article states model systems for testing more than two secDrugs are not yet developed.
- The input lacks drug IC50 response data per cell line, so S/R/N grouping via AssignGroup cannot be computed.

## Inputs

- GDSC1000 pharmacogenomics dataset (drug IC50 values) downloaded from cancerrxgene.org
- Filtered list of computable B-cell cancer cell lines (e.g., the 94-line B-cell panel)
- Categorized PI response groups (S/R/N labels per cell line) produced by AssignGroup(x, l)

## Outputs

- Ranked top-28 PI-backbone regimen table with percent cell-line coverage for 2 secDrugs and 3 secDrugs (secdrug_regimen_coverage.csv)
- Selected secDrugs per regimen (e.g., 17-AAG/HSP90i, FK866/NAMPTi, YM201636/PIKfyvei, PLX-4720/Rafi)

## How to apply

Load the GDSC1000 pharmacogenomics dataset (cancerrxgene.org) in R and restrict to the computable B-cell cancer cell line panel used for coverage. Run AssignGroup(x, l) from the CombinationDrugMyeloma package to label each cell line's PI IC50 as sensitive (S), resistant (R), or null (N); the resistant group defines the coverage target. Feed the categorized groups into the secDrug optimization-regularization greedy set-cover algorithm to select secDrugs added to the PI backbone, then call CombinationDrugs(dir, file, dest) to generate the top-28 PI-backbone regimens and compute percent coverage with 2 and with 3 secDrugs. Write the regimen table with percent-coverage values to secdrug_regimen_coverage.csv and validate against published values (PI only 33.0%; PI+FK866+17-AAG 72.2% with 2 secDrugs and 82.5% with 3 secDrugs).

## Related tools

- **R** (Implementation language of the universally applicable R package for predicting secondary therapies in chemotherapy-resistant cancers) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Optimization-regularization greedy set-cover algorithm that selects secDrugs added to the PI backbone before coverage computation) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (R package exporting AssignGroup(x, l) for S/R/N response grouping and CombinationDrugs(dir, file, dest) for top-28 regimen generation and coverage computation) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
CombinationDrugs(dir = "gdsc1000", file = "bcell_pi_groups.csv", dest = "secdrug_regimen_coverage.csv")
```

## Evaluation signals

- secdrug_regimen_coverage.csv reproduces published coverage: PI only 33.0%; top regimen PI+FK866+17-AAG at 72.2% with 2 secDrugs and 82.5% with 3 secDrugs.
- Every cell line in the coverage denominator carries exactly one AssignGroup label (S, R, or N) with no null-labeled line counted toward regimen coverage.
- Output contains the top-28 regimens, each with coverage values for both the 2-secDrug and 3-secDrug conditions.
- Top-ranked combinations include the known secDrugs: 17-AAG (HSP90 inhibitor), FK866 (NAMPT inhibitor), YM201636 (PIKfyve inhibitor), and PLX-4720 (Raf inhibitor).
- All percent-coverage values fall within [0, 100] and monotonically increase (or stay equal) when a third secDrug is added to a 2-secDrug regimen.

## Limitations

- The pipeline is currently limited to two-drug combinations; model systems for testing more than two secDrugs are not yet developed.
- Coverage depends on the computable B-cell cell line panel; predicted secDrugs (e.g., KIN001-102, SB505124) still await pre-clinical/clinical validation in xenograft or mouse models.
- Testing in a wider panel of HMCLs representing refractory and clonally derived acquired resistant lines is not yet done.

## Evidence

- [abstract] uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma: "uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma"
- [intro] we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance: "we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance"
- [discussion] we performed extensively in vitro, ex vivo, and functional validation in research models of refractory and resistant myeloma: "we performed extensively in vitro, ex vivo, and functional validation in research models of refractory and resistant myeloma"

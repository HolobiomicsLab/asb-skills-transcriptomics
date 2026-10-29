---
name: cell-line-panel-filtering-b-cell-cancer
description: Use when a GDSC1000-style pharmacogenomics screen must be subset to a
  defined cancer-type cell-line panel (e.g., B-cell cancers) before grouping drug
  responses (S/R/N) and computing percent coverage of drug-combination regimens.
when_to_use_negative:
- The goal is single-cell screening of secDrugs by subclonal drug-target enrichment
  (scRNAseq) or CyTOF proteomics — this skill handles pharmacogenomics panel subsetting,
  not single-cell multi-omics.
- Cell-line responses have already been categorized into S/R/N groups and regimen
  coverage has been computed — re-filtering would double-count or distort the coverage
  denominator.
- The target indication is not represented in the GDSC1000 B-cell cancer panel or
  a different backbone drug class (e.g., IMiD) is the focus.
tools:
- name: R
  role: Environment in which the GDSC1000 data is loaded, the B-cell panel is filtered,
    and CombinationDrugMyeloma functions are called
- name: CombinationDrugMyeloma
  role: R package supplying AssignGroup(x, l) for S/R/N categorization and CombinationDrugs(dir,
    file, dest) for regimen coverage computation
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Optimization-regularization greedy set-cover algorithm that consumes the categorized
    panel to select PI-backbone secondary drugs
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:74a83f6d1e8ca92d0da1582ff40823f532f6ffaed3bec86d14bf4f024ea855b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-line-panel-filtering-b-cell-cancer

## Summary

Subsets a public pharmacogenomics cell-line screen (GDSC1000 from cancerrxgene.org) to the computable B-cell cancer cell-line panel (94 lines) and categorizes proteasome-inhibitor (PI) IC50 responses into sensitive/resistant/null groups, providing the input universe for secDrug regimen-coverage computation.

## When to use

Use when a GDSC1000-style pharmacogenomics screen must be subset to a defined cancer-type cell-line panel (e.g., B-cell cancers) before grouping drug responses (S/R/N) and computing percent coverage of drug-combination regimens.

## When NOT to use

- The goal is single-cell screening of secDrugs by subclonal drug-target enrichment (scRNAseq) or CyTOF proteomics — this skill handles pharmacogenomics panel subsetting, not single-cell multi-omics.
- Cell-line responses have already been categorized into S/R/N groups and regimen coverage has been computed — re-filtering would double-count or distort the coverage denominator.
- The target indication is not represented in the GDSC1000 B-cell cancer panel or a different backbone drug class (e.g., IMiD) is the focus.

## Inputs

- GDSC1000 pharmacogenomics dataset (IC50 values per cell line per drug) downloaded from cancerrxgene.org
- List of computable B-cell cancer cell lines (94 lines in the secDrug analysis)
- Proteasome-inhibitor backbone IC50 values per cell line

## Outputs

- Filtered panel of 94 B-cell cancer cell lines with PI response metadata
- Per-cell-line PI response group assignments: sensitive (S), resistant (R), or null (N) from AssignGroup(x, l)
- secdrug_regimen_coverage.csv: top-28 PI-backbone regimens with percent cell-line coverage at 2 and 3 secDrugs

## How to apply

Download the public GDSC1000 pharmacogenomics dataset from cancerrxgene.org and load it in R. Filter the cell-line panel to the computable B-cell cancer lines (94 lines in the secDrug paper) that have measurable PI response. Use the AssignGroup(x, l) function exported by the CombinationDrugMyeloma R package to categorize each cell line's PI IC50 as sensitive (S), resistant (R), or null (N) relative to a response threshold. These categorized groups — not raw IC50s — are the input to the secDrug optimization-regularization greedy set-cover algorithm that selects secondary drugs for the PI backbone. Downstream, CombinationDrugs(dir, file, dest) computes percent cell-line coverage of the resulting top regimens over the filtered panel, so panel composition directly determines reported coverage percentages. The rationale is that an in vitro HMCL/cell-line panel represents inter-individual heterogeneity in drug-response/resistance, making panel composition the denominator for regimen-kill estimates.

## Related tools

- **R** (Environment in which the GDSC1000 data is loaded, the B-cell panel is filtered, and CombinationDrugMyeloma functions are called)
- **CombinationDrugMyeloma** (R package supplying AssignGroup(x, l) for S/R/N categorization and CombinationDrugs(dir, file, dest) for regimen coverage computation) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Optimization-regularization greedy set-cover algorithm that consumes the categorized panel to select PI-backbone secondary drugs) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
library(CombinationDrugMyeloma); groups <- AssignGroup(pi_ic50_vector, threshold); CombinationDrugs(dir = "gdsc1000", file = "bcell_panel.csv", dest = "secdrug_regimen_coverage.csv")
```

## Evaluation signals

- The filtered panel contains exactly the 94 computable B-cell cancer cell lines; spot-check a few GDSC1000 cell-line identifiers against the B-cell subset.
- AssignGroup(x, l) output contains only three categories (S, R, N) with no missing labels across the panel.
- Coverage invariants: PI-only regimen covers 33.0% of the panel; PI + FK866 + 17-AAG covers 72.2% with 2 secDrugs and 82.5% with 3 secDrugs.
- secdrug_regimen_coverage.csv contains the top-28 PI-backbone regimens with both 2-drug and 3-drug coverage percentages populated.
- Coverage percentages computed against the 94-line denominator are monotone non-decreasing as secDrugs are added.

## Limitations

- Coverage estimates are only as representative as the HMCL/panel composition; the paper frames the panel as an in vitro model of inter-individual heterogeneity, not clinical response.
- The pipeline is currently limited to two-drug combinations; model systems for testing more than two secDrugs together are not yet developed.
- Testing of predicted secDrugs in a wider HMCL panel representing refractory and clonally derived acquired resistant lines is not yet done; supplementary Methods/Tables with filter thresholds are referenced but not included/verifiable in the provided text.

## Evidence

- [intro] we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance to show that the top predicted secondary secDrugs are indeed effective: "we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance to show that the top predicted secondary secDrugs are indeed effective"
- [abstract] a computational pipeline that uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma: "a computational pipeline that uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma"
- [abstract] we used a panel of human myeloma cell lines representing drug-sensitive, innate/refractory, and acquired/relapsed PI- and IMiD resistance: "we used a panel of human myeloma cell lines representing drug-sensitive, innate/refractory, and acquired/relapsed PI- and IMiD resistance"
- [intro] We have compiled a panel of >70 human myeloma cell lines (HMCLs) representing the broad spectrum of biological and genetic heterogeneity of myeloma patients: "We have compiled a panel of >70 human myeloma cell lines (HMCLs) representing the broad spectrum of biological and genetic heterogeneity of myeloma patients"

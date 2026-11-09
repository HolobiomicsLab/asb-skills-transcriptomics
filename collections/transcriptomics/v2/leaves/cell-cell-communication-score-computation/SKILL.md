---
name: cell-cell-communication-score-computation
description: Use when you need to quantify communication potential between an input
  cell population and partner cell types from transcriptomic data, e.g. comparing
  outward signaling of CAF-S1 vs CAF-S4 or LPS-stimulated DCs against the Primary
  Cell Atlas.
when_to_use_negative:
- The analysis requires chemokine or immune-checkpoint interactions — ICELLNET's curated
  database is very specific and exhaustive for cytokines but does not yet include
  chemokine and checkpoint interactions
- The question concerns downstream intracellular signaling effectors — these are deliberately
  not integrated in ICELLNET at this stage
- Non-human species data — the ICELLNET database is curated exclusively from human
  studies
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ICELLNET
  role: 'Core framework: expert-curated ligand-receptor database, icellnet scoring
    function, and visualization of communication scores'
  repo: https://github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Installs the icellnet package from GitHub via install_github('soumelis-lab/ICELLNET',
    ref='master', subdir='icellnet')
  repo: https://github.com/soumelis-lab/ICELLNET
- name: hgu133plus2.db
  role: Maps Affymetrix microarray probe identifiers to gene symbols before scoring
    (DC microarray workflow)
- name: jetset
  role: CRAN dependency of ICELLNET used in its installation procedure
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:a62a74e13efb93978cd13892bec86af8c1afe1257a59d79f5781fbff43697e1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - devtools
  - hgu133plus2.db
  - jetset
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cell-cell-communication-score-computation

## Summary

Compute ICELLNET cell-to-cell communication scores by connecting a cell population's transcriptome (bulk RNA-seq, microarray, or scRNA-seq) to reference partner transcriptomes using a manually curated human ligand-receptor interaction database (1669 interactions) that accounts for ligand/receptor complex subunits.

## When to use

Use when you need to quantify communication potential between an input cell population and partner cell types from transcriptomic data, e.g. comparing outward signaling of CAF-S1 vs CAF-S4 or LPS-stimulated DCs against the Primary Cell Atlas.

## When NOT to use

- The analysis requires chemokine or immune-checkpoint interactions — ICELLNET's curated database is very specific and exhaustive for cytokines but does not yet include chemokine and checkpoint interactions
- The question concerns downstream intracellular signaling effectors — these are deliberately not integrated in ICELLNET at this stage
- Non-human species data — the ICELLNET database is curated exclusively from human studies

## Inputs

- Gene-symbol-indexed bulk RNA-seq expression matrix per cell population (e.g. CAF-S1 n=6, CAF-S4 n=3)
- Microarray expression matrix with probe IDs mapped to gene symbols via hgu133plus2.db (e.g. Medium, LPS, LPS+aTNFR, LPS+aIL-10R DC conditions)
- Partner-cell reference transcriptomes: Human Primary Cell Atlas (33 reference human cell types)
- ICELLNET ligand-receptor interaction database shipped with the package (1669 human interactions)

## Outputs

- Raw and normalized ICELLNET communication scores per source cell / partner cell pair (icellnet_connectivity_scores.csv)
- Rescaled outward communication score table with per-partner comparisons (caf_s1_s4_icellnet_scores.csv)
- Communication scores decomposed by molecular family for visualization

## How to apply

Install the icellnet R package (devtools::install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet')) with its CRAN (jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table) and Bioconductor (BiocGenerics, org.Hs.eg.db, hgu133plus2.db, annotate) dependencies, matching the R version constraint (ICELLNET v2.0.0 needs R < 4.1; use v2.2.x for R > 4.1.x). Preprocess the expression matrix into a gene-symbol-indexed format (mapping microarray probes via hgu133plus2.db), load the expert-curated ligand-receptor database (1669 human interactions with logical rules for co-expression of protein subunits), and run the icellnet scoring function to connect the transcriptome of interest to the Primary Cell Atlas reference (33 human cell types). Rescale the outward (emitted) scores per cell population so scores are comparable across subsets, then rank and compare partner cells, e.g. verifying CAF-S1 > CAF-S4 outward scores toward epithelial (6 vs 3) and endothelial (7 vs 4) cells. The expert manual curation is the rationale: it adds biologically relevant, experimentally validated interactions (e.g. MIF/CXCR2 and MIF/CXCR4 absent from CellPhoneDB) missing from uncurated databases.

## Related tools

- **ICELLNET** (Core framework: expert-curated ligand-receptor database, icellnet scoring function, and visualization of communication scores) — https://github.com/soumelis-lab/ICELLNET
- **devtools** (Installs the icellnet package from GitHub via install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet')) — https://github.com/soumelis-lab/ICELLNET
- **hgu133plus2.db** (Maps Affymetrix microarray probe identifiers to gene symbols before scoring (DC microarray workflow))
- **jetset** (CRAN dependency of ICELLNET used in its installation procedure)

## Examples

```
install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")
```

## Evaluation signals

- Output table icellnet_connectivity_scores.csv exists and contains one score row per condition/reference pair (33 Primary Cell Atlas cell types)
- For the DC microarray validation: LPS shows increased connectivity relative to Medium, and LPS+aIL-10R shows the largest amplification of scores relative to LPS across all 12 cellular targets
- For the CAF comparison: rescaled outward scores are higher for CAF-S1 than CAF-S4 for epithelial (6 vs 3), endothelial (7 vs 4), pDC (6 vs 4), and B cells (3 vs 1)
- Scores reflect only interactions in the curated 1669-interaction database, with ligand/receptor complex subunits respected via co-expression logical rules
- R version compatibility check passed: ICELLNET v2.0.0 under R < 4.1, or ICELLNET v2.2.x under R > 4.1.x

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; latest R versions (> 4.1.x) require ICELLNET v2.2.x
- ICELLNET includes a relatively small number of interactions compared to other databases; chemokine and checkpoint interactions are not yet included
- Some receptors without known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and ligands without known receptors (IL17D) remain
- Predicted interactions such as JAG1-Notch and PDGF in CAF-S4 warrant further experimental validation studies

## Evidence

- [readme] Manually curated ligand-receptor database accounting for ligand and receptor complex subunits drives the scoring: "curated exclusively from **human** studies, ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes (more than 1600 interactions)"
- [methods] Installation command for the package: "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [methods] R version constraint for v2.0.0 vs v2.2.x: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [discussion] Use of Human Primary Cell Atlas as reference partner transcriptomes: "By using the Human Primary Cell Atlas as a reference for transcriptomic profiles17,18, ICELLNET allows us to integrate cell communication partners"
- [methods] Database extension to 1669 interactions in v2: "New structure and extension of ICELLNET database up to 1669 interactions."

---
name: stimulus-condition-comparative-analysis
description: Use when multiple transcriptomic conditions from one cell type under
  distinct stimuli/blocking treatments must be compared for ligand-receptor communication
  amplitude against a common reference panel.
when_to_use_negative:
- Input is single-cell RNA-seq data from multiple populations in the same sample rather
  than bulk/stimulus-condition transcriptomes — use the ICELLNET scRNAseq tutorial
  workflow instead
- The goal is inference of downstream signaling effectors rather than ligand-receptor
  communication, since ICELLNET explicitly does not integrate downstream effectors
- Non-human transcriptomic data, since the ICELLNET ligand-receptor database is curated
  exclusively from human studies
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ICELLNET
  role: Computes cell-cell communication scores between each stimulus-condition transcriptome
    and the Primary Cell Atlas reference using the curated ligand-receptor database
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Runtime environment for the ICELLNET scoring pipeline (v2.0.0 requires R <
    4.1; use ICELLNET v2.2.x for R > 4.1.x)
  repo: github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Installs the icellnet package from GitHub via install_github
  repo: github.com/soumelis-lab/ICELLNET
- name: hgu133plus2.db
  role: Bioconductor annotation package used to map Affymetrix HG-U133 Plus 2.0 probe
    identifiers to gene symbols before scoring
- name: jetset
  role: CRAN dependency installed alongside icellnet for microarray probe-to-gene
    mapping quality
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:3d28a144e75f96366fc76de521a30cf5c4777162450898c601889199ecee25cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - devtools
  - hgu133plus2.db
  - jetset
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# stimulus-condition-comparative-analysis

## Summary

Compares ICELLNET-derived cell-cell communication scores across stimulus perturbation conditions (e.g., Medium, LPS, LPS+aTNFR, LPS+aIL-10R on monocyte-derived dendritic cells) to identify which autocrine loop controls communication amplitude with partner cell types.

## When to use

Use when multiple transcriptomic conditions from one cell type under distinct stimuli/blocking treatments must be compared for ligand-receptor communication amplitude against a common reference panel.

## When NOT to use

- Input is single-cell RNA-seq data from multiple populations in the same sample rather than bulk/stimulus-condition transcriptomes — use the ICELLNET scRNAseq tutorial workflow instead
- The goal is inference of downstream signaling effectors rather than ligand-receptor communication, since ICELLNET explicitly does not integrate downstream effectors
- Non-human transcriptomic data, since the ICELLNET ligand-receptor database is curated exclusively from human studies

## Inputs

- Microarray (hgu133plus2) expression matrices for each stimulus condition (Medium, LPS, LPS+aTNFR, LPS+aIL-10R) of monocyte-derived dendritic cells
- ICELLNET manually curated ligand-receptor interaction database (1669 human interactions, with ligand/receptor subunit rules)
- Primary Cell Atlas reference transcriptome of 33 human cell types

## Outputs

- icellnet_connectivity_scores.csv with one normalized score row per condition/reference cell-type pair
- Per-condition table of communication scores across all communication families
- Per-target amplification ranking identifying which condition (LPS+aIL-10R vs LPS+aTNFR) maximally increases communication with each of the 12 partner cell types

## How to apply

Load the microarray expression matrices for all conditions (Medium, LPS, LPS+aTNFR, LPS+aIL-10R) in R and map probe identifiers to gene symbols using the hgu133plus2.db annotation package. Install ICELLNET from GitHub (devtools::install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet")) together with its CRAN dependencies (jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table). Connect each condition's transcriptome to the Primary Cell Atlas reference transcriptome of 33 human cell types using the icellnet scoring function and the manually curated database of 1669 human ligand-receptor interactions that accounts for ligand and receptor subunits. Normalize the connectivity scores and assemble a per-condition table of scores across all communication families so conditions are directly comparable. The decision point is the pairwise contrast: compare each perturbed condition against LPS to attribute amplification (or loss) of global communication to a specific autocrine loop — here blocking the IL-10 loop (aIL-10R) versus the TNF loop (aTNFR).

## Related tools

- **ICELLNET** (Computes cell-cell communication scores between each stimulus-condition transcriptome and the Primary Cell Atlas reference using the curated ligand-receptor database) — github.com/soumelis-lab/ICELLNET
- **R** (Runtime environment for the ICELLNET scoring pipeline (v2.0.0 requires R < 4.1; use ICELLNET v2.2.x for R > 4.1.x)) — github.com/soumelis-lab/ICELLNET
- **devtools** (Installs the icellnet package from GitHub via install_github) — github.com/soumelis-lab/ICELLNET
- **hgu133plus2.db** (Bioconductor annotation package used to map Affymetrix HG-U133 Plus 2.0 probe identifiers to gene symbols before scoring)
- **jetset** (CRAN dependency installed alongside icellnet for microarray probe-to-gene mapping quality)

## Examples

```
library(devtools); install.packages(c("devtools","jetset","readxl","psych","GGally","gplots","ggplot2","RColorBrewer","data.table")); install_github("soumelis-lab/ICELLNET",ref="master",subdir="icellnet")
```

## Evaluation signals

- Output file icellnet_connectivity_scores.csv exists and contains one score row per condition/reference pair (4 conditions x 33 reference cell types)
- LPS condition shows increased connectivity relative to Medium, consistent with LPS activation of DCs
- LPS+aIL-10R shows the largest amplification of communication scores relative to LPS across all 12 cellular targets, reproducing the published autocrine IL-10 result
- LPS+aTNFR shows negligible change in the global communication score relative to LPS
- All probe identifiers are successfully mapped to gene symbols via hgu133plus2.db before icellnet scoring is run

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x
- ICELLNET includes a relatively small number of interactions compared to other databases; chemokine and checkpoint interactions are not yet included
- Downstream effectors are not integrated in ICELLNET at this stage
- Some receptors lack known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and some ligands lack known receptors (e.g., IL17D)

## Evidence

- [results] Blocking the IL-10 loop in LPS-activated DCs produced the largest amplification of communication with all 12 cellular targets, whereas blocking the TNF loop had a negligible effect on the global communication score.: [evidence span withheld — anchor status: not_found]
- [methods] Primary Cell Atlas reference: "with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types)"
- [methods] ICELLNET database size: "New structure and extension of ICELLNET database up to 1669 interactions."
- [readme] ICELLNET installation: "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [methods] R version constraint: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."

---
name: bulk-transcriptome-connectivity-normalization
description: Use when you have one or more bulk cell-population transcriptomes (microarray
  probes or gene-level RNA-seq) and need to quantify their communication potential
  with many partner cell types via ligand-receptor scoring rather than co-expression
  of genes across cells.
when_to_use_negative:
- Input is single-cell data where per-cell-type expression can be averaged within
  a sample — use the dedicated ICELLNET scRNAseq workflow instead
- You need chemokine or immune-checkpoint interactions, which are not yet included
  in the ICELLNET database
- The transcriptome is non-human; the curated database is derived exclusively from
  human studies
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ICELLNET
  role: Computes ligand-receptor-based connectivity scores between the condition transcriptome
    and the Primary Cell Atlas reference, using its 1669-interaction curated database
  repo: https://github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Installs the icellnet package from GitHub (install_github with ref="master",
    subdir="icellnet")
  repo: https://github.com/soumelis-lab/ICELLNET
- name: jetset
  role: CRAN dependency installed alongside icellnet; supports microarray probe-to-gene
    mapping for Affymetrix arrays
- name: hgu133plus2.db
  role: Bioconductor annotation package used to map HG-U133 Plus 2.0 microarray probe
    identifiers to gene symbols before scoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:cfced8801d57028cd509ea65e89692dcf1a6ad6c5800b255929d1d4e46aa6754
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bulk-transcriptome-connectivity-normalization

## Summary

Scores a bulk transcriptome (e.g., Affymetrix microarray or RNA-seq) of one cell type against a reference atlas using ICELLNET's manually curated ligand-receptor database, then normalizes connectivity scores per condition to quantify and compare intercellular communication.

## When to use

Use when you have one or more bulk cell-population transcriptomes (microarray probes or gene-level RNA-seq) and need to quantify their communication potential with many partner cell types via ligand-receptor scoring rather than co-expression of genes across cells.

## When NOT to use

- Input is single-cell data where per-cell-type expression can be averaged within a sample — use the dedicated ICELLNET scRNAseq workflow instead
- You need chemokine or immune-checkpoint interactions, which are not yet included in the ICELLNET database
- The transcriptome is non-human; the curated database is derived exclusively from human studies

## Inputs

- Bulk transcriptome expression matrix (microarray probe-level or gene-level RNA-seq) for one or more cell-type conditions
- Probe-to-gene annotation via Bioconductor packages (e.g., hgu133plus2.db) or pre-mapped gene symbols
- ICELLNET manually curated human ligand-receptor interaction database (1669 interactions, shipped with the package)
- Reference transcriptome atlas (Human Primary Cell Atlas, 33 reference cell types, provided with icellnet)

## Outputs

- Raw and normalized ICELLNET connectivity scores per condition/reference-cell-type pair
- Table of connectivity scores across all communication families per condition (e.g., icellnet_connectivity_scores.csv)
- Comparative connectivity profiles across experimental conditions for downstream visualization

## How to apply

Map platform probe identifiers to gene symbols with the appropriate annotation package (e.g., hgu133plus2.db for HG-U133 Plus 2.0 microarrays, using jetset for probe selection), then assemble a gene-by-sample expression matrix per condition. Install icellnet from GitHub via devtools (install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet")) together with CRAN dependencies (jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table) and load the package's manually curated ligand-receptor database (1669 human interactions accounting for ligand and receptor subunits). Run the icellnet scoring function between each condition's transcriptome and the Primary Cell Atlas reference transcriptome of 33 human cell types, respecting the database's logical rules for co-expression of multimeric ligand/receptor subunits. Normalize the raw connectivity scores and assemble a per-condition table of scores across all communication families (e.g., cytokines, TNF family, growth factors) so conditions are directly comparable. In the DC use case, four conditions (Medium, LPS, LPS+aTNFR, LPS+aIL-10R) are scored and compared to detect which autocrine loop amplifies communication with 12 target cell types.

## Related tools

- **ICELLNET** (Computes ligand-receptor-based connectivity scores between the condition transcriptome and the Primary Cell Atlas reference, using its 1669-interaction curated database) — https://github.com/soumelis-lab/ICELLNET
- **devtools** (Installs the icellnet package from GitHub (install_github with ref="master", subdir="icellnet")) — https://github.com/soumelis-lab/ICELLNET
- **jetset** (CRAN dependency installed alongside icellnet; supports microarray probe-to-gene mapping for Affymetrix arrays)
- **hgu133plus2.db** (Bioconductor annotation package used to map HG-U133 Plus 2.0 microarray probe identifiers to gene symbols before scoring)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")
```

## Evaluation signals

- The output file (e.g., icellnet_connectivity_scores.csv) exists and contains one score row per condition/reference-cell-type pair
- Scores are reported across all communication families for every condition, not only a global aggregate
- Positive control directionality: LPS-activated DCs show increased connectivity relative to Medium
- Perturbation check: LPS+aIL-10R yields the largest amplification of connectivity relative to LPS, while LPS+aTNFR has a negligible effect on the global score
- Normalized scores are on a comparable scale across all four conditions before any ranking or comparison

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x, use ICELLNET v2.2.x
- ICELLNET includes a relatively small number of interactions compared to other databases; chemokine and checkpoint interactions are not yet included
- Predicted connectivity is not per-interaction experimentally validated; downstream signaling effects are not integrated, which can affect predicted responses to combined stimuli

## Evidence

- [abstract] We develop ICELLNET, a transcriptomic-based framework integrating: 1) an original expert-curated database of ligand-receptor interactions: "We develop ICELLNET, a transcriptomic-based framework integrating: 1) an original expert-curated database of ligand-receptor interactions"
- [methods] with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types): "with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types)"
- [methods] New structure and extension of ICELLNET database up to 1669 interactions.: "New structure and extension of ICELLNET database up to 1669 interactions."
- [readme] install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"): "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [abstract] ICELLNET reveals autocrine IL-10 control of human dendritic cell communication with up to 12 cell types.: "ICELLNET reveals autocrine IL-10 control of human dendritic cell communication with up to 12 cell types."

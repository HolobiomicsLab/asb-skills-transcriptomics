---
name: caf-subset-communication-comparison
description: Use when bulk RNA-seq or microarray expression matrices of CAF-S1 (n=6)
  and CAF-S4 (n=3) subsets are available and the goal is to test whether one subset
  emits stronger signals to epithelial, endothelial, pDC, or B cell partners via ICELLNET
  scoring.
when_to_use_negative:
- Input is single-cell RNA-seq data from multiple populations of the same sample —
  use the ICELLNET scRNAseq tutorial or the sorted cell-population script (github.com/soumelis-lab/ICELLNET/issues/12)
  instead
- The goal is a full two-sided interaction network between many cell types rather
  than comparing emitted (outward) signaling from one cell type of interest against
  reference transcriptomes
- The organism is non-human — the ICELLNET ligand-receptor database is curated exclusively
  from human studies
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ICELLNET
  role: Computes transcriptome-based cell-cell communication scores between CAF subsets
    and HPCA partner cells using the expert-curated ligand-receptor database; outward
    scores are then rescaled for comparison
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Runtime environment; ICELLNET v2.0.0 requires R < 4.1, v2.2.x is required
    for R > 4.1.x
- name: devtools
  role: Provides install_github used to install the icellnet package from the soumelis-lab
    repository
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:72b76ddcaf5a72b87550a70662e4f3fc57944538472d2bdb8a25d08fed580d66
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# caf-subset-communication-comparison

## Summary

A method for comparing the outward (emitted) communication potential of CAF-S1 versus CAF-S4 fibroblast subsets toward tumor microenvironment partner cells, computed with the ICELLNET transcriptome-based framework using its expert-curated human ligand-receptor database and the Human Primary Cell Atlas reference transcriptomes.

## When to use

Use when bulk RNA-seq or microarray expression matrices of CAF-S1 (n=6) and CAF-S4 (n=3) subsets are available and the goal is to test whether one subset emits stronger signals to epithelial, endothelial, pDC, or B cell partners via ICELLNET scoring.

## When NOT to use

- Input is single-cell RNA-seq data from multiple populations of the same sample — use the ICELLNET scRNAseq tutorial or the sorted cell-population script (github.com/soumelis-lab/ICELLNET/issues/12) instead
- The goal is a full two-sided interaction network between many cell types rather than comparing emitted (outward) signaling from one cell type of interest against reference transcriptomes
- The organism is non-human — the ICELLNET ligand-receptor database is curated exclusively from human studies

## Inputs

- CAF-S1 bulk RNA-seq expression matrix (n=6 samples)
- CAF-S4 bulk RNA-seq expression matrix (n=3 samples)
- Human Primary Cell Atlas reference transcriptomes (database of reference transcriptomes of 33 cell types) as partner-cell profiles

## Outputs

- Rescaled outward ICELLNET communication score table per CAF subset and partner cell type (caf_s1_s4_icellnet_scores.csv)
- Per-partner comparisons of CAF-S1 vs CAF-S4 scores (epithelial 6 vs 3, endothelial 7 vs 4, pDC 6 vs 4, B cells 3 vs 1)

## How to apply

Install ICELLNET with its CRAN dependencies (devtools, jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table, grid, gridExtra, ggthemes, scales, rlist) and Bioconductor dependencies (BiocGenerics, org.Hs.eg.db, hgu133plus2.db, annotate) via install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet'), respecting the R-version constraint (v2.0.0 needs R < 4.1; use v2.2.x for R > 4.1.x). Preprocess the CAF-S1 and CAF-S4 expression matrices into gene-symbol-indexed format and load the Human Primary Cell Atlas reference transcriptomes as partner-cell profiles. Run the ICELLNET scoring step, which uses the expert-curated human ligand-receptor database (1669 interactions in v2) that accounts for multiple subunits of ligand and receptor complexes via logical co-expression rules. Rescale the outward (emitted) scores per CAF subset so they are comparable across subsets, then rank partner cells by score and compare CAF-S1 vs CAF-S4 per partner. The expected direction of the result is CAF-S1 > CAF-S4, significantly toward epithelial cells (6 vs 3) and endothelial cells (7 vs 4), p < 0.1, with CAF-S1 also higher toward pDC (6 vs 4) and B cells (3 vs 1). Write the rescaled score table with per-partner comparisons to caf_s1_s4_icellnet_scores.csv.

## Related tools

- **ICELLNET** (Computes transcriptome-based cell-cell communication scores between CAF subsets and HPCA partner cells using the expert-curated ligand-receptor database; outward scores are then rescaled for comparison) — github.com/soumelis-lab/ICELLNET
- **R** (Runtime environment; ICELLNET v2.0.0 requires R < 4.1, v2.2.x is required for R > 4.1.x)
- **devtools** (Provides install_github used to install the icellnet package from the soumelis-lab repository)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet")
```

## Evaluation signals

- Rescaled outward scores for CAF-S1 exceed those of CAF-S4 for epithelial cells (6 vs 3), endothelial cells (7 vs 4), pDC (6 vs 4), and B cells (3 vs 1)
- Statistical significance of CAF-S1 vs CAF-S4 differences toward epithelial and endothelial cells reaches p < 0.1
- Scores are rescaled (comparable in magnitude) across the CAF-S1 and CAF-S4 subsets before ranking partner cells
- Output CSV contains the per-partner communication score comparisons as specified in the workflow
- The ICELLNET version matches the R version: v2.0.0 only on R < 4.1, v2.2.x on R > 4.1.x

## Limitations

- ICELLNET's database includes a relatively small number of interactions compared to other databases; chemokine and checkpoint interactions are not yet included
- The database lacks some interactions present in other resources (e.g., 14 interactions in ICELLNET are absent from CellPhoneDB, and vice versa); 14 interactions included in ICELLNET were not in CellPhoneDB
- Some CAF-S4 interactions (JAG1-Notch, PDGF) warrant further experimental validation
- Downstream effectors are not integrated in ICELLNET, and combining it with downstream-signaling approaches can lead to unpredictable false positives and false negatives
- Version constraint: ICELLNET v2.0.0 requires R version < 4.1; use v2.2.x for R > 4.1.x

## Evidence

- [methods] Rescaled outward scores and CAF-S1 > CAF-S4 direction: "we develop ICELLNET, a versatile computational framework to infer cell-to-cell communication from a wide range of bulk and single-cell transcriptomic datasets."
- [discussion] Database and subunit logic: "Similar to CellPhoneDB10, ICELLNET takes into account the multiple subunits of ligands and receptors, by introducing logical rules for co-expression of protein subunits."
- [methods] Database size: "New structure and extension of ICELLNET database up to 1669 interactions."
- [discussion] Partner reference: "By using the Human Primary Cell Atlas as a reference for transcriptomic profiles17,18, ICELLNET allows us to integrate cell communication partners (sender or receiver) not included in a given"
- [readme] R version constraint: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [readme] CAF tutorial grounding: "infer cell-cell communication from the transcriptome of a single cell type of interest with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types)."
- [readme] Database curation scope: "curated exclusively from **human** studies, ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes"

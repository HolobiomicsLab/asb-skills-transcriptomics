---
name: microarray-probe-to-gene-annotation
description: Use when input expression data are Affymetrix HG-U133 Plus 2.0 probe-level
  matrices (e.g., microarray CEL-derived probe IDs) that must be collapsed or mapped
  to gene symbols before downstream gene-name-based analysis.
when_to_use_negative:
- Input data are RNA-seq or single-cell RNA-seq counts already indexed by gene symbols
  or Ensembl IDs — no Affymetrix probe mapping is needed
- Arrays are not Affymetrix HG-U133 Plus 2.0 — hgu133plus2.db mappings would be incorrect
  for other platforms
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: hgu133plus2.db
  role: Bioconductor annotation resource providing Affymetrix HG-U133 Plus 2.0 probe-to-gene-symbol
    mappings used to convert probe-level DC microarray matrices into gene-symbol space
- name: jetset
  role: ICELLNET CRAN dependency used alongside hgu133plus2.db for jetset-based probe-quality
    assessment / best-probe selection when collapsing probes to gene symbols
- name: BiocManager
  role: Installer used to fetch hgu133plus2.db and the other Bioconductor dependencies
    (BiocGenerics, org.Hs.eg.db, annotate)
- name: ICELLNET
  role: Downstream framework that requires gene-symbol input; its curated 1669-interaction
    ligand-receptor database is intersected with the annotated transcriptome to compute
    communication scores
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Environment in which the probe-to-gene annotation and subsequent ICELLNET
    scoring are performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:300f7155d0f9997c68858c4b4f31ea037acb9c8915e8119ea9428adf5b66b55b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - ICELLNET
  - R
  - devtools
  - hgu133plus2.db
  - jetset
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# microarray-probe-to-gene-annotation

## Summary

Maps Affymetrix HG-U133 Plus 2.0 microarray probe identifiers to official gene symbols using the Bioconductor annotation package hgu133plus2.db (with jetset for probe-quality filtering) so that probe-level expression matrices can be scored by gene-symbol-based frameworks such as ICELLNET.

## When to use

Use when input expression data are Affymetrix HG-U133 Plus 2.0 probe-level matrices (e.g., microarray CEL-derived probe IDs) that must be collapsed or mapped to gene symbols before downstream gene-name-based analysis.

## When NOT to use

- Input data are RNA-seq or single-cell RNA-seq counts already indexed by gene symbols or Ensembl IDs — no Affymetrix probe mapping is needed
- Arrays are not Affymetrix HG-U133 Plus 2.0 — hgu133plus2.db mappings would be incorrect for other platforms

## Inputs

- Probe-level expression matrix for the Medium condition (Affymetrix HG-U133 Plus 2.0 probe IDs as row identifiers)
- Probe-level expression matrices for LPS, LPS+aTNFR, and LPS+aIL-10R conditions of human monocyte-derived dendritic cells
- Bioconductor annotation package hgu133plus2.db (HG-U133 Plus 2.0 probe-to-gene mappings)

## Outputs

- Gene-symbol-indexed expression matrices (one per condition) suitable for ICELLNET icellnet scoring
- Explicit probe-to-gene-symbol mapping table documenting multi-mapping probes

## How to apply

Load the raw probe-level expression matrices for each condition (here: Medium, LPS, LPS+aTNFR, LPS+aIL-10R monocyte-derived DC microarrays) in R and use hgu133plus2.db to map each Affymetrix probe ID to its official human gene symbol. Optionally use jetset, listed as an ICELLNET CRAN dependency, to select the best probe per gene when multiple probes map to one symbol. Install hgu133plus2.db via BiocManager along with the other Bioconductor dependencies (BiocGenerics, org.Hs.eg.db, annotate). The rationale is that ICELLNET scoring intersects the input transcriptome with its 1669-interaction curated ligand-receptor database using gene symbols, so unmapped probes would be silently dropped from communication-score computation. Retain the one-to-one or many-to-one probe-to-symbol mapping explicitly so gene-level aggregation is reproducible across the four conditions.

## Related tools

- **hgu133plus2.db** (Bioconductor annotation resource providing Affymetrix HG-U133 Plus 2.0 probe-to-gene-symbol mappings used to convert probe-level DC microarray matrices into gene-symbol space)
- **jetset** (ICELLNET CRAN dependency used alongside hgu133plus2.db for jetset-based probe-quality assessment / best-probe selection when collapsing probes to gene symbols)
- **BiocManager** (Installer used to fetch hgu133plus2.db and the other Bioconductor dependencies (BiocGenerics, org.Hs.eg.db, annotate))
- **ICELLNET** (Downstream framework that requires gene-symbol input; its curated 1669-interaction ligand-receptor database is intersected with the annotated transcriptome to compute communication scores) — github.com/soumelis-lab/ICELLNET
- **R** (Environment in which the probe-to-gene annotation and subsequent ICELLNET scoring are performed)

## Examples

```
library(hgu133plus2.db); symbols <- unlist(as.list(hgu133plus2SYMBOL[rownames(expr)])); expr.gene <- aggregate(expr, by=list(gene=symbols), FUN=max)
```

## Evaluation signals

- Every row of the annotated matrix has a valid HG-U133 Plus 2.0 gene symbol; no residual 'AFFX-' control probes or NA symbols remain in the scored input
- Dimensionality decreases (or stays stable) relative to the probe-level matrix, consistent with many-to-one probe-to-symbol collapsing
- Annotated matrices for all four conditions (Medium, LPS, LPS+aTNFR, LPS+aIL-10R) share a consistent gene-symbol row ordering so ICELLNET scores are comparable
- Downstream sanity check passes: LPS shows increased connectivity relative to Medium, and LPS+aIL-10R shows the largest amplification relative to LPS, confirming the annotation preserved the expected biology

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x — annotation and scoring must be run in a compatible R environment
- Probe-to-gene mapping is platform-specific: hgu133plus2.db only applies to Affymetrix HG-U133 Plus 2.0 arrays, and some genes may lack a well-annotated probe, removing them from downstream ligand-receptor scoring
- The ICELLNET database is human-curated and human-specific; annotation output is only useful for the human interaction database (interactions curated exclusively from human studies)

## Evidence

- [other] Load the monocyte-derived DC microarray expression matrices ... in R and map probe identifiers to gene symbols using the hgu133plus2.db annotation package.: [evidence span withheld — anchor status: not_found]
- [readme] BiocManager installs hgu133plus2.db as a Bioconductor dependency: "BiocManager::install(c("BiocGenerics", "org.Hs.eg.db", "hgu133plus2.db", "annotate"))"
- [readme] ICELLNET applies to microarray among other transcriptomic technologies: "**Versatile** tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics)"
- [abstract] ICELLNET applied to microarray datasets per abstract finding: "We apply ICELLNET to three datasets generated through RNA-seq, single-cell RNA-seq, and microarray."
- [readme] ICELLNET database is human-curated with subunit logic, constraining what gene-symbol input is matched against: "curated exclusively from **human** studies, ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes (more than 1600 interactions)"

---
name: entrez-to-symbol-gene-identifier-mapping
description: Use when an stAPAminer RUD/APA count matrix or PACdataset is keyed by
  ENTREZID and downstream labeling, clustering, or DEAPA reporting needs gene SYMBOLs
  (mouse data).
when_to_use_negative:
- Non-mouse ST data — org.Mm.eg.db is mouse-specific; use the matching org.Xx.eg.db
  package instead
- Input matrices already keyed by gene SYMBOL or Ensembl IDs where re-mapping would
  add ambiguity or ID-collision noise
- Poly(A) site-level rows without a gene-level ENTREZID attribute, where no gene key
  exists to translate
tools:
- name: org.Mm.eg.db
  role: AnnotationDbi-based mapping of mouse ENTREZID to gene SYMBOL for APA matrix
    row keys
- name: stAPAminer
  role: Produces the ENTREZID-keyed RUD/APA matrices (computeAPAIndex) whose keys
    are mapped to SYMBOLs before clustering and DEAPA analysis
  repo: https://github.com/BMILAB/stAPAminer
- name: movAPA
  role: Stores the poly(A) site data as a PACdataset (APA.RDA) whose gene identifiers
    are the mapping source
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:273256288d6946bfe602c0339f09948774a9478686ab011f74e6331785db30b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - Seurat
  - movAPA
  - org.Mm.eg.db
  - scAPAtrap
  - stAPAminer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Entrez-to-Symbol gene identifier mapping

## Summary

Maps ENTREZID identifiers on poly(A) site / APA-index matrices to official mouse gene SYMBOLs using the org.Mm.eg.db annotation package, a required preprocessing step in the stAPAminer MOB workflow so that DEAPA/LSAPA results (e.g., Adarb2) are reported with human-readable gene symbols.

## When to use

Use when an stAPAminer RUD/APA count matrix or PACdataset is keyed by ENTREZID and downstream labeling, clustering, or DEAPA reporting needs gene SYMBOLs (mouse data).

## When NOT to use

- Non-mouse ST data — org.Mm.eg.db is mouse-specific; use the matching org.Xx.eg.db package instead
- Input matrices already keyed by gene SYMBOL or Ensembl IDs where re-mapping would add ambiguity or ID-collision noise
- Poly(A) site-level rows without a gene-level ENTREZID attribute, where no gene key exists to translate

## Inputs

- RUD_RAW matrix or PACdataset (APA.RDA) with rows/genes keyed by ENTREZID (mouse)
- org.Mm.eg.db annotation package (required dependency of stAPAminer)

## Outputs

- APA/RUD matrix and stAPAminer object with gene rownames mapped to official mouse SYMBOLs
- Symbol-keyed DEAPA/LSAPA/SVAPA gene lists (e.g., Adarb2)

## How to apply

After computing the raw RUD matrix with computeAPAIndex (rows are poly(A) sites/genes keyed by ENTREZID from the APA.RDA PACdataset), translate the ENTREZID keys to official mouse gene SYMBOLs using org.Mm.eg.db (e.g., mapIds/AnnotationDbi, keytype='ENTREZID', column='SYMBOL'). This is applied per replicate (Replicates 5, 11, 12 of the MOB ST data) so that spot-level APA profiles, imputed matrices, and DEAPA gene lists share consistent SYMBOL keys. Retain the ENTREZID column as a backup key for join-back to the poly(A) site annotation, and handle unmapped IDs (drop or keep as NA) before createStAPAminerObject. Rationale: symbol-level keys are required for the layer-labeling and findContrastAPA/findstAPASet outputs to be interpretable and comparable across replicates.

## Related tools

- **org.Mm.eg.db** (AnnotationDbi-based mapping of mouse ENTREZID to gene SYMBOL for APA matrix row keys)
- **stAPAminer** (Produces the ENTREZID-keyed RUD/APA matrices (computeAPAIndex) whose keys are mapped to SYMBOLs before clustering and DEAPA analysis) — https://github.com/BMILAB/stAPAminer
- **movAPA** (Stores the poly(A) site data as a PACdataset (APA.RDA) whose gene identifiers are the mapping source)

## Examples

```
library(org.Mm.eg.db); sym <- mapIds(org.Mm.eg.db, keys=as.character(rownames(RUD_RAW)), column="SYMBOL", keytype="ENTREZID")
```

## Evaluation signals

- All SYMBOL-mapped rows are valid mouse gene symbols with no duplicated symbols colliding across distinct ENTREZIDs
- Spot-level Pearson correlations between replicates (≈0.87 for Replicate 5, ≈0.86 for Replicate 12 vs Replicate 11) are unchanged by identifier remapping
- Known DEAPA genes are recovered under SYMBOL keys, e.g., Adarb2 differential APA between GCL and ONL
- Number of genes/rows is preserved after mapping except for explicitly dropped unmapped IDs

## Limitations

- No software version numbers were reported for any tool, including the annotation package, in the stAPAminer workflow
- Identifier mapping is only defined for the mouse MOB example (org.Mm.eg.db); other organisms require a different org.Xx.eg.db resource
- Unmapped ENTREZIDs must be dropped or set to NA before imputation, which may silently lose poly(A) sites in poorly annotated loci

## Evidence

- [other] Quantify APA site usage per spot as the RUD index ... mapping ENTREZID to gene SYMBOL via org.Mm.eg.db.: [evidence span withheld — anchor status: not_found]
- [readme] org.Mm.eg.db is listed as a required R package for stAPAminer: "Required R Packages
* movAPA, org.Mm.eg.db, Seurat, ggplot2, fdm2id, ClusterR, cluster, clusterSim, SPARK, limma, edgeR, ggpubr"
- [readme] The APA-spot count matrix input to stAPAminer is a movAPA PACdataset loaded from APA.RDA: "## the APA-spot count matrix stored as a movAPA PACdataset
load(system.file("extdata", "APA.RDA", package = "stAPAminer"))"

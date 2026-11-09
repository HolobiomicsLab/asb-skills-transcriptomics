---
name: cluster-celltype-composition-analysis
description: Use when a scRNA-Seq matrix has known cell-type or technical-replicate
  labels (e.g. LC.PT vs LC.PT_RE) and you need to test whether clusters separate by
  batch or biology after a correction method (ccRemover, scLVM) has been applied.
when_to_use_negative:
- The cell-cycle activity is at a very low level (e.g. the Patel et al. glioblastoma
  data), where ccRemover used a smaller cutoff and clustering may be dominated by
  other effects
- The goal is cell-type discovery without reference labels — the skill requires known
  group labels to interpret cluster composition
- You want to remove batch effects themselves; ccRemover only removes the cell-cycle
  effect, so clustering may separate on other confounders that remain
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ccRemover
  role: Corrects the expression matrix by iteratively identifying and removing cell-cycle
    principal components before clustering
  repo: cran/ccRemover
- name: gene_indexer
  role: Maps gene names to cell-cycle gene indices (species='human', name_type='symbols')
    to build the if_cc control-gene indicator
- name: Biomart
  role: Downloads lists of genes annotated to the cell cycle for building the control-gene
    set
- name: org.Hs.eg.db
  role: Provides human gene annotations for identifying cell-cycle genes
- name: scLVM
  role: Alternative latent-variable correction applied for comparison so cluster composition
    can be evaluated across corrected and uncorrected data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:6a15a960a141c90cd55813aa48bc84711ee0f4c8c17afd641cf6b67976244302
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - ccRemover
  - gene_indexer
  - org.Hs.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cluster-celltype-composition-analysis

## Summary

Applies k-means clustering (e.g. 4-means on the Kim et al. lung adenocarcinoma scRNA-Seq data) to corrected and uncorrected single-cell expression matrices, then cross-tabulates cluster assignments against known cell-type/technical-replicate labels to reveal batch effects or biological structure.

## When to use

Use when a scRNA-Seq matrix has known cell-type or technical-replicate labels (e.g. LC.PT vs LC.PT_RE) and you need to test whether clusters separate by batch or biology after a correction method (ccRemover, scLVM) has been applied.

## When NOT to use

- The cell-cycle activity is at a very low level (e.g. the Patel et al. glioblastoma data), where ccRemover used a smaller cutoff and clustering may be dominated by other effects
- The goal is cell-type discovery without reference labels — the skill requires known group labels to interpret cluster composition
- You want to remove batch effects themselves; ccRemover only removes the cell-cycle effect, so clustering may separate on other confounders that remain

## Inputs

- scRNA-Seq count/expression matrix with cells as columns and genes as rows (e.g. GSE69405, 176 cells x 57,820 genes)
- Per-cell label vector of cell types or technical-replicate groups (LC.PT, LC.PT_RE, etc.)
- Cell-cycle gene annotation list for the relevant species (757 annotated cell-cycle genes in the human LUAD analysis)

## Outputs

- Cluster label assignment per cell for each data condition (original, scLVM-corrected, ccRemover-corrected)
- Cluster-vs-label cross-tabulation table (e.g. cluster_composition.csv) with per-cluster label fractions

## How to apply

Preprocess the count matrix: remove genes with zero expression in more than two thirds of cells (57,820 -> 10,977 genes in the Kim et al. LUAD data of 176 cells), then log-transform by adding 1 and taking the natural log, and center each gene to zero mean. Apply the correction under test — here ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10) after building the if_cc indicator for the 757 cell-cycle genes via gene_indexer with species='human' — and run the same clustering (4-means for the LUAD replicate test, 5 hierarchical clusters for glioblastoma) on original, scLVM-corrected, and ccRemover-corrected data. Cross-tabulate cluster assignments against the known labels (e.g. cluster_composition.csv) and compare composition fractions across corrected vs original data. Decision point: a cluster that is majority technical-replicate cells only after correction (cluster 3 ~80% LC.PT_RE, cluster 4 ~89% LC.PT) indicates the correction exposed a batch effect; separation on original data indicates true biological substructure.

## Related tools

- **ccRemover** (Corrects the expression matrix by iteratively identifying and removing cell-cycle principal components before clustering) — cran/ccRemover
- **gene_indexer** (Maps gene names to cell-cycle gene indices (species='human', name_type='symbols') to build the if_cc control-gene indicator)
- **Biomart** (Downloads lists of genes annotated to the cell cycle for building the control-gene set)
- **org.Hs.eg.db** (Provides human gene annotations for identifying cell-cycle genes)
- **scLVM** (Alternative latent-variable correction applied for comparison so cluster composition can be evaluated across corrected and uncorrected data)

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Cluster count matches the design: 4-means yields 4 clusters over 176 LUAD cells; hierarchical clustering splits glioblastoma cells into 5 clusters with majority-class labels
- On original and scLVM-corrected data, LC.PT and LC.PT_RE technical-replicate cells are non-separable across clusters (roughly uniform composition)
- After ccRemover correction, replicate separation appears: cluster 3 is ~80% LC.PT_RE cells and cluster 4 is ~89% LC.PT cells
- Gene filtering invariant: genes with zero expression in over two thirds of cells removed, reducing 57,820 to 10,977 genes with 757 annotated to the cell cycle
- Cross-tabulation saved as cluster_composition.csv with row sums equal to 176 cells

## Limitations

- ccRemover may miss the cell-cycle effect if it influences too few control genes (below ~8% affected), so cluster composition may not change
- ccRemover removes only the cell-cycle effect; other confounding effects may dominate the clustering after removal
- Reliance on annotation databases that are incomplete and inaccurate; a known set of cell-cycle genes is required

## Evidence

- [results] 4-means clustering on LUAD replicate test: "we consider using 4-means clustering, in order to see whether the 77 cells of the first cell type can be clustered accordingly to the two sets of technical replicates"
- [abstract] ccRemover as preprocessing before clustering: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
- [results] Gene filtering parameters: "any genes which had zero expression for over two thirds of the cells were removed from the data, leaving 10,977 genes of which 757 were annotated to the cell cycle"
- [methods] ccRemover call parameters: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [results] Compare across corrected and uncorrected conditions: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"

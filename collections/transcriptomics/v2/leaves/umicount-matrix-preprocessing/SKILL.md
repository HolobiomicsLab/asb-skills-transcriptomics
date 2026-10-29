---
name: umicount-matrix-preprocessing
description: Use when raw UMI count matrices from multiple individuals' droplet scRNA-seq
  libraries (10x Chromium) must be formatted and zero-gene filtered before population-level
  clustering with BAMM-SC.
when_to_use_negative:
- Data is a single individual's matrix rather than a multi-individual list — supply
  a matrix to BAMMSC/DIMMSC instead of treating it as a population problem
- Input is already normalized/log-transformed expression rather than raw UMI counts
  — BAMM-SC takes raw count data as input under a Dirichlet multinomial model
- The analysis targets rare cell types, where filtering genes with excessive zeros
  may discard the discriminating signal
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BAMMSC
  role: Target method that consumes the preprocessed per-individual UMI count list;
    validates formatting since it errors or falls back to DIMMSC if a matrix is passed
  repo: github.com/lichen-lab/BAMMSC
- name: Cell Ranger
  role: Upstream 10x pipeline that produces the UMI count matrices; its aggregation
    downsampling step should be avoided because it equalizes sequencing depth and
    loses information
- name: 10x Genomics Chromium system
  role: Droplet platform that generated the raw count data being preprocessed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:a4bf2c3be69c15cb773738baa3fcfa54faa5068ab0a781966148d05e340ec975
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - BAMMSC
  - Cell Ranger
  - CellTree
  - DIMM-SC
  - SC3
  - SIMLR
  - TSCAN
  - mclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# umicount-matrix-preprocessing

## Summary

Prepare raw droplet-based scRNA-seq UMI count matrices (e.g., from the 10x Genomics Chromium system) as a per-individual list with genes in rows and cells in columns, filtering out genes with excessive zeros so they can be clustered with BAMM-SC.

## When to use

Use when raw UMI count matrices from multiple individuals' droplet scRNA-seq libraries (10x Chromium) must be formatted and zero-gene filtered before population-level clustering with BAMM-SC.

## When NOT to use

- Data is a single individual's matrix rather than a multi-individual list — supply a matrix to BAMMSC/DIMMSC instead of treating it as a population problem
- Input is already normalized/log-transformed expression rather than raw UMI counts — BAMM-SC takes raw count data as input under a Dirichlet multinomial model
- The analysis targets rare cell types, where filtering genes with excessive zeros may discard the discriminating signal

## Inputs

- Raw per-individual UMI count matrices from 10x Chromium droplet scRNA-seq (genes x cells), e.g., produced by the Cell Ranger pipeline
- A single R list object X with one UMI count matrix per individual; number of genes identical across individuals, number of cells allowed to differ
- Optional: the bundled example data data_BAMMSC (10 individuals with 100 genes and 400 cells each) for a dry run

## Outputs

- A filtered, per-individual list of UMI count matrices suitable as the X argument of the BAMMSC function
- Cluster memberships per individual (mem) plus loglik, AIC, BIC, alpha, pi, sigma2 and posterior-probability (delta) outputs from the subsequent BAMMSC run

## How to apply

Assemble each individual's library as a UMI count matrix with rows as genes and columns as cells, then combine them into a single R list where the same gene set is shared across individuals while cell numbers may differ per individual. Filter out genes with excessive zeros, on the rationale that lowly-expressed genes contribute little to clustering. Avoid using Cell Ranger's aggregation downsampling, which equalizes sequencing depth across libraries and causes substantial information loss; instead keep raw per-individual counts so BAMM-SC's hierarchical model can absorb batch effects such as unbalanced sequencing depths. Feed the list directly to BAMMSC (e.g., with K=4 and default MCMC settings: maxIter.BAMMSC=500, likTol.BAMMSC=1e-10); if a single matrix is supplied instead of a list, BAMMSC will silently call DIMMSC.

## Related tools

- **BAMMSC** (Target method that consumes the preprocessed per-individual UMI count list; validates formatting since it errors or falls back to DIMMSC if a matrix is passed) — github.com/lichen-lab/BAMMSC
- **Cell Ranger** (Upstream 10x pipeline that produces the UMI count matrices; its aggregation downsampling step should be avoided because it equalizes sequencing depth and loses information)
- **10x Genomics Chromium system** (Droplet platform that generated the raw count data being preprocessed)

## Examples

```
install_github("lichen-lab/BAMMSC"); library(BAMMSC); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- All matrices in the output list have the same number of genes (rows) in the same order, with cell counts (columns) allowed to differ across individuals
- Genes with excessive zeros are removed from every individual's matrix consistently, so the gene sets remain aligned across the list
- Passing the list to BAMMSC(data, K) runs without coercion to DIMMSC — a matrix input would silently trigger single-individual DIMMSC behavior
- The MCMC log-likelihood trace converges within maxIter.BAMMSC=500 at likTol.BAMMSC=1e-10, and memberships are returned as a list with one vector per individual
- Downstream adjustedRandIndex against known labels (e.g., via mclust::adjustedRandIndex(unlist(result$mem), unlist(Z))) is computable, confirming memberships align with the original cell counts

## Limitations

- Filtering genes with excessive zeros 'may be problematic for rare cell type identification' because lowly-expressed genes are assumed uninformative
- BAMM-SC does not explicitly model zero-inflation, which may or may not affect clustering accuracy
- Cell Ranger's aggregation downsampling equalizes sequencing depth across libraries and leads to substantial information loss, so naive aggregation undermines this preprocessing path
- Each cell is assumed to belong to one distinct cluster; posterior probabilities measure clustering uncertainty, not continuous states like cell cycle

## Evidence

- [discussion] Zero-gene filter rationale: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [discussion] Rare cell type caveat: "This may be problematic for rare cell type identification"
- [intro] Avoid Cell Ranger downsampling: "reads from different libraries are downsampled such that all libraries have the same sequencing depth, leading to substantial information loss"
- [readme] Required input format: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells."
- [readme] Genes shared, cells may differ: "The numbers of genes are the same across different individuals but the number of cells might differ. If X is a matrix, DIMMSC will be used"

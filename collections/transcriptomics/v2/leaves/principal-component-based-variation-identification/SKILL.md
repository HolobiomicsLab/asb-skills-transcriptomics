---
name: principal-component-based-variation-identification
description: Use when scRNA-Seq expression data (log-transformed, gene-centered) shows
  a suspected dominant systematic effect and you must test whether PCs are cell-cycle
  effects before removal, using annotated control vs cell-cycle gene sets.
when_to_use_negative:
- When the goal is to remove other latent confounders (e.g., hidden cell-type or batch
  effects) — ccRemover is designed for removing the cell-cycle effect exclusively
- When the cell-cycle effect influences fewer than ~8% of control genes, where the
  effect is not captured by the leading principal components and will be missed
- When no reliable annotated set of cell-cycle genes is available, since ccRemover
  relies on a known cell-cycle gene set
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: R package implementing PCA on control genes, projection onto cell-cycle genes,
    and bootstrap significance testing of Δj to identify and remove cell-cycle components
  repo: https://github.com/cran/ccRemover
- name: gene_indexer
  role: Maps gene names/symbols to cell-cycle vs control gene indices for a given
    species
- name: bootstrap_diff
  role: Computes bootstrap replicates of Δj by resampling rows with replacement from
    cell-cycle and control gene projections
- name: get_diff
  role: Calculates the bootstrap-based t-statistic used with the cutoff CΔ
- name: Biomart
  role: Source of gene lists annotated to the cell cycle used to partition genes into
    cell-cycle and control sets
- name: org.Mm.eg.db
  role: Mouse annotation database supplying cell-cycle gene annotations
- name: org.Hs.eg.db
  role: Human annotation database supplying cell-cycle gene annotations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:9647e654057669f15ce0594459429600d77e1226377a9b30e1cd4e5e5672ab49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - bootstrap_diff
  - ccRemover
  - gene_indexer
  - get_diff
  - org.Hs.eg.db
  - org.Mm.eg.db
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# principal-component-based-variation-identification

## Summary

Identifies whether leading principal components of a single-cell RNA-Seq data matrix capture the cell-cycle effect by projecting control genes onto cell-cycle genes and testing each component with a bootstrap-based t-statistic (Δj) against a cutoff CΔ = 3, as implemented in ccRemover.

## When to use

Use when scRNA-Seq expression data (log-transformed, gene-centered) shows a suspected dominant systematic effect and you must test whether leading PCs are cell-cycle effects before removal, using annotated control vs cell-cycle gene sets.

## When NOT to use

- When the goal is to remove other latent confounders (e.g., hidden cell-type or batch effects) — ccRemover is designed for removing the cell-cycle effect exclusively
- When the cell-cycle effect influences fewer than ~8% of control genes, where the effect is not captured by the leading principal components and will be missed
- When no reliable annotated set of cell-cycle genes is available, since ccRemover relies on a known cell-cycle gene set

## Inputs

- scRNA-Seq expression data matrix X (cells x genes), log-transformed and gene-by-gene centered
- List or index of annotated cell-cycle genes (e.g., from Cyclebase, Biomart annotations via org.Mm.eg.db / org.Hs.eg.db)
- Control-gene indices (all genes not annotated as cell-cycle genes)
- Algorithm parameters: cutoff CΔ (3, or 2 for low cell-cycle activity), nboot (100–200), max_it (4), ntop (10)

## Outputs

- Set of principal components judged significant cell-cycle effects (Δj > CΔ)
- Cell-cycle-corrected expression matrix with significant components removed
- Per-iteration detection log of which components were identified and removed

## How to apply

First log-transform the expression matrix and center it gene-by-gene, then split genes into control genes (all genes not annotated to the cell cycle) and cell-cycle genes retrieved from annotation databases (e.g., via gene_indexer with org.Mm.eg.db / org.Hs.eg.db). Perform a PCA on the control-gene matrix X_B, project the data onto the cell-cycle genes, and for each leading component j compute Δj — the difference in captured variation between cell-cycle and control gene projections — assessed by bootstrapping (sampling |A| rows from X_A and |B| rows from X_B with replacement, nboot = 100–200 replicates). Declare component j a cell-cycle effect when Δj exceeds the cutoff CΔ (CΔ = 3 as default; CΔ = 2 for low cell-cycle-activity datasets like glioblastoma), and iterate (max_it = 4), removing significant components and re-testing until no further cell-cycle effects are detected. The rationale is that a genuine cell-cycle effect should explain disproportionately more variation among cell-cycle genes than among control genes, which guards against mistaking other factors (e.g., the cell-type effect scLVM removed by mistake in the six-group simulation) for the cell cycle.

## Related tools

- **ccRemover** (R package implementing PCA on control genes, projection onto cell-cycle genes, and bootstrap significance testing of Δj to identify and remove cell-cycle components) — https://github.com/cran/ccRemover
- **gene_indexer** (Maps gene names/symbols to cell-cycle vs control gene indices for a given species)
- **bootstrap_diff** (Computes bootstrap replicates of Δj by resampling rows with replacement from cell-cycle and control gene projections)
- **get_diff** (Calculates the bootstrap-based t-statistic used with the cutoff CΔ)
- **Biomart** (Source of gene lists annotated to the cell cycle used to partition genes into cell-cycle and control sets)
- **org.Mm.eg.db** (Mouse annotation database supplying cell-cycle gene annotations)
- **org.Hs.eg.db** (Human annotation database supplying cell-cycle gene annotations)

## Examples

```
gene_cycle_gene_indices <- gene_indexer(gene_names, species = "mouse", name_type = "symbols"); xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- The identified components match known biology: on the simulated data ccRemover correctly identified the 2nd and 3rd principal components as cell-cycle effects while preserving the cell-type structure (cells remained clustered by cell type with cell-cycle stages mixed)
- On the T helper dataset, the first principal component is flagged on the first iteration and, once removed, no further components exceed CΔ
- Bootstrap test behaves predictably with sparsity: the effect is reliably detected only once at least ~8% of control genes are influenced by the cell-cycle effect
- Corrected data preserves other biological signals — downstream clustering should separate cell types (e.g., TH2 signature odds ratio behavior) rather than degrade to noise
- Parameter choices follow the article: cutoff = 3 with nboot = 200, max_it = 4, ntop = 10 in the methods example; cutoff lowered to 2 only for low cell-cycle-activity data (glioblastoma)

## Limitations

- If the cell-cycle effect only influences a few of the control genes (below ~8%), it may not be captured by the principal components and will be missed for detection
- Annotation databases are always incomplete and inaccurate, so some cell-cycle genes may be misclassified as control genes
- ccRemover does not remove other confounding effects (e.g., hidden subtypes or batch effects) that may dominate after cell-cycle removal
- Cutoff choice depends on expected cell-cycle activity: the default CΔ = 3 is too strict for very low cell-cycle-activity datasets such as glioblastoma, where CΔ = 2 is used

## Evidence

- [methods] PCA on control genes with bootstrap Δj testing: "The ccRemover algorithm follows these steps: 1. Perform a PCA on the data matrix of control genes XB."
- [methods] Cutoff parameter in example invocation: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [methods] Bootstrap replicate computation: [evidence span withheld — anchor status: not_found]
- [results] Correct identification of cell-cycle PCs on simulated data: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [results] Detection on T helper data stops after one component: "When ccRemover is applied to the original data it identifies the first principal component to be a cell-cycle effect on the first iteration."
- [discussion] Sparsity limitation on control-gene influence: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
- [methods] Lower cutoff for low cell-cycle activity datasets: "except for the glioblastoma data, where it is known that the cell-cycle activity is at a very low level47. We use a smaller cutoff value CΔ = 2"

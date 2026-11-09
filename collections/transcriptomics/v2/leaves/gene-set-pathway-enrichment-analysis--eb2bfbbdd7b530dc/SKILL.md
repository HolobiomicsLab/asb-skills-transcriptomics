---
name: gene-set-pathway-enrichment-analysis
description: Use when you have JOINTLY factor/module gene sets (liver$modules) or
  factor-associated genes and need to know which biological processes or pathways
  each interpretable factor represents.
when_to_use_negative:
- When the goal is only to score integration/clustering performance — ARI/NMI metrics
  do not provide insight into the genes or features driving integration and clustering
- When no interpretable factor modules or gene sets are available (e.g. inputs are
  only expression matrices without a jointly() run), as there are no factor gene sets
  to test
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: JOINTLY
  role: Produces the learned interpretable factor modules whose gene sets are tested
    for pathway enrichment
  repo: https://github.com/madsen-lab/rJOINTLY
- name: UCell
  role: Computes rank-based per-cell module scores via AddModuleScore_UCell on liver$modules
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc gene-set/marker testing grouped by cell type on the RNA assay
  repo: https://github.com/immunogenomics/presto
- name: Seurat
  role: Hosts the JOINTLY reduction and UMAP used to visualise module scores
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:34af5c3829e9f94b38094dd91bac48a843bdf074768a35005c4d31ca8eae3633
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - presto
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# gene-set-pathway-enrichment-analysis

## Summary

Interpret JOINTLY's learned factor gene modules by computing per-cell module scores (UCell) and running gene-set/pathway enrichment on the factors, optionally tracked across ordered conditions such as scVelo latent-time bins.

## When to use

Use when you have JOINTLY factor/module gene sets (liver$modules) or factor-associated genes and need to know which biological processes or pathways each interpretable factor represents.

## When NOT to use

- When the goal is only to score integration/clustering performance — ARI/NMI metrics do not provide insight into the genes or features driving integration and clustering
- When no interpretable factor modules or gene sets are available (e.g. inputs are only expression matrices without a jointly() run), as there are no factor gene sets to test

## Inputs

- Seurat or SingleCellExperiment object after jointly() (returned list with $object and $modules)
- Gene-set lists from JOINTLY factors (liver$modules)
- Cell-level ordering/bins along scVelo latent time (erythropoiesis workflow: 9815 cells, three batches)
- Cell type labels (e.g. Human_Liver_labels.tsv) for group_by testing

## Outputs

- Per-cell UCell module scores added to the object (e.g. factor_1_UCell) for FeaturePlot on the JOINTLY UMAP
- Factor temporal profiles (mean activity per latent-time bin, Fig. 4I-K)
- Pathway/gene-set enrichment tables per factor or per latent-time bin (exported as CSV)
- wilcoxauc marker/enrichment results per cell type

## How to apply

After running jointly(liver, batch.var = "batch_label"), the returned list contains learned modules (liver$modules); compute rank-based per-cell module activity with UCell via AddModuleScore_UCell(features = liver$modules) and visualise scores (e.g. factor_1_UCell) on the JOINTLY UMAP (RunUMAP on the 'JOINTLY' reduction, dims 1:15). For marker-style validation, test factor/cell-type associations with presto's wilcoxauc(group_by = "celltype", seurat_assay = "RNA"). In temporal analyses (e.g. mouse gastrulation erythropoiesis vs scVelo latent time), bin cells along latent time, compute mean factor activity per bin to build temporal profiles (Fig. 4I-K), and run pathway/gene-set enrichment on each factor's genes per bin, exporting profiles and enrichment tables as CSV. Because JOINTLY solutions vary with the seed, rerun up to 5 times and keep the best solution before interpreting enrichment; the rationale is that the paper shows factor-associated genes are more specific for cell types than traditional markers and can guide cell type annotation and discovery of active biological processes.

## Related tools

- **JOINTLY** (Produces the learned interpretable factor modules whose gene sets are tested for pathway enrichment) — https://github.com/madsen-lab/rJOINTLY
- **UCell** (Computes rank-based per-cell module scores via AddModuleScore_UCell on liver$modules) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc gene-set/marker testing grouped by cell type on the RNA assay) — https://github.com/immunogenomics/presto
- **Seurat** (Hosts the JOINTLY reduction and UMAP used to visualise module scores) — https://github.com/satijalab/seurat

## Examples

```
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)
```

## Evaluation signals

- AddModuleScore_UCell adds score columns (e.g. factor_1_UCell) that render as coherent regions on the JOINTLY UMAP (RunUMAP, reduction = "JOINTLY", dims 1:15)
- On the Human_Liver test set, the upstream JOINTLY clustering used for interpretation should reach approximately ARI 0.7415981 and NMI 0.6325386 before enrichment interpretation
- JOINTLY stability check: solutions differ across seeds, so rerun up to 5 times and keep the best solution before building enrichment profiles
- In the erythropoiesis application, pathway enrichment across early/early-mid/late latent-time bins should reveal signalling cascades with distinct timing consistent with erythropoiesis literature (Fig. 4I-K)
- wilcoxauc results (res.raw) grouped by celltype should agree with factor module composition for the same cell types

## Limitations

- JOINTLY is stochastic: setting a seed and re-running produces a different result, so enrichment is only as stable as the chosen run (retry up to 5 times)
- Consensus PCA (cPCA) in JOINTLY is not advised for complex multi-condition, multi-sample datasets; regular PCA is recommended, which affects the factor gene sets fed into enrichment
- Software version numbers are not stated in the methods, so module scores and enrichment results may not be exactly reproducible

## Evidence

- [methods] module scores via UCell: "## Calculate module scores
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [intro] factor genes as interpretable markers: "the genes associated with the interpretable factors are more specific for cell types than traditional marker genes and that they can be used to guide cell type annotation and discover active"
- [methods] wilcoxauc testing: "res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA")"
- [readme] run-to-run variability: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."

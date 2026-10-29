---
name: gene-module-interpretation-for-cell-type-annotation
description: Use when you have run JOINTLY and need to interpret its learned factors
  as gene modules — e.g., to annotate cell types, confirm marker genes (DCN for IMR90,
  CD3B for Jurkat), or discover active biological processes per cluster.
when_to_use_negative:
- You only need integration/clustering performance (ARI/NMI) and have no goal of annotating
  clusters or interpreting factor genes
- The genes or features driving integration and clustering are already known and validated,
  so no module interpretation is needed
- Input is a raw unintegrated clustering result with no JOINTLY factor output — there
  are no interpretable modules to extract
tools:
- name: JOINTLY
  role: Learns the interpretable factors and returns the gene modules (liver$modules)
    used for cell type annotation
  repo: https://github.com/madsen-lab/rJOINTLY
- name: UCell
  role: Scores the learned gene modules per cell (AddModuleScore_UCell) for visualization
    on the UMAP
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Hosts the JOINTLY reduction and provides FeaturePlot/DimPlot to visualize
    module scores by cell type
  repo: https://github.com/satijalab/seurat
- name: presto
  role: wilcoxauc ranks raw expression genes per cell type to compare module genes
    against traditional markers
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:94503796f9be7940015358157a81a94393977713d8d074197afbe5c94534ef9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - presto
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-module-interpretation-for-cell-type-annotation

## Summary

Interprets the gene modules learned by JOINTLY's joint NMF factors to assign biological meaning to clusters and guide cell type annotation, by extracting top genes per factor and scoring them per cell with UCell module scores.

## When to use

Use when you have run JOINTLY and need to interpret its learned factors as gene modules — e.g., to annotate cell types, confirm marker genes (DCN for IMR90, CD3B for Jurkat), or discover active biological processes per cluster.

## When NOT to use

- You only need integration/clustering performance (ARI/NMI) and have no goal of annotating clusters or interpreting factor genes
- The genes or features driving integration and clustering are already known and validated, so no module interpretation is needed
- Input is a raw unintegrated clustering result with no JOINTLY factor output — there are no interpretable modules to extract

## Inputs

- Output list from jointly(): Seurat object with JOINTLY reduction plus a modules list (gene sets per factor)
- H matrix / factor gene loadings from JOINTLYsolve (advanced usage)
- Raw count expression (Seurat/SingleCellExperiment/matrix objects) for module scoring and wilcoxauc comparison
- Known cell type labels or marker gene expectations (e.g., Human_Liver_labels.tsv)

## Outputs

- Gene modules (top genes per JOINTLY factor) mapping factors to cell type marker programs
- Per-cell UCell module scores (e.g., factor_1_UCell columns) added to the Seurat object
- FeaturePlot visualizations of module scores on the JOINTLY UMAP
- wilcoxauc gene rankings per cell type for comparing module genes vs. traditional markers

## How to apply

After running jointly(), use the second list entry (the modules) returned by the wrapper — one gene set per interpretable factor. Extract the top genes per JOINTLY factor to recover the interpretable modules, then compute per-cell module scores with AddModuleScore_UCell(features = liver$modules) and visualize them on the UMAP via FeaturePlot (e.g., factor_1_UCell). To verify the modules are more cell-type specific than traditional markers, run presto's wilcoxauc on the raw expression grouped by celltype and compare its top genes against the module genes. In the eight cell-line benchmark (Fig. 1C), each dominant module should map to a single cell line's known marker genes (e.g., DCN in IMR90, CD3B in Jurkat).

## Related tools

- **JOINTLY** (Learns the interpretable factors and returns the gene modules (liver$modules) used for cell type annotation) — https://github.com/madsen-lab/rJOINTLY
- **UCell** (Scores the learned gene modules per cell (AddModuleScore_UCell) for visualization on the UMAP) — https://github.com/carmonalab/UCell
- **Seurat** (Hosts the JOINTLY reduction and provides FeaturePlot/DimPlot to visualize module scores by cell type) — https://github.com/satijalab/seurat
- **presto** (wilcoxauc ranks raw expression genes per cell type to compare module genes against traditional markers) — https://github.com/immunogenomics/presto

## Examples

```
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules); FeaturePlot(liver$object, features = "factor_1_UCell")
```

## Evaluation signals

- Each dominant JOINTLY module maps to a single cell line's marker genes (e.g., DCN for IMR90, CD3B for Jurkat), reproducing Fig. 1C
- Module genes from the jointly() output are present in liver$modules as one gene set per factor (factor_1, factor_2, ...)
- FeaturePlot of a module score (e.g., factor_1_UCell) shows enrichment restricted to the matching cell type/cluster on the UMAP
- wilcoxauc results on raw expression (res.raw) confirm module genes are at least as cell-type specific as traditional marker genes

## Limitations

- Integration metrics alone do not reveal which genes drive clustering, so interpretation requires this separate module-extraction/scoring step
- JOINTLY results vary with seed; the README advises re-running up to 5 times and choosing the best solution, so module composition can shift between runs
- Batch-corrected/smoothed expression from the H and F matrices should be used with caution and only for visualization, not for marker discovery on raw values

## Evidence

- [readme] Module scores computed with UCell on learned modules: "## Calculate module scores
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [intro] Factor genes guide cell type annotation (article intro): "the genes associated with the interpretable factors are more specific for cell types than traditional marker genes and that they can be used to guide cell type annotation"
- [other] Eight cell-line validation of modules vs markers: [evidence span withheld — anchor status: not_found]
- [discussion] Integration metrics lack interpretability (discussion): "such metrics do not provide insight into the genes or features driving integration and clustering."

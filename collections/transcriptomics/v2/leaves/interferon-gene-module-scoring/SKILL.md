---
name: interferon-gene-module-scoring
description: 'Use when validating that an integration method (e.g. JOINTLY, Harmony,
  scVI) resists over-correction: score a biologically meaningful module, such as interferon
  signalling, on cells from different conditions and check whether condition-specific
  signal remains separable after integration.'
when_to_use_negative:
- When the goal is only to quantify batch removal (iLISI/cLISI, ARI/NMI) rather than
  retention of condition-specific biology — module scoring addresses over-correction,
  not mixing
- When the integrated object lacks a biologically validated gene set (e.g. the interferon-signalling
  module) or cell type labels, since score interpretation depends on known across-condition
  biology
tools:
- name: UCell
  role: Computes per-cell module scores via AddModuleScore_UCell with the interferon-signalling
    gene set as features
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Hosts the integrated embeddings and UMAPs; DimPlot/FeaturePlot visualize module
    scores per method, cluster, cell type and batch
  repo: https://github.com/satijalab/seurat
- name: presto (wilcoxauc)
  role: Wilcoxon rank-sum test identifying the 3529 DE genes between pancreas- and
    lung-derived endothelial cells, used as an orthogonal retention signal
  repo: https://github.com/immunogenomics/presto
- name: JOINTLY
  role: Primary integration method under test, run in regular PCA mode with batch.var
    = 'batch_label' before module scoring
  repo: https://github.com/madsen-lab/rJOINTLY
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:96fe1288ebfe5af374604b9d92fbb977558dc2826a2411beb8211174393c3c09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - Harmony
  - JOINTLY
  - LIGER
  - Scanorama
  - Seurat
  - UCell
  - fastMNN
  - presto
  - presto (wilcoxauc)
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# interferon-gene-module-scoring

## Summary

Scores an interferon-signalling gene module on individual cells (notably endothelial cells in a merged pancreas-lung mixture) with AddModuleScore_UCell to test whether a batch-integration method retains across-tissue biological signal while removing batch effects.

## When to use

Use when validating that an integration method (e.g. JOINTLY, Harmony, scVI) resists over-correction: score a biologically meaningful module, such as interferon signalling, on cells from different conditions and check whether condition-specific signal remains separable after integration.

## When NOT to use

- When the goal is only to quantify batch removal (iLISI/cLISI, ARI/NMI) rather than retention of condition-specific biology — module scoring addresses over-correction, not mixing
- When the integrated object lacks a biologically validated gene set (e.g. the interferon-signalling module) or cell type labels, since score interpretation depends on known across-condition biology

## Inputs

- Seurat object with integrated embeddings from each method (JOINTLY, fastMNN, Harmony, scVI, LIGER, Scanorama) and a batch_label condition variable
- Gene set for the interferon-signalling module (list of gene identifiers, e.g. a named list compatible with UCell features)
- Cell type labels identifying endothelial cells across pancreas and lung tissues

## Outputs

- Per-cell interferon-signalling module scores (e.g. a '<module>_UCell' metadata column per integrated object)
- UMAP FeaturePlots of module scores per integration method, colored by module score, cell type and batch
- Qualitative/quantitative verdict on over-correction: whether pancreas vs lung endothelial interferon signal remains separable per method

## How to apply

After running each integration method (JOINTLY in regular PCA mode with batch.var = 'batch_label', plus fastMNN, Harmony, scVI, LIGER, Scanorama) on the merged pancreas-lung Seurat object, attach each method's embedding as a DimReduc and run UMAP on dims 1:15. Compute per-cell module scores for the interferon-signalling gene set on the endothelial cells using AddModuleScore_UCell, passing the gene sets as the features argument. Compare module scores against the 3529 Wilcoxon (wilcoxauc) differentially expressed genes between pancreas- and lung-derived endothelial cells: a method that over-corrects erases the across-tissue difference in interferon module scores. Visualize module scores on per-method UMAPs alongside cluster, cell-type and batch coloring (FeaturePlot/DimPlot) to reproduce the Fig. 3A-F style comparison. The rationale is that metrics like iLISI/cLISI quantify batch removal but not biological retention, so module scoring directly probes the over-correction failure mode.

## Related tools

- **UCell** (Computes per-cell module scores via AddModuleScore_UCell with the interferon-signalling gene set as features) — https://github.com/carmonalab/UCell
- **Seurat** (Hosts the integrated embeddings and UMAPs; DimPlot/FeaturePlot visualize module scores per method, cluster, cell type and batch) — https://github.com/satijalab/seurat
- **presto (wilcoxauc)** (Wilcoxon rank-sum test identifying the 3529 DE genes between pancreas- and lung-derived endothelial cells, used as an orthogonal retention signal) — https://github.com/immunogenomics/presto
- **JOINTLY** (Primary integration method under test, run in regular PCA mode with batch.var = 'batch_label' before module scoring) — https://github.com/madsen-lab/rJOINTLY

## Examples

```
liver$object <- AddModuleScore_UCell(liver$object, features = interferon_module); FeaturePlot(liver$object, features = "interferon_UCell", reduction = "umap")
```

## Evaluation signals

- Interferon-signalling module scores on endothelial cells remain visibly separable between pancreas- and lung-derived cells on the JOINTLY UMAP, indicating retained across-tissue signal
- Per-cell module score columns (e.g. '<module>_UCell') are present for every compared method's object and computed on the same gene set
- Module-score separation patterns are consistent with the 3529 DE genes found by wilcoxauc between pancreas and lung endothelial cells
- Within-tissue batch mixing still improves (iLISI/cLISI) even as the across-tissue module difference persists — both criteria must hold to accept the result
- Per-method UMAP panels colored by module score, cell type and batch (Fig. 3A-F style) are produced for all six methods for a like-for-like comparison

## Limitations

- Integration metrics such as ARI/NMI do not reveal the genes or features driving integration, which is why module scoring is needed, but module choice itself is analysis-specific
- Setting a seed and re-running JOINTLY produces a different result; the README notes it may be worth running JOINTLY up to 5 times and choosing the best solution, which affects the resulting module-score visualizations
- The R and Python versions of JOINTLY, although very similar, do not produce identical results; manuscript analyses used the R version
- Consensus PCA in JOINTLY is not advised for multi-condition, multi-sample datasets — regular PCA should be used before scoring modules

## Evidence

- [methods] Module scoring method: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [intro] Over-correction evaluation rationale: "We evaluate the robustness against over-correction by integrating multi-donor datasets from different tissues and find that JOINTLY removes within-tissue batch effects but retains across-tissue"
- [discussion] Methods retaining biological variation: [evidence span withheld — anchor status: not_found]
- [readme] Module score visualization workflow: "## Calculate module scores
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)

## Visualize module scores on the UMAP
FeaturePlot(liver$object, features = "factor_1_UCell")"

---
name: anchor-based-integration-execution
description: Use when integrating multi-sample or multi-study scRNA-seq data in Seurat
  where batch correction must preserve cell type structure, especially when partial
  cell type labels (e.g., scGate CD8_TIL gates) are available to guide anchor finding.
when_to_use_negative:
- Input is not raw per-sample scRNA-seq count data (e.g., an already integrated embedding
  or non-transcriptomic data) — anchors are computed between scRNA-seq datasets.
- You only need to score an existing integration rather than perform one — use the
  scIntegrationMetrics/scib-pipeline evaluation tools instead.
- A single dataset with no batch structure is supplied; there are no inter-dataset
  anchors to compute.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: STACAS
  role: 'Provides the anchor-based integration machinery: FindAnchors.STACAS, FilterAnchors.STACAS,
    SampleTree.STACAS, FindVariableFeatures.STACAS and IntegrateData.STACAS with semisupervised
    option.'
  repo: https://github.com/carmonalab/STACAS
- name: Seurat v4
  role: Integration framework in which STACAS anchors are computed and the integrated
    assay/embedding is produced.
- name: scGate
  role: Supplies partial prior cell type labels (e.g., CD8_TIL gating models from
    carmonalab/scGate_models) used to guide semisupervised anchor integration.
  repo: https://github.com/carmonalab/scGate_models
- name: Harmony
  role: Unsupervised integration comparator; expected to show high batch mixing (CiLISI)
    but low celltype ASW relative to semisupervised STACAS.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:fef60ce3a9495ddfcbfdd701c4e56779d97df61be51eb0940d2e6f9ab8eb97ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - Harmony
  - STACAS
  - Seurat v4
  - scGate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# anchor-based-integration-execution

## Summary

Executes anchor-based batch correction of scRNA-seq collections using STACAS within Seurat v4, computing inter-dataset anchors, filtering them by pairwise rPCA distance, and integrating data with an optional semisupervised mode that uses prior cell type labels to preserve biological variance.

## When to use

Use when integrating multi-sample or multi-study scRNA-seq data in Seurat where batch correction must preserve cell type structure, especially when partial cell type labels (e.g., scGate CD8_TIL gates) are available to guide anchor finding.

## When NOT to use

- Input is not raw per-sample scRNA-seq count data (e.g., an already integrated embedding or non-transcriptomic data) — anchors are computed between scRNA-seq datasets.
- You only need to score an existing integration rather than perform one — use the scIntegrationMetrics/scib-pipeline evaluation tools instead.
- A single dataset with no batch structure is supplied; there are no inter-dataset anchors to compute.

## Inputs

- Per-sample scRNA-seq Seurat objects (e.g., the 20 seed CD8 TIL samples, 11,021 cells across 7 tumor types)
- Partial cell type labels from scGate gating models (e.g., CD8_TIL) used as semisupervised priors
- Gene reference tables: EnsemblGeneTable.Hs / EnsemblGeneTable.Mm and genes.blocklist for symbol standardization and feature filtering
- Integration parameters (nfeat for FindVariableFeatures.STACAS; dims, k.weight, new.assay.name for IntegrateData.STACAS)

## Outputs

- Anchor set between datasets (FindAnchors.STACAS), filtered by rPCA distance (FilterAnchors.STACAS)
- Hierarchical sample tree for Seurat integration (SampleTree.STACAS)
- Integrated Seurat object with a new corrected assay (IntegrateData.STACAS, semisupervised=TRUE)
- Integrated embedding ready for celltype_ASW and CiLISI evaluation via scIntegrationMetrics

## How to apply

Load per-sample Seurat objects, standardize gene symbols (StandardizeGeneSymbols with EnsemblGeneTable) and select variable features with FindVariableFeatures.STACAS (respecting genesBlockList). Compute inter-dataset anchors with FindAnchors.STACAS, then apply FilterAnchors.STACAS to discard anchors failing the pairwise rPCA distance criterion; optionally inspect anchor distance distributions with PlotAnchors.STACAS. Use SampleTree.STACAS to determine a hierarchical tree for Seurat integration, or pass sample.tree to IntegrateData.STACAS. Run IntegrateData.STACAS with semisupervised=TRUE, supplying partial prior labels (e.g., scGate CD8_TIL annotations) so anchors are biased toward matching cell types; in the CD8 TIL reference workflow this is done in two rounds, refining labels after round 1 via annotate.by.neighbors and re-running with 800 highly variable genes and dims=50. Validate the integrated embedding by computing celltype_ASW and CiLISI with carmonalab/scIntegrationMetrics, expecting improved celltype ASW over unsupervised STACAS without loss of batch mixing.

## Related tools

- **STACAS** (Provides the anchor-based integration machinery: FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, FindVariableFeatures.STACAS and IntegrateData.STACAS with semisupervised option.) — https://github.com/carmonalab/STACAS
- **Seurat v4** (Integration framework in which STACAS anchors are computed and the integrated assay/embedding is produced.)
- **scGate** (Supplies partial prior cell type labels (e.g., CD8_TIL gating models from carmonalab/scGate_models) used to guide semisupervised anchor integration.) — https://github.com/carmonalab/scGate_models
- **Harmony** (Unsupervised integration comparator; expected to show high batch mixing (CiLISI) but low celltype ASW relative to semisupervised STACAS.)

## Examples

```
anchors <- FindAnchors.STACAS(obj.list); anchors <- FilterAnchors.STACAS(anchors); integrated <- IntegrateData.STACAS(anchors, new.assay.name="STACAS.integrated", dims=50, semisupervised=TRUE)
```

## Evaluation signals

- Anchor distance distributions between dataset pairs are inspected with PlotAnchors.STACAS and filtered anchors satisfy the pairwise rPCA distance criterion via FilterAnchors.STACAS.
- IntegrateData.STACAS returns a Seurat object with a new assay (new.assay.name) integrating all samples along the sample.tree.
- On the 20-seed CD8 map, semisupervised STACAS improves both celltype_ASW and CiLISI over uncorrected data.
- Round-2 ssSTACAS (800 HVGs, dims=50) achieves higher celltype_ASW than unsupervised STACAS without degrading batch mixing (CiLISI).
- Harmony control shows the expected pattern of high batch mixing but low celltype ASW, confirming the metric computation is behaving correctly.

## Limitations

- CiLISI and similar metrics require cell type annotations, cannot be calculated for cell types present in only one dataset, and are suboptimal when datasets have highly unequal numbers of cells.
- Batch mixing metrics robust to lopsided cell proportions are still lacking, complicating evaluation of integrations with unbalanced sample sizes.
- STACAS is robust to incomplete and imprecise labels, but benchmarks including supervised/semisupervised methods should still account for noisy, partial cell-type information.

## Evidence

- [abstract] we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration.: "we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
- [methods] FindAnchors.STACAS computes anchors between datasets for integration in Seurat: "`FindAnchors.STACAS`  Computes anchors between datasets for integration in Seurat"
- [methods] FilterAnchors.STACAS filters integration anchors based on pairwise rPCA distance: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [methods] SampleTree.STACAS determines a hierarchical tree for Seurat integration using STACAS anchors: "`SampleTree.STACAS`   Determine automatically a hierarchical tree for Seurat integration using anchors calculated in STACAS"
- [readme] Prior cell type labels enable semi-supervised integration with increased preservation of biological variability: "Prior cell type knowledge, given as cell type labels, can be provided to the algorithm to perform semi-supervised integration"

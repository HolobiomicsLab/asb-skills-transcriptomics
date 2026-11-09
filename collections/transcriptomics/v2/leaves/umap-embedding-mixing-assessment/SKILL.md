---
name: umap-embedding-mixing-assessment
description: Use when you have cell-by-gene expression matrices (e.g. expr_mat.csv)
  segmented independently from two replicate spatial transcriptomics sections and
  need to quantify whether segmentation output is replicable across replicates.
when_to_use_negative:
- Comparing non-replicate conditions (different tissues, platforms, or biological
  states) where poor embedding mixing is expected and not evidence of segmentation
  failure
- Only one section is available — mixing cannot be assessed without a second replicate
- Input lacks a per-cell expression matrix (e.g. only the raw transcripts.csv.gz has
  not yet been segmented via BIDCell)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BIDCell
  role: Produces the replicate-specific segmented cell-gene expression matrices (expr_mat.csv)
    and _connected.tif masks that are embedded and compared
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform providing the replicate Xenium human breast cancer in situ
    output bundles (transcripts.csv.gz, morphology_mip.ome.tif)
- name: SydneyBioX/scClassify
  role: Source of example single-cell reference data used to annotate predicted cell
    types prior to agreement assessment
  repo: https://github.com/SydneyBioX/scClassify
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:ab68b6457b00b02347296173b75a74e08aae9d04ec5658ffb05f52954e89004f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - SydneyBioX/scClassify
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# umap-embedding-mixing-assessment

## Summary

Assesses cross-replicate replicability of BIDCell cell segmentation by embedding cells from two replicate sections (e.g. Xenium-BreastCancer1 and Xenium-BreastCancer2) into a low-dimensional UMAP space and checking whether replicate labels are well mixed. This is one of the CellSPA evaluation-framework metrics for segmentation quality.

## When to use

Use when you have cell-by-gene expression matrices (e.g. expr_mat.csv) segmented independently from two replicate spatial transcriptomics sections and need to quantify whether segmentation output is replicable across replicates.

## When NOT to use

- Comparing non-replicate conditions (different tissues, platforms, or biological states) where poor embedding mixing is expected and not evidence of segmentation failure
- Only one section is available — mixing cannot be assessed without a second replicate
- Input lacks a per-cell expression matrix (e.g. only the raw transcripts.csv.gz has not yet been segmented via BIDCell)

## Inputs

- expr_mat.csv cell-gene expression matrix for Xenium-BreastCancer1 segmented cells
- expr_mat.csv cell-gene expression matrix for Xenium-BreastCancer2 segmented cells
- Breast cancer single-cell reference and marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv)
- Replicate labels per cell

## Outputs

- UMAP embedding of cells from both replicates coloured by replicate label
- Qualitative/quantitative assessment of replicate mixing as part of CellSPA agreement metrics

## How to apply

Run the BIDCell pipeline (segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat with is_cell=False, preannotate, train UNet3+ for 4,000 steps, predict) on each replicate to extract expr_mat.csv per replicate. Optionally annotate predicted cell types in both replicates using the breast cancer single-cell reference and marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv). Embed the combined cells with UMAP (as in the CellSPA evaluation framework categories of Fig. 3i-j and Supplementary Fig. 16) and colour/inspect cells by replicate label: a well-mixed population of cells between replicates indicates the segmentation is replicable, whereas replicate-specific clusters indicate discordance. Interpret the embedding alongside the other cross-replicate agreement metrics — correlations of baseline cell characteristics, cell type proportion comparison, and morphology metric correlations — rather than in isolation.

## Related tools

- **BIDCell** (Produces the replicate-specific segmented cell-gene expression matrices (expr_mat.csv) and _connected.tif masks that are embedded and compared) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform providing the replicate Xenium human breast cancer in situ output bundles (transcripts.csv.gz, morphology_mip.ome.tif))
- **SydneyBioX/scClassify** (Source of example single-cell reference data used to annotate predicted cell types prior to agreement assessment) — https://github.com/SydneyBioX/scClassify

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel("xenium_example_config.yaml"); model.preprocess(); model.train(); model.predict()
```

## Evaluation signals

- Cells from both replicates form a well-mixed population in the embedding, with no replicate-specific clusters, demonstrating replicability
- expr_mat.csv and the *_connected.tif mask exist for both replicates before running the comparison
- Embedding-based agreement pattern is consistent with the replicate-concordance patterns reported in Fig. 3i-j and Supplementary Fig. 16
- Cell type proportions and baseline cell characteristic correlations between replicates corroborate the embedding-level agreement
- Interpret embedding mixing alongside other CellSPA metric categories, not as the sole criterion

## Limitations

- Embedding mixing alone is not sufficient: achieving homogeneous cell bodies can also result from arbitrary over-segmentation of nuclei, motivating use of multiple CellSPA metrics
- Two-dimensional embedding may achieve limited representation of cells' three-dimensional structure
- If replicate-label clustering is observed, it may reflect data processing differences rather than true biological or segmentation failure; verify file paths and cpus settings in xenium_example_config.yaml were consistent across replicates

## Evidence

- [intro] CellSPA evaluation framework: "we created a comprehensive evaluation framework for cell segmentation, CellSPA, that assesses five complementary categories of criteria"
- [results] Replicate mixing finding: [evidence span withheld — anchor status: not_found]
- [readme] BIDCell reference data: "The breast cancer reference data are provided with this package under `data/sc_references`"
- [readme] Pipeline stages: "the pipeline can be broken down into 3 main stages"

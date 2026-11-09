---
name: cell-gene-matrix-extraction-from-segmentation-mask
description: Use when a BIDCell segmentation mask (pixel values = cell IDs, *_connected.tif)
  must be converted into a per-cell transcript count matrix (expr_mat.csv) from a
  transcripts file such as Xenium's transcripts.csv.gz.
when_to_use_negative:
- No segmentation mask exists yet — the matrix can only be built after model.preprocess(),
  model.train(), and model.predict() have produced the *_connected.tif mask.
- Transcript data must be assigned without an image-based mask (e.g. segmentation-free,
  transcript-based approaches such as Baysor), since this skill depends on pixel-level
  cell IDs.
- Nuclei images are separated into fields of view (e.g. CosMx) and have not yet been
  merged with model.stitch_nuclei(), so the mask does not cover the full tissue.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: BIDCell
  role: Provides run_pipeline() / make_cell_gene_mat(is_cell=False) to aggregate transcripts
    into the cell-gene matrix from the _connected.tif segmentation mask
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source of the quality-filtered transcripts.csv.gz and DAPI morphology_mip.ome.tif
    that are joined against the mask to quantify per-cell expression
  repo: https://www.10xgenomics.com/products/xenium-in-situ/preview-dataset-human-breast
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:f9714fdb325ec14a1187b868d69fbf26a98fedec3498c6d10af08f32a6235887
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - Cellpose
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-gene-matrix-extraction-from-segmentation-mask

## Summary

Builds a cell × gene expression matrix by assigning quality-filtered spatial transcripts (e.g. Xenium transcripts.csv.gz) to cells in a BIDCell segmentation mask (_connected.tif, whose pixel values are cell IDs), producing expr_mat.csv for downstream single-cell spatial analyses.

## When to use

Use when a BIDCell segmentation mask (pixel values = cell IDs, *_connected.tif) must be converted into a per-cell transcript count matrix (expr_mat.csv) from a transcripts file such as Xenium's transcripts.csv.gz.

## When NOT to use

- No segmentation mask exists yet — the matrix can only be built after model.preprocess(), model.train(), and model.predict() have produced the *_connected.tif mask.
- Transcript data must be assigned without an image-based mask (e.g. segmentation-free, transcript-based approaches such as Baysor), since this skill depends on pixel-level cell IDs.
- Nuclei images are separated into fields of view (e.g. CosMx) and have not yet been merged with model.stitch_nuclei(), so the mask does not cover the full tissue.

## Inputs

- Segmentation mask .tif from BIDCell inference, file name ending in _connected.tif, pixel values = cell IDs
- Quality-filtered transcripts file (transcripts.csv.gz) from the Xenium Output Bundle In Situ Replicate 1
- DAPI image (morphology_mip.ome.tif) used as the dimension reference for the mask

## Outputs

- expr_mat.csv cell × gene count matrix (one row per segmented cell)
- Number of segmented cells (unique cell IDs in the *_connected.tif mask; ~103,209 for Xenium-BreastCancer1)
- Fraction of transcripts assigned to cells (~53.4% on the Xenium breast cancer replicate)

## How to apply

After model.predict() and postprocessing yield the *_connected.tif segmentation mask, aggregate all quality-filtered transcripts from transcripts.csv.gz into their containing cell to produce the expr_mat.csv cell-gene matrix, the final stage of model.run_pipeline() ('data processing, training, prediction, and extracting the cell-gene matrix'). Use model.make_cell_gene_mat(is_cell=False) when operating on nuclei-based segmentation during preprocessing (is_cell=True for cell-body-level matrices), after model.generate_expression_maps() and model.generate_patches() have rasterized transcripts into expression maps. Ensure the mask .tif has the same dimensions as the DAPI image (morphology_mip.ome.tif) so cell IDs can be joined to transcript coordinates and the mask imported via xeniumranger import-segmentation. The matrix is limited to the genes needed for segmentation — the BIDCell README notes 'Only <1,000 genes are needed to perform segmentation' — so specify gene selections in a file for very high-plex platforms like Stereo-seq. Validate by counting unique cell IDs in the mask (target ~103,209 cells for Xenium-BreastCancer1 In Situ Replicate 1) and checking that ~53.4% of transcripts are assigned to cells.

## Related tools

- **BIDCell** (Provides run_pipeline() / make_cell_gene_mat(is_cell=False) to aggregate transcripts into the cell-gene matrix from the _connected.tif segmentation mask) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source of the quality-filtered transcripts.csv.gz and DAPI morphology_mip.ome.tif that are joined against the mask to quantify per-cell expression) — https://www.10xgenomics.com/products/xenium-in-situ/preview-dataset-human-breast

## Examples

```
from bidcell import BIDCellModel
model = BIDCellModel('xenium_example_config.yaml')
model.run_pipeline()
```

## Evaluation signals

- Count of unique cell IDs in the *_connected.tif mask matches the reported ~103,209 cells for Xenium-BreastCancer1 In Situ Replicate 1.
- Fraction of transcripts assigned to cells is ~53.4% on the Xenium-BreastCancer1 run.
- expr_mat.csv contains one row per segmented cell and columns for the segmentation gene panel (fewer than 1,000 genes are required).
- Mask .tif dimensions equal those of morphology_mip.ome.tif, so the segmentation can be imported with xeniumranger import-segmentation after resizing.
- expr_mat.csv cell IDs are consistent with the pixel values of the _connected.tif mask (no IDs in the matrix absent from the mask).

## Limitations

- Only ~53.4% of transcripts are assigned to cells on the Xenium breast cancer replicate; unassigned transcripts are excluded from the matrix.
- 2D segmentation may achieve limited representation of cells' 3D structure, and cell slicing may affect measured tissue expression.
- Resource constraints apply: reduce the cpus parameter if 'pickle.UnpicklingError: pickle data was truncated' occurs, and segmentation at higher resolution requires larger patches and more GPU memory.

## Evidence

- [methods] The .tif file needs to be resized to be the same dimensions as the DAPI image (`morphology_mip.ome.tif`): "The .tif file needs to be resized to be the same dimensions as the DAPI image (`morphology_mip.ome.tif`)"
- [readme] .tif file of segmented cells, where the value corresponds to cell IDs: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [readme] To run the entire pipeline (data processing, training, prediction, and extracting the cell-gene matrix): "To run the entire pipeline (data processing, training, prediction, and extracting the cell-gene matrix)"
- [readme] Only <1,000 genes are needed to perform segmentation. Specify a selection of genes in a file: "Only <1,000 genes are needed to perform segmentation. Specify a selection of genes in a file (see Stero-seq example)."

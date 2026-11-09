---
name: hippocampal-subregion-marker-expression-validation
description: Use when BIDCell segmenting a Xenium mouse brain hippocampus section
  needs biological validation, i.e. verifying that segmented cells resolve spatially
  distinct DG, CA1, CA2, and CA3 marker expression.
when_to_use_negative:
- Input spatial data is from a non-brain or non-hippocampal tissue where hippocampal
  subregion markers (Prox1, Necab2, Slit2, Wfs1, Cpne8) carry no validation signal
- Only nuclei segmentation is needed without a transcriptional check — this skill
  validates segmentation via marker expression, not morphological metrics alone
- The spatial platform lacks subcellular resolution or a compatible BIDCell .yaml
  config (Xenium, CosMx, MERSCOPE, Stereo-seq)
edam_operation: http://edamontology.org/operation_0337
edam_topics: []
tools:
- name: BIDCell
  role: Runs the self-supervised segmentation pipeline (preprocess/train/predict)
    on Xenium mouse brain data to produce the cell-gene matrix and connected-component
    segmentation
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform providing the subcellular spatial transcriptomics mouse brain
    dataset with transcripts and DAPI morphology image
- name: scClassify
  role: Hierarchical cell-type classifier whose pretrained mouse brain models annotate
    spatial cells into hippocampal subregions used to aggregate expression
  repo: https://github.com/SydneyBioX/scClassify
- name: Allen Brain Map
  role: Public repository supplying the SMART-seq single-cell reference used to build
    positive/negative marker files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:5e9c49637fa7a3e7774b660e07f25785b81e873ce9bf0a2bc1ede2f88fe80e46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - Allen Brain Map
  - BIDCell
  - scClassify
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# hippocampal-subregion-marker-expression-validation

## Summary

Validates BIDCell segmentation of densely packed pyramidal neurons on 10x Genomics Xenium mouse brain data by checking that hippocampal subregion marker genes (Prox1, Neurod6, Wfs1, Cpne8, Necab2, Slit2) are enriched in their expected subregions (DG, CA1, CA2, CA3), reproducing the Fig. 5f pattern.

## When to use

Use when BIDCell segmenting a Xenium mouse brain hippocampus section needs biological validation, i.e. verifying that segmented cells resolve spatially distinct DG, CA1, CA2, and CA3 marker expression.

## When NOT to use

- Input spatial data is from a non-brain or non-hippocampal tissue where hippocampal subregion markers (Prox1, Necab2, Slit2, Wfs1, Cpne8) carry no validation signal
- Only nuclei segmentation is needed without a transcriptional check — this skill validates segmentation via marker expression, not morphological metrics alone
- The spatial platform lacks subcellular resolution or a compatible BIDCell .yaml config (Xenium, CosMx, MERSCOPE, Stereo-seq)

## Inputs

- 10x Genomics Xenium mouse brain dataset (transcripts.csv.gz and morphology_mip.ome.tif DAPI image)
- Allen Brain Map SMART-seq single-cell reference dataset
- BIDCell Xenium .yaml configuration file
- Single-cell positive/negative marker CSV files (e.g. sc_breast_markers_pos.csv-style templates)
- scClassify pretrained mouse brain models (e.g. Tasic 2018/2016, Hrvatin .rds)

## Outputs

- expr_mat.csv cell-gene expression matrix
- *_connected.tif segmentation image with cell IDs as pixel values
- scClassify cell-type/subregion annotations (DG, CA1, CA2, CA3)
- Subregion-by-marker expression enrichment table comparable to Fig. 5f

## How to apply

Download the 10x Genomics Xenium mouse brain dataset and configure the BIDCell Xenium .yaml (BIDCellModel.get_example_config('xenium')) with correct paths to transcripts.csv.gz and morphology_mip.ome.tif. Build the single-cell reference from the Allen Brain Map SMART-seq dataset, generating positive/negative marker files (genes in the highest and lowest 10 percentile of expression per cell type, removing positive markers common to at least a third of cell types), and annotate spatial cells with scClassify pretrained models. Run the BIDCell pipeline (preprocess(), train() ~4000 steps, predict()) on a GPU with at least 12GB VRAM using the default UNet3+ architecture, yielding expr_mat.csv and a *_connected.tif segmentation. Aggregate expression in expr_mat.csv by scClassify-annotated hippocampal subregion (DG, CA1, CA2, CA3), then check subregion-specific enrichment of Prox1 (DG), Neurod6 (CA1-3), Wfs1 and Cpne8 (CA1), Necab2 (CA2), and Slit2 (CA3) against the expected Fig. 5f pattern. The rationale is that accurate segmentation of densely packed pyramidal neurons should spatially separate subregion-restricted expression, so missegmentation would blur marker boundaries between CA1/CA2/CA3 and DG.

## Related tools

- **BIDCell** (Runs the self-supervised segmentation pipeline (preprocess/train/predict) on Xenium mouse brain data to produce the cell-gene matrix and connected-component segmentation) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform providing the subcellular spatial transcriptomics mouse brain dataset with transcripts and DAPI morphology image)
- **scClassify** (Hierarchical cell-type classifier whose pretrained mouse brain models annotate spatial cells into hippocampal subregions used to aggregate expression) — https://github.com/SydneyBioX/scClassify
- **Allen Brain Map** (Public repository supplying the SMART-seq single-cell reference used to build positive/negative marker files)

## Examples

```
from bidcell import BIDCellModel; BIDCellModel.get_example_config('xenium'); model = BIDCellModel('xenium_example_config.yaml'); model.preprocess(); model.train(); model.predict()
```

## Evaluation signals

- Pipeline produces both expr_mat.csv and a *_connected.tif segmentation output
- Prox1 expression is restricted to DG (dentate gyrus) cells only
- Neurod6 is detected across all CA regions (CA1, CA2, CA3)
- Wfs1 and Cpne8 enrich specifically in CA1; Necab2 in CA2; Slit2 in CA3
- Overall subregion-by-marker pattern matches Fig. 5f of the BIDCell paper

## Limitations

- Training requires a GPU with at least 12GB VRAM (README reports a 12GB NVIDIA GTX Titan V) and ~32GB RAM; higher-resolution segmentation needs larger patches and more GPU memory
- Positive/negative marker curation affects results: using a larger number of positive markers tends to increase the size of predicted cells; removing positive markers common to at least a third of cell types is a heuristic
- Cells are analyzed in 2D, so analyses may achieve limited representation of their 3D structure, and dendritic/axon morphologies remain unsegmented — hippocampal neuropil-associated signal may be missed

## Evidence

- [results] finding: Prox1 only in DG, Neurod6 in all CA regions, Slit2 in CA3, Necab2 in CA2, and Wfs1 and Cpne8 in CA1: [evidence span withheld — anchor status: not_found]
- [readme] The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample.: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [readme] Note: A GPU with at least 12GB VRAM is strongly recommended: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
- [readme] Using a larger number of positive markers tends to increase the size of predicted cells.: "Using a larger number of positive markers tends to increase the size of predicted cells."
- [methods] These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas.: "These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas."

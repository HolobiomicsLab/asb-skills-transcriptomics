---
name: batch-effect-correction-with-harmony
description: Use when scRNA-seq data contain multiple batches (e.g., patients or samples,
  with a metadata CSV batch column such as 'PatientID') and the top PCs must be batch-corrected
  with Harmony before Louvain clustering in the scDrug/Scanpy pipeline.
when_to_use_negative:
- Input is a single homogeneous sample with no batch column defined — batch correction
  is applied only 'if needed' and would remove genuine variation
- The goal is drug response prediction (CaDRReS-Sc IC50 / PRISM AUC) or treatment
  selection on an already-clustered object — those stages take scanpyobj.h5ad and
  do not re-correct PCs
- Cells lack matched batch metadata, since scDrug requires a metadata CSV and a named
  batch column (e.g., 'PatientID') to run correction
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Harmony
  role: Adjusts the principal components to eliminate batch effects before clustering
- name: Scanpy
  role: Performs data preprocessing and PCA whose top PCs are fed to Harmony
- name: scDrug
  role: Wraps Scanpy preprocessing, Harmony correction, and Louvain clustering in
    single_cell_analysis.py with --metadata and --batch flags
  repo: https://github.com/ailabstw/scDrug
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:cd8c6a95905bea84831d6563708c5dea7cbad1457b22518c42228bc79406e4cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - Harmony
  - Python
  - Scanpy
  - scDrug
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# batch-effect-correction-with-harmony

## Summary

Corrects batch effects in scRNA-seq data by adjusting the top principal components with the Harmony algorithm within the scDrug pipeline, so that downstream Louvain clustering and sub-clustering (e.g., of the GSE156625 hepatocellular carcinoma dataset) are not driven by technical or patient-level batch structure.

## When to use

Use when scRNA-seq data contain multiple batches (e.g., patients or samples, with a metadata CSV batch column such as 'PatientID') and the top PCs must be batch-corrected with Harmony before Louvain clustering in the scDrug/Scanpy pipeline.

## When NOT to use

- Input is a single homogeneous sample with no batch column defined — batch correction is applied only 'if needed' and would remove genuine variation
- The goal is drug response prediction (CaDRReS-Sc IC50 / PRISM AUC) or treatment selection on an already-clustered object — those stages take scanpyobj.h5ad and do not re-correct PCs
- Cells lack matched batch metadata, since scDrug requires a metadata CSV and a named batch column (e.g., 'PatientID') to run correction

## Inputs

- scRNA-seq count matrix in 10x-Genomics-formatted mtx directory or CSV format
- Metadata CSV file for batch correction (row index of the input in the first column)
- Name of the batch column in the metadata or adata.obs (e.g., 'PatientID')
- Optionally, a previously produced Scanpy AnnData object (scanpyobj.h5ad) for sub-clustering

## Outputs

- Harmony-adjusted top principal components used for neighborhood graph construction and Louvain clustering
- Scanpy AnnData object scanpyobj.h5ad with cluster assignments and resolution scores
- UMAP plot showing batch effects (umap_batch.png)

## How to apply

In the scDrug Single-Cell Data Analysis pipeline, compute principal components in Scanpy after preprocessing (filtering cells with <200 genes, genes in <3 cells, mitochondrial proportion >=30%; normalization to 10,000 counts, log transform, HVG annotation, scaling), then adjust the top PCs with Harmony to eliminate batch effects before building the neighborhood graph from the top 20 PCs for Louvain clustering. Supply a metadata CSV (with the input index in its first column) and the name of the batch column (e.g., 'PatientID') via the -m/--metadata and -b/--batch flags; the corrected object additionally yields a umap_batch.png UMAP for inspecting residual batch structure. Harmony operates on the PCs, not the raw count matrix, and is applied only when batch correction is needed — e.g., before sub-clustering tumor subpopulations at the auto-determined resolution (0.6 in the GSE156625 case), so that the chooseR robustness scores reflect biology rather than batch.

## Related tools

- **Harmony** (Adjusts the principal components to eliminate batch effects before clustering)
- **Scanpy** (Performs data preprocessing and PCA whose top PCs are fed to Harmony)
- **scDrug** (Wraps Scanpy preprocessing, Harmony correction, and Louvain clustering in single_cell_analysis.py with --metadata and --batch flags) — https://github.com/ailabstw/scDrug

## Examples

```
python3 single_cell_analysis.py --input INPUT --metadata METADATA --batch BATCH --resolution 1.0 --annotation --gsea
```

## Evaluation signals

- A batch column is defined (via --batch) and the metadata CSV's first column matches the input cell index
- umap_batch.png is produced, allowing visual confirmation that cells from different batches intermix after correction
- Downstream Louvain clustering runs on the neighborhood graph built from the top 20 Harmony-adjusted PCs
- The final scanpyobj.h5ad loads as a valid AnnData object with a 'louvain' categorical observation field, and if --auto-resolution was used the chosen resolution lies within [0.4, 1.4]
- For GSE156625, reproducing the pipeline yields 15 Louvain cell groups and 14 tumor sub-clusters after Harmony correction

## Limitations

- Harmony is applied 'if needed' — it is optional in scDrug and inappropriate for single-batch data
- Requires metadata with a batch column; scDrug expects the metadata CSV index as the first column
- Docker-in-Docker may not operate on some machines, requiring the CIBERSORTx container to be run outside the scDrug container (relevant when combining correction with downstream treatment selection)
- The article limits scDrug evaluation; extensive experimental validations of the pipeline were described as still necessary

## Evidence

- [methods] Harmony adjusts PCs before clustering: "adjusted the principal components (PCs) with the Harmony algorithm to eliminate batch effects if needed"
- [readme] Metadata and batch flags in single_cell_analysis.py: "-m METADATA, --metadata METADATA
                        path to metadata CSV file for batch correction (index as input in first column)"
- [readme] Batch UMAP output: "provided additional UMAPs showing batch effects and cell types (`umap_batch.png` and `umap_cell_type.png`)"
- [methods] Preprocessing thresholds preceding PCA/Harmony: "we filtered out cells with less than 200 genes expressed and genes expressed in less than three cells and kept the cells with a proportion of mitochondrial genes below 30 %"

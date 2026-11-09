---
name: atac-seq-gene-activity-matrix-preprocessing
description: Use when integrating scATAC-seq data with annotated scRNA-seq data via
  scBridge (or similar transfer-learning pipelines) and the target is only a peak-by-cell
  matrix that must first become a gene activity matrix suitable for cross-omics label
  transfer.
when_to_use_negative:
- The source and target are already matched gene-space expression matrices preprocessed
  by the upstream study (e.g., the 4,644 CITE-seq / 4,506 ASAP-seq cells from the
  scJoint data.zip) and only need loading
- The goal is ATAC peak-level analysis (peak calling, LSI clustering) rather than
  cross-omics integration into gene activity space
- The target modality is protein abundance (CITE-seq/ASAP-seq protein data), which
  scBridge handles differently via Supplementary Note 1 procedures not included in
  the main text
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: Signac
  role: R package used to transform the scATAC-seq peak matrix into a scATAC-seq activity
    matrix as scBridge input
  repo: https://github.com/stuart-lab/signac
- name: scBridge
  role: Consumes the gene activity / TF-IDF preprocessed target h5ad together with
    the Standard-preprocessed source scRNA-seq h5ad for integration
  repo: https://github.com/XLearning-SCU/scBridge
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:7c4d191122dafa52c36426331d7f0308777627ff9ae4222a76e0d1149bc0ee96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stuart-lab/signac
  tools:
  - GLUE
  - PyTorch
  - Python
  - Signac
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# atac-seq-gene-activity-matrix-preprocessing

## Summary

Transform scATAC-seq peak-by-cell matrices into gene activity matrices (e.g., with Signac) and apply TF-IDF preprocessing with per-cell normalization to 10,000 counts so scATAC-seq target data can be integrated with scRNA-seq source data in scBridge.

## When to use

Use when integrating scATAC-seq data with annotated scRNA-seq data via scBridge (or similar transfer-learning pipelines) and the target is only a peak-by-cell matrix that must first become a gene activity matrix suitable for cross-omics label transfer.

## When NOT to use

- The source and target are already matched gene-space expression matrices preprocessed by the upstream study (e.g., the 4,644 CITE-seq / 4,506 ASAP-seq cells from the scJoint data.zip) and only need loading
- The goal is ATAC peak-level analysis (peak calling, LSI clustering) rather than cross-omics integration into gene activity space
- The target modality is protein abundance (CITE-seq/ASAP-seq protein data), which scBridge handles differently via Supplementary Note 1 procedures not included in the main text

## Inputs

- scATAC-seq peak-by-cell matrix (e.g., from GEO GSE111586 for the mouse atlas, or Zenodo records 6578553/6578617)
- Unnormalized scATAC-seq gene activity matrix and cell-type annotations (e.g., from atlas.gs.washington.edu/mouse-atac/)
- Annotated scRNA-seq source h5ad with obs['CellType'] (e.g., Tabula Muris FACS/droplet count matrices)
- Common-gene list shared between scRNA-seq and scATAC-seq modalities

## Outputs

- scATAC-seq gene activity matrix converted from the peak-by-cell matrix (Signac or equivalent)
- TF-IDF preprocessed target AnnData (h5ad) with per-cell counts scaled to 10,000
- Target AnnData ready for scBridge integration (common genes and shared cell types retained)

## How to apply

Raw scATAC-seq peak-by-cell matrices are not directly comparable to scRNA-seq gene expression, so first transform peaks into a scATAC-seq activity matrix using a tool such as Signac. Then preprocess the target with the TF-IDF strategy while the scRNA-seq source uses the Standard strategy, normalizing each cell by total counts scaled to 10,000 (divide each cell by its total read counts across all genes, then multiply by 10,000). Restrict analysis to the common genes between modalities (e.g., the 12,689 common genes of the mouse atlas analysis) and keep only shared cell types. Feed the resulting h5ad target file into scBridge with reliability_threshold=0.95, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, and early stopping when annotated-data classification accuracy reaches 0.99. Deeper discussion of TF-IDF choices is deferred to Supplementary Note 3, so treat the Standard/TFIDF split as the paper's validated default.

## Related tools

- **Signac** (R package used to transform the scATAC-seq peak matrix into a scATAC-seq activity matrix as scBridge input) — https://github.com/stuart-lab/signac
- **scBridge** (Consumes the gene activity / TF-IDF preprocessed target h5ad together with the Standard-preprocessed source scRNA-seq h5ad for integration) — https://github.com/XLearning-SCU/scBridge

## Examples

```
python main.py --data_path="MouseAtlas/" --source_data="RNA.h5ad" --target_data="ATAC.h5ad" --source_preprocess="Standard" --target_preprocess="TFIDF" --reliability_threshold=0.95
```

## Evaluation signals

- The target preprocessing flag is TFIDF while the source uses Standard (--source_preprocess="Standard", --target_preprocess="TFIDF")
- Every target cell's counts sum to 10,000 after normalization (divide by total counts, multiply by 10,000)
- Gene space of target and source matrices matches exactly on the common gene set (e.g., 12,689 genes for the mouse atlas)
- Downstream scBridge run reproduces expected label transfer quality (e.g., 89.66% F1 on the mouse atlas vs scJoint 64.60% and Seurat 44.64%)
- Transferred labels and reliability scores appear in obs['Prediction'] and obs['Reliability'] of the output AnnData

## Limitations

- Discussion of the TF-IDF preprocessing choice is deferred to Supplementary Note 3, which is not included in the provided document
- Peak-to-gene-activity conversion depends on an external tool (Signac); scBridge itself accepts only AnnData h5ad inputs with obs['CellType'] annotations in the source
- Cells annotated 'Unknown' or not belonging to shared cell types between modalities should be excluded before preprocessing (e.g., 18 common types reserved for the mouse atlas)

## Evidence

- [readme] For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac (https://github.com/stuart-lab/signac)."
- [methods] transformed into a scATAC-seq activity matrix using tools such as Signac: "transformed into a scATAC-seq activity matrix using tools such as Signac (https://github.com/stuart-lab/signac)"
- [discussion] normalized each cell by dividing its total number of read counts: "we first normalized each cell by dividing its total number of read counts on all genes, and then multiplied them by 10,000"
- [discussion] TF-IDF preprocessing discussion deferred to Supplementary Note 3: "(more discussions are provided in Supplementary Note 3)"
- [discussion] Reserved cells of 18 common types and 12,689 common genes: "We reserved cells of 18 common types and 12,689 common genes for the analysis"

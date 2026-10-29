---
name: scrna-seq-qc-filtering-and-normalization
description: Use when raw scRNA-seq/scATAC-seq count matrices (e.g. filtered_feature_bc_matrix.h5
  plus ATAC fragments .tsv.gz) must be QC-filtered and normalized in R/Seurat/Signac
  before multimodal integration or pairing.
when_to_use_negative:
- Input is already a QC-filtered, normalized, and dimension-reduced co-embedding (e.g.
  a Seurat object with a CCA/Harmony reduction) — proceed directly to PairCells/CreatePairedObject
  instead.
- Data are not paired single-cell multiome counts but unimodal bulk or pre-aggregated
  feature tables, where per-cell QC thresholds like percent.mt < 20 are not applicable.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Loading 10x Multiome RNA counts, per-cell QC filtering (nFeature_RNA, percent.mt),
    NormalizeData/FindVariableFeatures/ScaleData/RunPCA, and label transfer to annotate
    cell types
  repo: github.com/satijalab/seurat
- name: Signac
  role: Loading ATAC peak counts from Multiome files and preprocessing with RunTFIDF/FindTopFeatures/RunSVD
    to obtain the LSI embedding
  repo: github.com/stuart-lab/signac
- name: scMEGA
  role: Consumes the QC-filtered, normalized embeddings for CCA co-embedding (CoembedData)
    and cell pairing (PairCells); installed via devtools from GitHub
  repo: github.com/CostaLab/scMEGA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:6e22a61715159cba8d1a130c6a333b3b6bc18e3ffdcbea6bbad72e2360fd274b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Seurat
  - Signac
  - scMEGA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrna-seq-qc-filtering-and-normalization

## Summary

Quality-control filtering and modality-specific normalization/preprocessing of single-cell multiome data as used in the scMEGA PBMC 10x Multiome workflow: low-quality RNA cells are removed with explicit thresholds, RNA is normalized and PCA-embedded, and ATAC peaks are TF-IDF transformed and LSI-embedded before downstream integration.

## When to use

Use when raw scRNA-seq/scATAC-seq count matrices (e.g. filtered_feature_bc_matrix.h5 plus ATAC fragments .tsv.gz) must be QC-filtered and normalized in R/Seurat/Signac before multimodal integration or pairing.

## When NOT to use

- Input is already a QC-filtered, normalized, and dimension-reduced co-embedding (e.g. a Seurat object with a CCA/Harmony reduction) — proceed directly to PairCells/CreatePairedObject instead.
- Data are not paired single-cell multiome counts but unimodal bulk or pre-aggregated feature tables, where per-cell QC thresholds like percent.mt < 20 are not applicable.

## Inputs

- 10x Multiome filtered_feature_bc_matrix.h5 (gene expression + peak counts)
- ATAC fragments file (.tsv.gz) and its .tbi index
- Seurat/Signac objects for the RNA and ATAC modalities
- Annotated PBMC reference Seurat object for label transfer

## Outputs

- QC-filtered Seurat (RNA) and Signac (ATAC) objects restricted to shared barcodes
- RNA PCA embedding (30 dims) after NormalizeData/FindVariableFeatures/ScaleData
- ATAC LSI embedding (dims 2:30) after RunTFIDF/FindTopFeatures/RunSVD
- Cell-type annotations via Seurat label transfer (FindTransferAnchors/TransferData, 30 dims)

## How to apply

Load gene-expression and peak counts into Seurat/Signac objects from the 10x Multiome files (filtered_feature_bc_matrix.h5, ATAC fragments .tsv.gz and index), then filter low-quality RNA cells using nFeature_RNA > 200 and < 3000 with percent.mt < 20, and restrict both modalities to the shared barcodes. Preprocess the RNA modality with NormalizeData/FindVariableFeatures/ScaleData/RunPCA using 30 principal components. Preprocess the ATAC modality with RunTFIDF, FindTopFeatures, and RunSVD, keeping LSI dimensions 2:30 because the first LSI dimension correlates with sequencing depth. These QC/normalized embeddings are the required input for downstream CCA co-embedding (CoembedData) and cell pairing (PairCells) in scMEGA.

## Related tools

- **Seurat** (Loading 10x Multiome RNA counts, per-cell QC filtering (nFeature_RNA, percent.mt), NormalizeData/FindVariableFeatures/ScaleData/RunPCA, and label transfer to annotate cell types) — github.com/satijalab/seurat
- **Signac** (Loading ATAC peak counts from Multiome files and preprocessing with RunTFIDF/FindTopFeatures/RunSVD to obtain the LSI embedding) — github.com/stuart-lab/signac
- **scMEGA** (Consumes the QC-filtered, normalized embeddings for CCA co-embedding (CoembedData) and cell pairing (PairCells); installed via devtools from GitHub) — github.com/CostaLab/scMEGA

## Examples

```
obj.rna <- NormalizeData(obj.rna); obj.rna <- FindVariableFeatures(obj.rna); obj.rna <- ScaleData(obj.rna); obj.rna <- RunPCA(obj.rna, npcs = 30)
```

## Evaluation signals

- QC filter must enforce nFeature_RNA > 200 and < 3000 and percent.mt < 20, and both modalities must be restricted to the same shared barcodes
- RNA PCA must be run with 30 dimensions; ATAC SVD must yield LSI dims 2:30 with dimension 1 excluded (depth-correlated)
- Cell-type annotation is produced by Seurat label transfer (FindTransferAnchors/TransferData, 30 dims) from the annotated PBMC reference
- Downstream pairing on the co-embedding should reproduce the benchmark: 71.8% of cells matched to the correct cell type and n=41 true pairs recovered, written to pairing_metrics.csv
- sessionInfo() output should be recorded because package versions are not specified in the tutorials

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so exact reproducibility of version-dependent results is not guaranteed
- QC thresholds and dim ranges are tuned to the PBMC 10x Multiome (pbmc_granulocyte_sorted_10k) dataset; other tissues may need different cutoffs
- Cell-type mismatches after pairing generally occur between similar sub-populations, e.g. non-classical and intermediate monocytes

## Evidence

- [other] Load gene-expression and peak counts into Seurat/Signac objects, filter low-quality RNA cells (nFeature_RNA > 200 and < 3000, percent.mt < 20): [evidence span withheld — anchor status: not_found]
- [other] Preprocess RNA with NormalizeData/FindVariableFeatures/ScaleData/RunPCA (30 dims) and ATAC with RunTFIDF/FindTopFeatures/RunSVD (LSI dims 2:30...): [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("satijalab/seurat", "seurat5", quiet = TRUE): "devtools::install_github("satijalab/seurat", "seurat5", quiet = TRUE)"
- [methods] # Check session information
sessionInfo(): "# Check session information
sessionInfo()"

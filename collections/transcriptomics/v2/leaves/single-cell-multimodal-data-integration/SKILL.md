---
name: single-cell-multimodal-data-integration
description: Use when two single-cell omics matrices (e.g., snRNA-seq counts and snATAC-seq
  peak counts from the same study but unpaired barcodes, such as 10x Multiome PBMC
  data) must be co-embedded and cross-modal cell pairing is needed.
when_to_use_negative:
- The dataset is already intrinsically paired (same barcode in both modalities), so
  cross-modal cell matching is unnecessary — proceed directly to paired-object analysis.
- Input is a single-modality dataset with no second omics layer to integrate.
- The goal is enhancer-to-promoter linking or GRN inference itself (AddMotifs/RunChromVAR)
  rather than modality integration and cell pairing.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3940
tools:
- name: scMEGA
  role: provides CoembedData, PairCells (optmatch method), and CreatePairedObject
    for CCA-based integration and cross-modal cell pairing
  repo: github.com/CostaLab/scMEGA
- name: Seurat v5
  role: supplies the CCA co-embedding method wrapped by CoembedData, plus PCA and
    label-transfer anchors (FindTransferAnchors/TransferData)
  repo: github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: ATAC-seq preprocessing in Seurat-object format (RunTFIDF, FindTopFeatures,
    RunSVD for LSI)
  repo: github.com/stuart-lab/signac (branch seurat5)
- name: ArchR
  role: referenced integration/trajectory approach used alongside Seurat in scMEGA
    workflows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:954ab38606e7ddf16a08a3297fdbc41cc645536391fd17fab2e279ef7d2e2005
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Seurat v5
  - Signac
  - scMEGA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-multimodal-data-integration

## Summary

Integrate single-cell RNA-seq and ATAC-seq modalities by projecting both into a common low-dimensional space via Seurat's CCA method (wrapped in scMEGA's CoembedData), optionally batch-correcting with Harmony, and pairing cells across modalities with PairCells. This is the foundational co-embedding step of the scMEGA end-to-end multi-omics analysis workflow.

## When to use

Use when two single-cell omics matrices (e.g., snRNA-seq counts and snATAC-seq peak counts from the same study but unpaired barcodes, such as 10x Multiome PBMC data) must be co-embedded and cross-modal cell pairing is needed.

## When NOT to use

- The dataset is already intrinsically paired (same barcode in both modalities), so cross-modal cell matching is unnecessary — proceed directly to paired-object analysis.
- Input is a single-modality dataset with no second omics layer to integrate.
- The goal is enhancer-to-promoter linking or GRN inference itself (AddMotifs/RunChromVAR) rather than modality integration and cell pairing.

## Inputs

- Seurat/Signac object of gene-expression counts (e.g., filtered_feature_bc_matrix.h5 from 10x Multiome), quality-filtered (nFeature_RNA > 200 and < 3000, percent.mt < 20)
- Signac object of ATAC peak counts with LSI reduction (TF-IDF, top features, SVD on dims 2:30)
- Annotated PBMC reference Seurat object for label-transfer-based cell-type annotation
- Metadata covariates for batch correction (e.g., patient, region, tech)

## Outputs

- Co-embedded Seurat object with a joint low-dimensional reduction (CCA-based, optionally Harmony-corrected, e.g., 'harmony')
- Cross-modal cell-pairing data frame from PairCells mapping each ATAC cell to an RNA cell
- Paired multiomic object via CreatePairedObject suitable for downstream trajectory and GRN analysis
- Pairing evaluation metrics (e.g., fraction of cells matched to the correct cell type, number of true pairs) written to pairing_metrics.csv

## How to apply

Load each modality into Seurat/Signac objects and restrict them to shared barcodes; preprocess RNA with NormalizeData/FindVariableFeatures/ScaleData/RunPCA (30 dims) and ATAC with RunTFIDF/FindTopFeatures/RunSVD using LSI dims 2:30 (dropping the depth-correlated first LSI component). Call scMEGA's CoembedData, which wraps several Seurat functions, to project both modalities into a common low-dimensional space using Seurat's CCA method; when samples span patients, regions, or technologies, apply Harmony batch correction on group.by.vars such as c('patient','region','tech'). For cell pairing, call PairCells on the co-embedding reduction (e.g., reduction='harmony') using the optmatch method to match each snATAC-seq cell to an snRNA-seq cell, then optionally build a combined object with CreatePairedObject. Annotate matched cells (e.g., by Seurat label transfer with FindTransferAnchors/TransferData, 30 dims) to evaluate pairing against known multiome barcodes.

## Related tools

- **scMEGA** (provides CoembedData, PairCells (optmatch method), and CreatePairedObject for CCA-based integration and cross-modal cell pairing) — github.com/CostaLab/scMEGA
- **Seurat v5** (supplies the CCA co-embedding method wrapped by CoembedData, plus PCA and label-transfer anchors (FindTransferAnchors/TransferData)) — github.com/satijalab/seurat (branch seurat5)
- **Signac** (ATAC-seq preprocessing in Seurat-object format (RunTFIDF, FindTopFeatures, RunSVD for LSI)) — github.com/stuart-lab/signac (branch seurat5)
- **ArchR** (referenced integration/trajectory approach used alongside Seurat in scMEGA workflows)

## Examples

```
df.pair <- PairCells(object = coembed.sub, reduction = "harmony", method = "optmatch")
```

## Evaluation signals

- On the 10x Multiome PBMC benchmark (pbmc_granulocyte_sorted_10k), pairing should recover ~71.8% of cells matched to the correct cell type, with mismatches concentrated in similar sub-populations such as non-classical vs intermediate monocytes.
- The same benchmark should recover n=41 true pairs (barcodes shared between modalities) correctly; computed metrics in pairing_metrics.csv should reproduce these values.
- After Harmony batch correction on patient/region/tech, a UMAP embedding should show cells from different patients, regions, and modalities well mixed.
- Each ATAC cell should receive exactly one RNA-cell match from PairCells; the pair data frame should cover all cells of the queried modality.
- LSI should exclude the depth-correlated first dimension — verify the ATAC reduction uses dims 2:30, not 1:30.

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is suggested), so exact reproducibility of the 71.8% / n=41 benchmark may depend on package versions.
- Cell-pairing mismatches arise between closely related sub-populations (e.g., non-classical and intermediate monocytes), so pairing accuracy is cell-type dependent.
- PairCells is demonstrated on sub-clustered co-embedded objects (coembed.sub with a chosen reduction); results depend on upstream clustering and reduction choice (CCA vs harmony).
- No changelog or discussion section is available for the article, limiting versioning guidance.

## Evidence

- [methods] CCA co-embedding via CoembedData: "we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]"
- [methods] CoembedData wrapper: "we have wrapped several functions from Seurat into a single function _CoembedData_."
- [methods] Cell pairing: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [methods] PairCells invocation: "df.pair <- PairCells(object = coembed.sub, reduction = "harmony","
- [methods] Harmony batch correction: "we use [Harmony](https://www.nature.com/articles/s41592-019-0619-0) to 
perform batch correction and generate a new UMAP embedding."
- [readme] README install instructions: "devtools::install_github("CostaLab/scMEGA")"
- [readme] README PBMC multiome tutorial: "[CD4 T cells activcation using 10X multiome data](https://costalab.github.io/scMEGA/articles/pbmc_10x_multiome.html)"

---
name: cross-modality-cell-pairing
description: Use when single-nucleus RNA and ATAC data from the same tissue were captured
  in separate assays (no shared barcodes) and downstream pseudotime or eGRN analysis
  needs each ATAC cell paired to an RNA cell.
when_to_use_negative:
- Data are true 10x Multiome with shared barcodes — cells are already physically paired
  and PairCells is only needed for benchmarking predicted vs true pairs
- Input is already a paired multi-omic object ready for AddTrajectory or GetGRN
- The goal is modality integration for visualization only, without needing one-to-one
  cell pairing
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scMEGA
  role: Provides CoembedData, PairCells, and CreatePairedObject for co-embedding,
    greedy/optmatch cell pairing, and paired-object assembly
  repo: github.com/CostaLab/scMEGA
- name: Seurat
  role: CCA-based integration used under the hood by CoembedData to project RNA and
    ATAC into a common low-dimensional space
  repo: github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: Handles the snATAC-seq assay and gene activity matrix used as pairing input
  repo: github.com/stuart-lab/signac (branch seurat5)
- name: harmony
  role: Batch correction (RunHarmony over patient/region/tech) of the co-embedding
    prior to PairCells
- name: optmatch
  role: Optimal matching method used by PairCells in the PBMC multiome pairing benchmark
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:43f84251870b02207cd21ead2765deb9274851478aa372b49684e582e00b0c13
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - JASPAR2020
  - Nebulosa
  - R
  - Seurat
  - Signac
  - TFBSTools
  - destiny
  - devtools
  - ggraph
  - harmony
  - igraph
  - optmatch
  - scMEGA
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# cross-modality-cell-pairing

## Summary

Match cells between snRNA-seq and snATAC-seq modalities after CCA co-embedding and Harmony batch correction, so that paired multi-omic cells can feed trajectory and enhancer-based GRN inference in scMEGA.

## When to use

Use when single-nucleus RNA and ATAC data from the same tissue were captured in separate assays (no shared barcodes) and downstream pseudotime or eGRN analysis needs each ATAC cell paired to an RNA cell.

## When NOT to use

- Data are true 10x Multiome with shared barcodes — cells are already physically paired and PairCells is only needed for benchmarking predicted vs true pairs
- Input is already a paired multi-omic object ready for AddTrajectory or GetGRN
- The goal is modality integration for visualization only, without needing one-to-one cell pairing

## Inputs

- Seurat v5 co-embedded object (obj.coembed) with harmony reduction containing snRNA-seq and snATAC-seq modalities
- Sub-clustered Seurat/Signac objects (e.g. fibroblast snRNA.rds, snATAC.rds, gene.activity.rds from costalab.ukaachen.de/open_data/scMEGA/Fibroblast)
- 10x Multiome files (filtered_feature_bc_matrix.h5, ATAC fragments .tsv.gz) when pairing quality is benchmarked against true barcodes

## Outputs

- Cell-pairing table (df.pair) mapping each snATAC-seq cell to a matched snRNA-seq cell
- Paired multi-omic Seurat object (obj.pair) from CreatePairedObject
- Pairing quality metrics (fraction of cells matched to correct cell type, number of true pairs, e.g. pairing_metrics.csv)

## How to apply

First co-embed the RNA and ATAC modalities in a common low-dimensional space with scMEGA's CoembedData (Seurat CCA on the gene activity matrix), then batch-correct with RunHarmony over group-by variables such as patient, region, and tech and compute a UMAP on dims 1:30. Restrict to the relevant sub-clustered cell population (e.g. fibroblast sub-clusters) before pairing. Call PairCells on the harmony reduction — pair.mode='greedy' is used in the myofibroblast case study, and the optmatch method is used in the 10x Multiome PBMC benchmark — to find, for each cell in one modality, a matched cell from the other. Assemble a paired multi-omic object with CreatePairedObject from the pairing table and co-embedded object; pseudotime (AddTrajectory on diffusion-map dims 1:3) and TF-gene correlations are then computed on paired cells. Pairing quality is validated against known multiome barcodes by the fraction of cells matched to the correct cell type and the number of true pairs recovered.

## Related tools

- **scMEGA** (Provides CoembedData, PairCells, and CreatePairedObject for co-embedding, greedy/optmatch cell pairing, and paired-object assembly) — github.com/CostaLab/scMEGA
- **Seurat** (CCA-based integration used under the hood by CoembedData to project RNA and ATAC into a common low-dimensional space) — github.com/satijalab/seurat (branch seurat5)
- **Signac** (Handles the snATAC-seq assay and gene activity matrix used as pairing input) — github.com/stuart-lab/signac (branch seurat5)
- **harmony** (Batch correction (RunHarmony over patient/region/tech) of the co-embedding prior to PairCells)
- **optmatch** (Optimal matching method used by PairCells in the PBMC multiome pairing benchmark)

## Examples

```
df.pair <- PairCells(object = coembed.sub, reduction = "harmony", pair.mode = "greedy"); obj.pair <- CreatePairedObject(df.pair = df.pair, object = coembed.sub2)
```

## Evaluation signals

- On 10x Multiome PBMC data (pbmc_granulocyte_sorted_10k), pairing should match 71.8% of cells to the correct cell type and recover n=41 true pairs
- Mismatches, if any, should concentrate between similar sub-populations (e.g. non-classical vs intermediate monocytes), not across unrelated cell types
- df.pair from PairCells must contain one matched RNA cell per ATAC cell and be accepted by CreatePairedObject
- For CD4 T cell eGRNs, networks from predicted pairs should recover >75% of TFs, 83% of genes, and 60% of TF-gene regulations relative to true-pair networks (edge filter correlation > 0.5)
- Co-embedding UMAP should show cells from different patients, regions, and modalities well mixed after Harmony correction

## Limitations

- Mismatches generally occur between similar sub-populations such as non-classical and intermediate monocytes
- Not all co-embedded cells are retained: sub-clustering and CompareCellProp filtering (e.g. clusters with significant group differences across patient_group) precede pairing
- Software/package versions are not specified in the tutorials; pairing quality depends on Seurat v5/Signac seurat5 branch installs
- The first LSI dimension is depth-correlated and excluded (dims 2:30) when building the ATAC embedding used for co-embedding

## Evidence

- [methods] cell pairing between modalities: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [methods] PairCells usage: "df.pair <- PairCells(object = coembed.sub, reduction = "harmony","
- [methods] CreatePairedObject usage: "obj.pair <- CreatePairedObject(df.pair = df.pair, object = coembed.sub2,"
- [methods] CCA co-embedding rationale: "we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]"
- [results] pairing benchmark result: [evidence span withheld — anchor status: not_found]
- [methods] batch-corrected mixing: "after batch-correction, cells from different patients, regions, and modalities are well mixed."

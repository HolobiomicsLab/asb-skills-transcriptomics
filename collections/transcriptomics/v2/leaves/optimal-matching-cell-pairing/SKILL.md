---
name: optimal-matching-cell-pairing
description: Use when you have two unpaired single-cell modalities (e.g. snATAC-seq
  and snRNA-seq) already co-embedded in a common low-dimensional space and need matched
  cell pairs to build a pseudo-multimodal dataset.
when_to_use_negative:
- Data are truly paired multiome (e.g. 10x Multiome) where the same cell was assayed
  in both modalities — pairing is for unpaired modalities
- Modalities have not been co-embedded and batch-corrected into a shared low-dimensional
  space yet; pairing on uncorrected embeddings will match across batch/modality artifacts
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMEGA
  role: provides PairCells and CreatePairedObject for one-to-one cell pairing and
    pseudo-multimodal object assembly
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: supplies the CCA co-embedding and Seurat object framework on which pairing
    is performed
  repo: https://github.com/satijalab/seurat
- name: optmatch
  role: optimal one-to-one matching method (Kartha et al.) used by PairCells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:74e0ca17590e67e92536a2af350cf9a1830be81ec54690a7c5fa265f20991511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Seurat
  - Signac
  - optmatch
  - scMEGA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# optimal-matching-cell-pairing

## Summary

Performs one-to-one cell pairing between unpaired snATAC-seq and snRNA-seq modalities using scMEGA's PairCells on the shared harmony co-embedding, then assembles the matched cells into a pseudo-multimodal Seurat object with CreatePairedObject.

## When to use

Use when you have two unpaired single-cell modalities (e.g. snATAC-seq and snRNA-seq) already co-embedded in a common low-dimensional space and need matched cell pairs to build a pseudo-multimodal dataset.

## When NOT to use

- Data are truly paired multiome (e.g. 10x Multiome) where the same cell was assayed in both modalities — pairing is for unpaired modalities
- Modalities have not been co-embedded and batch-corrected into a shared low-dimensional space yet; pairing on uncorrected embeddings will match across batch/modality artifacts

## Inputs

- Co-embedded Seurat object (Assay5 format) containing snRNA-seq and snATAC-seq cells with a 'harmony' reduction (coembed.sub)
- Harmony-corrected UMAP/low-dimensional representation over patient, region, and tech covariates

## Outputs

- Pairing data frame (df.pair) mapping each ATAC cell to one matched RNA cell
- Pseudo-multimodal Seurat object created by CreatePairedObject (obj.pair)
- Saved RDS artifacts (coembed.rds, coembed.sub.rds)

## How to apply

First co-embed the snRNA-seq and snATAC-seq cells via scMEGA's CoembedData (Seurat CCA on the ArchR-derived gene activity matrix for ATAC cells) and run RunHarmony over patient, region, and tech covariates (dims 1:30) so modalities mix in the shared space. Then call PairCells on the co-embedded/sub-clustered object using the harmony reduction, with the optmatch method of Kartha et al., to find for each snATAC-seq cell a matched snRNA-seq cell in a one-to-one pairing. Pass the resulting pairing data frame and the sub-clustered co-embedding object to CreatePairedObject to assemble a pseudo-multimodal Seurat object that carries both modalities' information per paired cell, and save it (e.g. coembed.rds, coembed.sub.rds via saveRDS).

## Related tools

- **scMEGA** (provides PairCells and CreatePairedObject for one-to-one cell pairing and pseudo-multimodal object assembly) — https://github.com/CostaLab/scMEGA
- **Seurat** (supplies the CCA co-embedding and Seurat object framework on which pairing is performed) — https://github.com/satijalab/seurat
- **optmatch** (optimal one-to-one matching method (Kartha et al.) used by PairCells)

## Examples

```
df.pair <- PairCells(object = coembed.sub, reduction = "harmony")
```

## Evaluation signals

- Each ATAC cell is matched to at most one RNA cell (one-to-one pairing, verified by inspecting df.pair)
- Harmony-corrected UMAP before pairing shows cells from different patients, regions, and modalities are well mixed
- CreatePairedObject returns a valid, loadable Seurat object containing both modalities' data per paired cell
- Saved RDS files (coembed.rds, coembed.sub.rds) exist and load as Seurat objects

## Limitations

- Pairing quality depends on the upstream Harmony batch correction; if modalities/patients/regions are not well mixed in the UMAP, paired cells are unreliable
- Requires Seurat v5 with Assay5-format objects as set up in the scMEGA workflow
- Unmatched cells are left out of the pseudo-multimodal object, so downstream GRN inference operates only on the paired subset

## Evidence

- [methods] cell pairing methods step: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [methods] PairCells call: "df.pair <- PairCells(object = coembed.sub, reduction = "harmony","
- [methods] CreatePairedObject call: "obj.pair <- CreatePairedObject(df.pair = df.pair, object = coembed.sub2,"
- [methods] batch mixing prerequisite: "after batch-correction, cells from different patients, regions, and modalities are well mixed."
- [abstract] scMEGA scope: "We here describe scMEGA (Single-cell Multiomic Enhancer-based Gene Regulatory Network Inference) that enables an end-to-end analysis of multi-omics data"

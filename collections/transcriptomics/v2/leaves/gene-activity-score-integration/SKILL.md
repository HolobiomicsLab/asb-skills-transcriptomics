---
name: gene-activity-score-integration
description: Use when snRNA-seq and snATAC-seq come from unpaired samples and you
  need a shared low-dimensional representation before cell pairing or downstream GRN
  inference; the ATAC assay must be converted to a gene activity matrix.
when_to_use_negative:
- Data are paired 10x Multiome (pbmc_granulocyte_sorted_10k) measured in the same
  cells — no unpaired integration or cell pairing is needed.
- Input is a pre-integrated or already co-embedded multimodal object; re-running CCA
  co-embedding would be redundant.
- The goal is single-modality analysis of ATAC peaks without a gene-level feature
  space; gene activity integration is unnecessary.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3940
tools:
- name: scMEGA
  role: Provides CoembedData, PairCells, and CreatePairedObject for CCA co-embedding,
    optmatch cell pairing, and pseudo-multimodal object assembly
  repo: github.com/CostaLab/scMEGA
- name: Seurat v5
  role: CCA integration wrapped by CoembedData to project snRNA-seq and snATAC-seq
    into a common low-dimensional space
  repo: github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: Seurat-v5-compatible single-cell chromatin object handling for the snATAC-seq
    assay
  repo: github.com/stuart-lab/signac (branch seurat5)
- name: ArchR
  role: Generates the gene activity matrix used as the ATAC-side feature representation
    for CCA integration
- name: harmony
  role: Batch correction on the co-embedded cells over patient, region, and technology
    covariates (RunHarmony, dims 1:30)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:f9d93e87441cc47290a7822abd39bf880a4c5d9be99c26580ab69ef571b1a227
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Seurat v5
  - Signac
  - harmony
  - optmatch
  - scMEGA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gene-activity-score-integration

## Summary

Use ArchR-derived gene activity scores from snATAC-seq as the common feature space for integrating unpaired snRNA-seq and snATAC-seq data in scMEGA, enabling CCA co-embedding, Harmony batch correction, and one-to-one cell pairing into a pseudo-multimodal dataset.

## When to use

Use when snRNA-seq and snATAC-seq come from unpaired samples and you need a shared low-dimensional representation before cell pairing or downstream GRN inference; the ATAC assay must be converted to a gene activity matrix.

## When NOT to use

- Data are paired 10x Multiome (pbmc_granulocyte_sorted_10k) measured in the same cells — no unpaired integration or cell pairing is needed.
- Input is a pre-integrated or already co-embedded multimodal object; re-running CCA co-embedding would be redundant.
- The goal is single-modality analysis of ATAC peaks without a gene-level feature space; gene activity integration is unnecessary.

## Inputs

- snRNA-seq Seurat object (Assay5 format)
- snATAC-seq Seurat object with ArchR-derived gene activity matrix (Assay5 format)
- Covariate metadata columns: patient, region, tech

## Outputs

- Co-embedded Seurat object with harmony reduction and mixed UMAP (coembed.rds)
- Sub-clustered co-embedded object (coembed.sub.rds)
- Paired-cell data frame from PairCells (df.pair)
- Pseudo-multimodal Seurat object from CreatePairedObject (obj.pair)

## How to apply

Convert the snATAC-seq assay to an ArchR-derived gene activity matrix and load both the snRNA-seq and snATAC-seq Seurat objects as Assay5 objects for Seurat v5. Call scMEGA's CoembedData, which wraps Seurat's CCA integration, using the gene activity matrix as the ATAC-side feature representation so both modalities project into a common low-dimensional space. Run Harmony batch correction on the co-embedded object over patient, region, and technology covariates with dims 1:30 (weight.reduction = "harmony"), then inspect the new UMAP to confirm cells from different patients, regions, and modalities are well mixed. Finally use PairCells on the harmony reduction with the optmatch method (Kartha et al.) to find, for each snATAC-seq cell, a matched snRNA-seq cell, and assemble the pairs with CreatePairedObject into a pseudo-multimodal Seurat object.

## Related tools

- **scMEGA** (Provides CoembedData, PairCells, and CreatePairedObject for CCA co-embedding, optmatch cell pairing, and pseudo-multimodal object assembly) — github.com/CostaLab/scMEGA
- **Seurat v5** (CCA integration wrapped by CoembedData to project snRNA-seq and snATAC-seq into a common low-dimensional space) — github.com/satijalab/seurat (branch seurat5)
- **Signac** (Seurat-v5-compatible single-cell chromatin object handling for the snATAC-seq assay) — github.com/stuart-lab/signac (branch seurat5)
- **ArchR** (Generates the gene activity matrix used as the ATAC-side feature representation for CCA integration)
- **harmony** (Batch correction on the co-embedded cells over patient, region, and technology covariates (RunHarmony, dims 1:30))

## Examples

```
df.pair <- PairCells(object = coembed.sub, reduction = "harmony", method = "optmatch")
```

## Evaluation signals

- Harmony-corrected UMAP shows cells from different patients, regions, and modalities are well mixed.
- coembed.rds and coembed.sub.rds exist and load as valid Seurat objects via readRDS.
- PairCells returns a one-to-one df.pair mapping where each snATAC-seq cell is matched to exactly one snRNA-seq cell.
- CreatePairedObject yields a pseudo-multimodal Seurat object containing both RNA and ATAC assays for the paired cells.
- Both input assays were successfully converted to Assay5 format for Seurat v5 compatibility.

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is suggested).
- Integration quality depends on the gene activity matrix as a proxy for ATAC-side gene expression; scMEGA targets enhancer-based GRN inference and pairing is approximate for unpaired data.
- No changelog found for the repository.

## Evidence

- [methods] CoembedData wraps Seurat CCA: "we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]"
- [methods] Harmony batch correction: "after batch-correction, cells from different patients, regions, and modalities are well mixed."
- [methods] Cell pairing: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [methods] RunHarmony call: "obj.coembed <- RunHarmony(obj.coembed, 
                          group.by.vars = c("patient", "region", "tech"),"
- [methods] PairCells call: "df.pair <- PairCells(object = coembed.sub, reduction = "harmony","
- [readme] README install steps: "devtools::install_github("satijalab/seurat", "seurat5", quiet = TRUE)"

---
name: harmony-batch-correction
description: Use when unpaired snRNA-seq and snATAC-seq cells from multiple patients,
  regions, or technologies have been co-embedded (e.g. via Seurat CCA in CoembedData)
  but remain separated by batch, and a corrected embedding is needed for sub-clustering,
  PairCells, or trajectory analysis.
when_to_use_negative:
- Data are already paired multiome (e.g. 10x multiome) where modalities share cells
  natively and no cross-modality batch correction is needed
- A single-modality dataset without batch covariates, where Harmony correction has
  no group.by.vars to correct
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: harmony
  role: Performs batch correction on the co-embedded object via RunHarmony over patient,
    region, and technology covariates
- name: scMEGA
  role: Provides CoembedData (Seurat CCA wrapping with weight.reduction='harmony')
    and PairCells which consumes the harmony reduction
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: Supplies CCA integration underlying CoembedData and UMAP generation on the
    corrected embedding
  repo: https://github.com/satijalab/seurat
- name: optmatch
  role: Method used by PairCells for one-to-one cell pairing on the harmony reduction
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:d77d11a0aa147edf87ca1b4280002c6bcf74c1b7f0d286c6fc6c87e7105b420d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# harmony-batch-correction

## Summary

Applies Harmony batch correction to a co-embedded multi-modal single-cell object (e.g. scMEGA's CCA co-embedding of snRNA-seq and snATAC-seq) so that cells from different donors, regions, and technologies mix in low-dimensional space before clustering, cell pairing, and trajectory/GRN inference.

## When to use

Use when unpaired snRNA-seq and snATAC-seq cells from multiple patients, regions, or technologies have been co-embedded (e.g. via Seurat CCA in CoembedData) but remain separated by batch, and a corrected embedding is needed for sub-clustering, PairCells, or trajectory analysis.

## When NOT to use

- Data are already paired multiome (e.g. 10x multiome) where modalities share cells natively and no cross-modality batch correction is needed
- A single-modality dataset without batch covariates, where Harmony correction has no group.by.vars to correct

## Inputs

- Co-embedded Seurat object (obj.coembed) from CoembedData containing both snRNA-seq and snATAC-seq modalities in Assay5 format
- Batch covariate metadata columns (e.g. patient, region, tech)
- Gene activity matrix / ArchR-derived gene activity used for CCA co-embedding

## Outputs

- Seurat object with a 'harmony' batch-corrected low-dimensional reduction (dims 1:30)
- New UMAP embedding computed on the harmony reduction for mixing diagnostics
- Corrected embedding used as input to PairCells, CreatePairedObject, and RunDiffusionMap

## How to apply

First build a shared low-dimensional space with scMEGA's CoembedData, which wraps Seurat's CCA integration using the gene activity matrix as the ATAC-side assay. Then run RunHarmony on the co-embedded object with group.by.vars covering all batch covariates (in the fibroblast MI case study: c("patient", "region", "tech")) and dims 1:30, producing a 'harmony' reduction. Generate a new UMAP on the harmony embedding and inspect it to confirm cells from different patients, regions, and modalities are well mixed before proceeding to FindClusters and downstream steps. The corrected harmony reduction is subsequently used for one-to-one cell pairing via PairCells (pair.mode='greedy') and for RunDiffusionMap prior to trajectory and GRN inference.

## Related tools

- **harmony** (Performs batch correction on the co-embedded object via RunHarmony over patient, region, and technology covariates)
- **scMEGA** (Provides CoembedData (Seurat CCA wrapping with weight.reduction='harmony') and PairCells which consumes the harmony reduction) — https://github.com/CostaLab/scMEGA
- **Seurat** (Supplies CCA integration underlying CoembedData and UMAP generation on the corrected embedding) — https://github.com/satijalab/seurat
- **optmatch** (Method used by PairCells for one-to-one cell pairing on the harmony reduction)

## Examples

```
obj.coembed <- RunHarmony(obj.coembed, group.by.vars = c("patient", "region", "tech"), reduction = "cca", dims = 1:30)
```

## Evaluation signals

- UMAP computed on the harmony reduction shows cells from different patients, regions, and modalities well mixed (article's stated success criterion)
- harmony reduction exists in the Seurat object with the requested dimensionality (dims 1:30)
- Downstream PairCells returns a valid one-to-one pairing table (df.pair) between ATAC and RNA cells
- Sub-clustering on the corrected embedding reproduces myofibroblast populations (e.g. cluster 1 classified as myofibroblasts by top-10 marker dot plot)
- Saved RDS outputs (coembed.rds, coembed.sub.rds) exist and load as valid Seurat objects

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is suggested), so reproducibility of exact embeddings may vary
- One-to-one pairing is approximate: unpaired snATAC and snRNA cells are matched greedily via optmatch, so the pseudo-multimodal object is not true paired data
- Mixing must be verified visually on UMAP; the article provides no quantitative mixing metric

## Evidence

- [methods] Batch correction with Harmony: "we use [Harmony](https://www.nature.com/articles/s41592-019-0619-0) to 
perform batch correction and generate a new UMAP embedding."
- [methods] Harmony covariates: "obj.coembed <- RunHarmony(obj.coembed, 
                          group.by.vars = c("patient", "region", "tech"),"
- [methods] Mixing success criterion: "after batch-correction, cells from different patients, regions, and modalities are well mixed."
- [methods] Integration rationale: "we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]"
- [readme] README install and pairing context: "devtools::install_github("satijalab/seurat", "seurat5", quiet = TRUE)"

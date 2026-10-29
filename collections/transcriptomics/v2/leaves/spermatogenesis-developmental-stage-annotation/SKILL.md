---
name: spermatogenesis-developmental-stage-annotation
description: Use when a mouse germ-cell scRNA-seq query (WT or knockout, e.g. Zfp541-KO)
  needs developmental stage annotation or pseudotime placement along spermatogenesis
  by mapping onto the mTCA reference atlas.
when_to_use_negative:
- Query cells contain novel states absent from the mTCA reference — fixed-model projections
  may be misleading; compare unsupervised clustering to mapping results first
- Projection spans sequencing platforms (e.g. Smart-seq2 query onto a 10x-built atlas)
  — performance declines due to differing UMI distributions and sequencing depth (Table
  S5)
- The reference atlas was built with a data-tailored integration strategy incompatible
  with the query — ProjectSVR is integration-agnostic, but verification against unsupervised
  results is still required
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Computes gene set activity scores (ComputeModuleScore), projects query cells
    into reference UMAP (ProjectNewdata), assesses mapping quality (AddProjQual, MapQCPlot),
    transfers labels (KnnLabelTransfer, MajorityVote), and fits the pseudotime ensemble
    SVR (FitEnsembleSVM)
  repo: https://github.com/JarningGau/ProjectSVR
- name: AUCell
  role: Gene activity score method (gss.method) used by ComputeModuleScore; rank-based
    0–1 scores robust to normalization and usable without raw counts
- name: Seurat
  role: Carries the query/reference objects (.qs/.rds), stores ref.umap reductions,
    and subsets cells (subset) at the map-quality cutoff
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:b01d0c05f90a680be28b4526bc5ad9f631c8254579ab1f4e25a67c0e99688b57
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spermatogenesis-developmental-stage-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates spermatogenic germ-cell developmental stages and recovers pseudotime for WT and genetically perturbed (Zfp541-KO, Ythdc2-KO) scRNA-seq by projecting query cells onto the mouse testicular cell atlas (mTCA) with ProjectSVR ensemble SVR models, then transferring stage labels and fitting a pseudotime model against the reference's Pseudotime_dm values.

## When to use

Use when a mouse germ-cell scRNA-seq query (WT or knockout, e.g. Zfp541-KO) needs developmental stage annotation or pseudotime placement along spermatogenesis by mapping onto the mTCA reference atlas.

## When NOT to use

- Query cells contain novel states absent from the mTCA reference — fixed-model projections may be misleading; compare unsupervised clustering to mapping results first
- Projection spans sequencing platforms (e.g. Smart-seq2 query onto a 10x-built atlas) — performance declines due to differing UMI distributions and sequencing depth (Table S5)
- The reference atlas was built with a data-tailored integration strategy incompatible with the query — ProjectSVR is integration-agnostic, but verification against unsupervised results is still required

## Inputs

- Query germ-cell scRNA-seq Seurat object (.qs or .rds, e.g. query_Zfp541-KO.seurat.slim.qs) with genotype labels in orig.ident
- Pre-trained mTCA reference model (model.mTCA.rds)
- mTCA reference atlas (mTCA.seurat.slim.qs) with published Pseudotime_dm values
- mTCA reference gene sets (gene.sets) from the pre-trained model

## Outputs

- Query-cell embeddings in reference UMAP space stored as a 'ref.umap' dimension reduction
- Per-cell mapping quality metrics (mean.knn.dist, mapQ.p.adj)
- Transferred spermatogenic cell-type/stage labels per query cell (KnnLabelTransfer + MajorityVote)
- Predicted pseudotime per filtered query cell plus Pearson correlation against published pseudotime and cumulative-proportion plots by genotype

## How to apply

Load the pre-trained mTCA reference model (model.mTCA.rds, Zenodo record 8350732), the slim mTCA reference atlas carrying published Pseudotime_dm values (mTCA.seurat.slim.qs, Zenodo record 8350746), and the query Seurat object (e.g. query_Zfp541-KO.seurat.slim.qs, Zenodo record 8350748), assigning genotype labels from orig.ident. Compute gene set activity scores on the query with ProjectSVR::ComputeModuleScore using the reference's gene.sets and the AUCell method (the reference model's gss.method) on 10 cores, since AUCell's rank-based 0–1 scores are robust to normalization and work without raw counts. Project query cells into the reference UMAP space with ProjectNewdata on the pre-trained ensemble SVR model (store as a 'ref.umap' reduction) and assess mapping quality with AddProjQual (k = 20, repeats = 1e4). Filter poorly mapped cells with mean.knn.dist < 2.4 (map.q.cutoff = 2.4 in MapQCPlot) before transferring stage labels via KnnLabelTransfer (k = 100) followed by MajorityVote (k = 100, min.prop = 0.3). For pseudotime, fit the ensemble SVR (FitEnsembleSVM, n.models = 20, batch.size = 8000, cores = 10) on mTCA germ-cell gene set scores against Pseudotime_dm, save the extended model (model.mTCA.v2.rds), and predict pseudotime for filtered query cells via ProjectNewdata; compare WT vs KO cumulative distributions along predicted pseudotime colored by transferred cell type.

## Related tools

- **ProjectSVR** (Computes gene set activity scores (ComputeModuleScore), projects query cells into reference UMAP (ProjectNewdata), assesses mapping quality (AddProjQual, MapQCPlot), transfers labels (KnnLabelTransfer, MajorityVote), and fits the pseudotime ensemble SVR (FitEnsembleSVM)) — https://github.com/JarningGau/ProjectSVR
- **AUCell** (Gene activity score method (gss.method) used by ComputeModuleScore; rank-based 0–1 scores robust to normalization and usable without raw counts)
- **Seurat** (Carries the query/reference objects (.qs/.rds), stores ref.umap reductions, and subsets cells (subset) at the map-quality cutoff) — https://github.com/satijalab/seurat

## Examples

```
seu.q <- ProjectSVR::ComputeModuleScore(seu.q, gene.sets = ref.gss, gss.method = 'AUCell', cores = 10); seu.q <- ProjectSVR::ProjectNewdata(seu.q, model.mTCA, dr.name = 'ref.umap')
```

## Evaluation signals

- Pearson correlation between predicted and published pseudotime is approximately r = 0.973 (Fig. S9b)
- No Zfp541-KO cells are assigned to the diplotene stage (Fig. 6c-d), confirming developmental arrest before diplotene
- Poorly mapped cells are removed at the threshold mean.knn.dist < 2.4 before label transfer and pseudotime prediction
- MajorityVote label transfer uses k = 100 with min.prop = 0.3, and majority-vote stage labels are consistent with WT cells progressing to diplotene
- Cumulative proportion of cells along predicted pseudotime, split by genotype (WT vs Zfp541-KO) and colored by transferred cell type, shows KO cells truncated before diplotene

## Limitations

- Projections may be misleading for query cells with novel states absent in the reference; users must compare unsupervised clustering to mapping results
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- Independent SVR models are trained per UMAP dimension, neglecting correlations among embedding dimensions
- Reference building time scales ~O(Nc^2), requiring subsampling to 4000–10,000 cells; software version numbers are not stated

## Evidence

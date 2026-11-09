---
name: decision-level-multimodal-cluster-integration
description: Use when two or more cluster annotation sets (e.g. RNA Leiden labels
  and splicing/PSI NMF subclusters, or RNA/ADT/ATAC modalities) disagree per cell
  and you need a reconciled, stability-scored final assignment rather than a joint-embedding
  integration.
when_to_use_negative:
- A single consistent clustering already exists and no conflicting annotation sets
  need reconciliation
- The goal is a joint latent-space embedding of modalities rather than decision-level
  assignment — Seurat-WNN or TotalVI are the competing data-level approaches, and
  scTriangulate deliberately avoids joint embedding
- More than 15 annotation sets are needed and Shapley runtime is prohibitive — although
  scTriangulate auto-substitutes rank-based selection, very large sources increase
  runtime significantly
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: 'Core framework: Shapley-value mix-and-match of conflicting cluster assignments
    with reassign/TFIDF/SCCAF stability scores; used via ScTriangulate(...).lazy_run()
    and preprocessing helpers (concat_rna_and_other, CLR_normalization, add_annotation,
    add_umap, scanpy_recipe)'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy 1.7.2
  role: Underlying single-cell framework in which scTriangulate is embedded; provides
    preprocessing, QC, filtering, and clustering inputs (e.g. Leiden annotations)
- name: Seurat-WNN
  role: Competing data-level multimodal integration baseline (weight-nearest-neighbor,
    e.g. 50 PCs RNA + 30 PCs PSI) used to benchmark whether decision-level integration
    resolves subpopulations that WNN misses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:1205677ef8dca57a23983c60d4f15fcdebf9696d767c6d9084441c5ae9075fce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - MultiPath-PSI
  - Seurat-WNN
  - scTriangulate
  - scanpy
  - scanpy 1.7.2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# decision-level-multimodal-cluster-integration

## Summary

Decision-level integration of conflicting single-cell clustering solutions (different modalities, algorithms, resolutions) with scTriangulate, which mixes-and-matches cluster assignments using the Shapley value from coalitional game theory plus stability metrics (reassign, TFIDF5/10, SCCAF) instead of building a joint latent space like Seurat WNN or TotalVI.

## When to use

Use when two or more cluster annotation sets (e.g. RNA Leiden labels and splicing/PSI NMF subclusters, or RNA/ADT/ATAC modalities) disagree per cell and you need a reconciled, stability-scored final assignment rather than a joint-embedding integration.

## When NOT to use

- A single consistent clustering already exists and no conflicting annotation sets need reconciliation
- The goal is a joint latent-space embedding of modalities rather than decision-level assignment — Seurat-WNN or TotalVI are the competing data-level approaches, and scTriangulate deliberately avoids joint embedding
- More than 15 annotation sets are needed and Shapley runtime is prohibitive — although scTriangulate auto-substitutes rank-based selection, very large sources increase runtime significantly

## Inputs

- AnnData object with normalized adata.X (log CPTT RNA, CLR ADT, or other modality-appropriate normalization)
- At least two conflicting cluster-annotation columns in adata.obs (from Leiden/Seurat/SnapATAC at different resolutions, label transfer, cellHarmony, or NMF-derived PSI subclusters)
- adata.obsm['X_umap'] coordinates for automatic visualization
- Modality-specific count matrices or normalized values (RNA counts, ADT CLR, PSI values from exon-exon junction counts)

## Outputs

- Final reconciled per-cell cluster assignments written to the user-specified output directory
- Stability-scored cluster labels with unstable clusters flagged (wfc < 0.25)
- UMAP visualizations of the triangulated clusters

## How to apply

Load an AnnData whose adata.X is properly normalized (log CPTT for RNA, CLR for ADT) and that carries at least two conflicting annotation columns in adata.obs plus adata.obsm['X_umap'] (generated via scTriangulate.preprocessing.scanpy_recipe); ensure adata.raw is deleted since it interferes with marker-gene calculation. For multimodal inputs, first normalize each modality (e.g. CLR_normalization for ADT) and merge them with scTriangulate.preprocessing.concat_rna_and_other, or use add_annotation/add_umap to attach external labels such as cellHarmony or label-transfer results. Instantiate ScTriangulate with dir, adata and the query list of annotation columns, then run lazy_run: Shapley-value coalitional iteration selects the most stable solution, and final assignments are pruned by winning fraction (default win_fraction_cutoff, wfc < 0.25 flagged unstable) with a minimum cluster size threshold (default abs_thresh = 10); small or unstable clusters are re-classified by nearest-neighbor. The decision-level approach assumes nothing about the distributions of raw underlying values, so modalities as different as PSI values and RNA counts can be mixed without joint embedding.

## Related tools

- **scTriangulate** (Core framework: Shapley-value mix-and-match of conflicting cluster assignments with reassign/TFIDF/SCCAF stability scores; used via ScTriangulate(...).lazy_run() and preprocessing helpers (concat_rna_and_other, CLR_normalization, add_annotation, add_umap, scanpy_recipe)) — https://github.com/frankligy/scTriangulate
- **scanpy 1.7.2** (Underlying single-cell framework in which scTriangulate is embedded; provides preprocessing, QC, filtering, and clustering inputs (e.g. Leiden annotations))
- **Seurat-WNN** (Competing data-level multimodal integration baseline (weight-nearest-neighbor, e.g. 50 PCs RNA + 30 PCs PSI) used to benchmark whether decision-level integration resolves subpopulations that WNN misses)

## Examples

```
from sctriangulate import *; sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- Each cell receives exactly one definitive annotation in the binary assignment matrix X (shape I x N) — no multi-label assignments
- Clusters with winning fraction wfc < 0.25 are labeled unstable; verify pruning and nearest-neighbor reclassification actually occurred
- Final clusters smaller than the minimum size threshold (default 10) should not survive
- In the AML benchmark, the triangulated solution must retain splicing-defined subpopulations marked by KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events, whereas Seurat-WNN (50 PCs RNA, 30 PCs PSI) fails to resolve them
- All result files are written to the user-specified dir passed to ScTriangulate

## Limitations

- Over 15 annotation sets significantly increases runtime; Shapley selection is auto-replaced by combined rank-based selection in that case
- Programmatic default analytical settings (e.g. wfc 0.25, abs_thresh 10) may not be optimal for all study designs
- Population naming of the resulting integrated clusters remains largely non-standardized
- Integration of multiple samples of the same modality and across technologies is planned but not yet available
- Del adata.raw before running — its presence interferes with how scanpy calculates marker genes

## Evidence

- [abstract] Shapley picks most stable solution: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [abstract] No distribution assumptions: "scTriangulate makes no assumption about the distributions of raw underlying values."
- [methods] concat multimodal AnnData: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other"
- [methods] pruning and reclassification: "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined)"
- [methods] winning fraction cutoff: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [readme] README Shapley + stability metrics: "scTriangulate leverages cooperative game theory (Shapley Value) in conjunction with complimentary stability metrics (i.e. reassign score, TFIDF score and SCCAF score)"
- [readme] README adata.raw caveat: [evidence span withheld — anchor status: external_doc]

---
name: umap-embedding-and-signature-score-visualization
description: Use when per-cell signature scores have been computed (e.g., UCell scores
  stored in altExp(sce,'UCell') or Seurat metadata) and you need to visualize signature
  enrichment across cells in 2D to interpret clusters.
when_to_use_negative:
- No dimensionality reduction or signature scores exist yet — scores must first be
  computed with ScoreSignatures_UCell and a UMAP must be computable from the data
- Data are bulk or pseudo-bulk samples rather than individual cells — per-cell UMAP
  projection of scores is not meaningful
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: Computes per-cell Mann-Whitney U-based signature scores that are visualized
    on the UMAP; provides SmoothKNN for kNN score smoothing on SingleCellExperiment
    or Seurat objects
  repo: https://github.com/carmonalab/UCell
- name: SingleCellExperiment
  role: Container for counts assay, UCell altExp scores, and the UMAP embedding used
    in the vignette-style plotting workflow
- name: R
  role: Implementation language for the UCell scoring, UMAP computation, and figure
    generation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:20ac509837edf410cb50248412c9def2882ce0259d7cddfbfd3f75ae199587c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# umap-embedding-and-signature-score-visualization

## Summary

Compute a UMAP low-dimensional embedding of single-cell data and project per-cell gene signature scores (e.g., UCell Mann-Whitney U-based scores bounded in [0,1]) onto the embedding so clusters can be interpreted in terms of signature enrichment.

## When to use

Use when per-cell signature scores have been computed (e.g., UCell scores stored in altExp(sce,'UCell') or Seurat metadata) and you need to visualize signature enrichment across cells in 2D to interpret clusters.

## When NOT to use

- No dimensionality reduction or signature scores exist yet — scores must first be computed with ScoreSignatures_UCell and a UMAP must be computable from the data
- Data are bulk or pseudo-bulk samples rather than individual cells — per-cell UMAP projection of scores is not meaningful

## Inputs

- SingleCellExperiment object with a 'counts' assay
- Per-cell signature score table returned by UCell::ScoreSignatures_UCell (stored in altExp(sce, 'UCell'))
- Named list of gene signatures (e.g., Tcell, Myeloid, NK, Plasma_cell)

## Outputs

- UMAP embedding of the cells
- UMAP figure colored by per-cell signature scores (ucell_signature_umap.png)
- Signature score table for all signatures with values in [0,1]

## How to apply

After running UCell::ScoreSignatures_UCell on a SingleCellExperiment (assay='counts', default rank cap rmax=1500), retrieve the normalized U scores for all signatures from the altExp object named 'UCell'. Compute a UMAP embedding of the cells (from the cell-to-cell distance structure in the expression data or an existing dimensionality reduction) as in the SingleCellExperiment vignette. Plot each cell on the UMAP colored by its per-cell signature score, so that clusters can be interpreted in terms of signature enrichment in low-dimensional space. Optionally apply SmoothKNN smoothing of scores over k nearest neighbors in a reduction to partially correct for sparsity. Save the signature score table and the UMAP figure (e.g., ucell_signature_umap.png).

## Related tools

- **UCell** (Computes per-cell Mann-Whitney U-based signature scores that are visualized on the UMAP; provides SmoothKNN for kNN score smoothing on SingleCellExperiment or Seurat objects) — https://github.com/carmonalab/UCell
- **SingleCellExperiment** (Container for counts assay, UCell altExp scores, and the UMAP embedding used in the vignette-style plotting workflow)
- **R** (Implementation language for the UCell scoring, UMAP computation, and figure generation)

## Examples

```
sce <- UCell::ScoreSignatures_UCell(sce, features=gene.sets, assay='counts'); plotUMAP(sce, colour_by='Tcell') # scores in altExp(sce,'UCell'); save ucell_signature_umap.png
```

## Evaluation signals

- The 'UCell' altExp contains score columns for all signatures (e.g., Tcell, Myeloid, NK, Plasma_cell)
- All signature scores are normalized U statistics bounded in [0,1]
- The UMAP figure file (ucell_signature_umap.png) exists
- Each cell in the UMAP plot carries a finite signature score colored on the embedding
- Clusters on the UMAP show coherent signature enrichment consistent with expected cell-type biology

## Limitations

- UCell scores are rank-based and bounded by the rmax=1500 rank cap, which mitigates but does not eliminate the uninformative tail of zero-expression genes
- Single-cell data are sparse, so raw signature scores may benefit from SmoothKNN imputation over k nearest neighbors in a dimensionality reduction
- Since UCell v2.7.6 the normalization Umax = n*maxRank - n(n+1)/2 differs from earlier implementations, so scores may not be directly comparable across package versions

## Evidence

- [results] UCell helps interpreting clusters in terms of signature enrichment in low-dimensional spaces such as the UMAP: "UCell helps interpreting clusters in terms of signature enrichment in low-dimensional spaces such as the UMAP"
- [readme] The new function SmoothKNN performs smoothing of single-cell signature scores by weighted average of the k-nearest neighbors in a given dimensionality reduction: [evidence span withheld — anchor status: external_doc]
- [methods] we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"

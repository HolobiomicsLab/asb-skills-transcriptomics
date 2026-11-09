---
name: attention-matrix-normalization
description: Use when you have run TOSICA.pre on query data and obtained the attention
  matrix (new_adata.X) that you intend to feed into scIB metric computation, PCA,
  nearest-neighbor graph construction, or UMAP visualization.
when_to_use_negative:
- Input is a raw gene-expression count matrix rather than a TOSICA attention matrix
  — standard scanpy normalization applies there, not this attention-matrix post-processing
  step.
- Query genes have not been reordered to match the reference (query_adata = query_adata[:,
  ref_adata.var_names]); inconsistent var_names make TOSICA.pre output invalid before
  normalization is meaningful.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TOSICA
  role: Produces the attention matrix (new_adata.X) via TOSICA.pre that is then library-size
    normalized
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Used to load the reference/query AnnData and perform the library-size normalization
    (default size factor 10,000), PCA, nearest-neighbor graph, and UMAP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:693d5ba124214a413105c645d1be3d167097d03750ddcb68e9505dc21972c3ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# attention-matrix-normalization

## Summary

Post-processing step in the TOSICA label-transfer workflow that converts the raw attention matrix produced by TOSICA.pre (new_adata.X) into a cell-embedding space suitable for downstream benchmarking and visualization by applying library-size normalization with a default size factor of 10,000.

## When to use

Use when you have run TOSICA.pre on query data and obtained the attention matrix (new_adata.X) that you intend to feed into scIB metric computation, PCA, nearest-neighbor graph construction, or UMAP visualization.

## When NOT to use

- Input is a raw gene-expression count matrix rather than a TOSICA attention matrix — standard scanpy normalization applies there, not this attention-matrix post-processing step.
- Query genes have not been reordered to match the reference (query_adata = query_adata[:, ref_adata.var_names]); inconsistent var_names make TOSICA.pre output invalid before normalization is meaningful.

## Inputs

- AnnData object returned by TOSICA.pre with new_adata.X = attention matrix, new_adata.obs['Prediction'], new_adata.obs['Probability'], new_adata.var['pathway_index']

## Outputs

- Library-size-normalized attention matrix (default size factor 10,000) suitable for scIB metric computation
- PCA / nearest-neighbor graph / UMAP embeddings derived from the normalized matrix

## How to apply

After TOSICA.pre returns new_adata whose .X holds the attention matrix over pathway tokens, normalize the matrix by library-size correction using the default size factor of 10,000. This normalization places TOSICA's attention-based embeddings on a comparable scale with cell embeddings from the other integration/label-transfer methods so that the 8 scIB metrics (combined as overall score = 0.6 × biology conservation + 0.4 × batch removal) are computed fairly. Normalized embeddings can then be used directly for downstream manifold steps: PCA, nearest-neighbor graph construction, and UMAP. In the pan-cancer myeloid benchmark (71,159 cells; KIDNEY/UCEC/ESCA reference vs six-cancer-type query), this normalized matrix was the basis for TOSICA's overall scIB score, on which it ranked second of 11 applicable methods (Seurat-based methods inapplicable for >85 batches).

## Related tools

- **TOSICA** (Produces the attention matrix (new_adata.X) via TOSICA.pre that is then library-size normalized) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Used to load the reference/query AnnData and perform the library-size normalization (default size factor 10,000), PCA, nearest-neighbor graph, and UMAP)

## Examples

```
new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], model_weight_path='./my_project/model-n.pth', project='my_project'); sc.pp.normalize_total(new_adata, target_sum=1e4)
```

## Evaluation signals

- After normalization, cell library sizes of the attention matrix are scaled to the default size factor 10,000.
- The normalized matrix retains one row per query cell and one column per pathway token (pathway_index metadata unchanged).
- Normalized embeddings support the 8 scIB metrics; overall score = 0.6 × biology conservation + 0.4 × batch removal is computed per method and TOSICA should rank among the top methods (second of 11 in the pan-cancer myeloid benchmark).
- Downstream PCA/UMAP on the normalized matrix should preserve query cell-type structure and show batch-insensitive mixing consistent with TOSICA's reported behavior.

## Limitations

- The attention matrix is meaningful only if ref_adata and query_adata var_names are consistent and in the same order; the README explicitly warns to run query_adata = query_adata[:, ref_adata.var_names].
- Normalization uses the default size factor 10,000; deviating from this default may break comparability with the benchmarked integration methods.
- Number of attention heads (H) is not specified in the methods, so attention-matrix dimensionality cannot be independently verified from the paper.

## Evidence

- [discussion] Normalize attention matrix by library-size correction and run PCA, nearest neighbor graph, UMAP: "the matrix is normalized by library-size correction using default size factor 10,000"
- [readme] Attention matrix output of TOSICA.pre: "+ `new_adata.X` : Attention matrix"
- [readme] Gene order consistency requirement: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [results] Benchmark ranking result grounding the scIB score combination: [evidence span withheld — anchor status: not_found]

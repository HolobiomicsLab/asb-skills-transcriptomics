---
name: scanpy-gene-scoring-equivalence
description: Use when re-implementing or auditing SenePy's score_hub on an AnnData
  object and you need a scanpy.score_genes-equivalent configuration (binarize=False,
  importance=False) as a ground-truth comparison for per-cell signature scores.
when_to_use_negative:
- When the goal is the default SenePy senescence score itself — binarize=True and
  importance weighting should stay enabled, not be disabled to mimic scanpy
- When the AnnData contains scaled (z-scored) expression — score_hub explicitly requires
  non-scaled data
- When scoring against a gene set with no SenePy hub or importance weights (e.g. SenMayo
  gene lists), where plain scanpy.score_genes is the appropriate tool
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SenePy
  role: Provides score_hub with binarize/importance options and the hub signatures
    being scored and cross-checked
  repo: https://github.com/jaleesr/senepy
- name: scanpy
  role: Reference gene-scoring implementation (score_genes) that the binarize=False,
    importance=False configuration of score_hub is designed to mimic
- name: AnnData
  role: Container format for the single-cell expression matrix scored in-place (adata.obs['sp_score'])
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:e5f7b29f9c33cffe2a88261fed76993b6f0b3526295d04fa7a5cd64cb3bb1ce8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# scanpy-gene-scoring-equivalence

## Summary

Cross-validate the SenePy score_hub senescence-scoring implementation against scanpy.score_genes by running SenePy with binarize=False and importance=False, which reproduces default scanpy gene-scoring behavior and provides a reference point for verifying re-implementations of per-cell signature scoring.

## When to use

Use when re-implementing or auditing SenePy's score_hub on an AnnData object and you need a scanpy.score_genes-equivalent configuration (binarize=False, importance=False) as a ground-truth comparison for per-cell signature scores.

## When NOT to use

- When the goal is the default SenePy senescence score itself — binarize=True and importance weighting should stay enabled, not be disabled to mimic scanpy
- When the AnnData contains scaled (z-scored) expression — score_hub explicitly requires non-scaled data
- When scoring against a gene set with no SenePy hub or importance weights (e.g. SenMayo gene lists), where plain scanpy.score_genes is the appropriate tool

## Inputs

- Single-cell gene expression AnnData object (unscaled; raw or normalized both acceptable when binarize=True)
- SenePy hub signature gene list, e.g. hubs.hubs[('Lung', 'fibroblast', 1)] from sp.load_hubs(species='Mouse')

## Outputs

- Per-cell senescence scores stored in adata.obs (e.g. adata.obs['sp_score'])
- Comparative score vectors from the reconstructed algorithm, sp.score_hub, and scanpy.score_genes-mimicking configuration
- Outlier-cell classification comparison and exported per-cell scores (e.g. senescence_scores.csv)

## How to apply

Load the hub signature via sp.load_hubs(species='Mouse') and hubs.get_genes((tissue, cell, hub_num)), e.g. ('Lung', 'fibroblast', 1), on an unscaled AnnData object. Run sp.score_hub with default settings (binarize=True, importance weighting by gene connection count in the hub network), where binarization makes normalization and transformation optional. Then run the equivalence configuration: normalize_total followed by log1p on the AnnData, and call sp.score_hub(adata, signature, binarize=False, importance=False) to mimic default scanpy.score_genes scoring. Compare the scores from the reconstructed algorithm, the direct sp.score_hub call, and the scanpy-equivalent configuration; validate that outlier-cell classifications agree before writing per-cell scores (e.g. to senescence_scores.csv).

## Related tools

- **SenePy** (Provides score_hub with binarize/importance options and the hub signatures being scored and cross-checked) — https://github.com/jaleesr/senepy
- **scanpy** (Reference gene-scoring implementation (score_genes) that the binarize=False, importance=False configuration of score_hub is designed to mimic)
- **AnnData** (Container format for the single-cell expression matrix scored in-place (adata.obs['sp_score']))

## Examples

```
adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)], binarize=False, importance=False)
```

## Evaluation signals

- Scores from the re-implemented algorithm match sp.score_hub output on the same AnnData and signature
- binarize=False, importance=False output is consistent with scanpy.score_genes scoring on normalized, log1p-transformed data
- Outlier-cell classifications derived from the reconstructed scores agree with the reference scored dataset
- Expression matrix remains unscaled (no sc.pp.scale) before scoring, per score_hub requirements
- Per-cell scores are finite, correctly indexed to adata.obs, and exportable to senescence_scores.csv without index misalignment

## Limitations

- With default settings (binarize=True) normalization and transformation do not matter, but the data must not be scaled
- Universal signatures from universal_signatures/*.txt lack importance weights; weights must be obtained via hubs.merge_hubs()
- Downregulated genes in senescent cells are not accounted for — no negative markers exist in the signatures
- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging

## Evidence

- [readme] scanpy equivalence configuration: "We can mimic default scanpy.score_genes by removing both of these parameters"
- [readme] score_hub defaults and unscaled requirement: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [methods] importance weighting rationale: "it weighs the importance of each gene based on the number of connections that gene has in the hub network"
- [methods] binarize default: "By default, it binarizes the data, which makes normalization and transformation optional."
- [methods] signature extraction: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"

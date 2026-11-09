---
name: outlier-cell-classification-agreement-analysis
description: Use when independently re-implementing SenePy score_hub and needing to
  verify that your per-cell senescence scores classify the same outlier cells as the
  reference package output.
when_to_use_negative:
- When only a generic gene-set enrichment score is needed without background correction
  — scanpy.score_genes alone (binarize=False, importance=False) suffices and the outlier-agreement
  validation is unnecessary
- When the AnnData object is already scaled (z-scored) — SenePy scoring requires unscaled
  data and results would be invalid
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs, get_genes, and the reference score_hub whose output the
    re-implementation must agree with
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the single-cell expression matrix and obs-level score
    storage
- name: scanpy
  role: score_genes run with binarize=False and importance=False as a baseline scoring
    comparison
- name: Python
  role: Implementation language for the re-implemented score_hub and validation logic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:2085d38d1e4cbcc1f7241d598156519768cbd1f2a57646d94f6c1bae7eede0ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# outlier-cell-classification-agreement-analysis

## Summary

Validates a re-implementation of the SenePy score_hub algorithm by comparing which cells are classified as senescent outliers against the reference scored dataset, ensuring binarization, importance weighting, and expression-bin background sampling reproduce the official per-cell scores.

## When to use

Use when independently re-implementing SenePy score_hub and needing to verify that your per-cell senescence scores classify the same outlier cells as the reference package output.

## When NOT to use

- When only a generic gene-set enrichment score is needed without background correction — scanpy.score_genes alone (binarize=False, importance=False) suffices and the outlier-agreement validation is unnecessary
- When the AnnData object is already scaled (z-scored) — SenePy scoring requires unscaled data and results would be invalid

## Inputs

- Single-cell gene expression AnnData object (unscaled; normalization optional because default scoring binarizes data)
- SenePy hub signature gene list (e.g. hubs.get_genes(('Lung', 'fibroblast', 1)))
- Reference per-cell scores / scored dataset for agreement comparison

## Outputs

- Per-cell senescence scores for the re-implementation (e.g. senescence_scores.csv)
- Outlier-cell classification sets from reconstructed vs. reference scores
- Agreement comparison of outlier-cell classifications between implementations

## How to apply

Load the single-cell AnnData object and retrieve the senescence signature with sp.load_hubs(species='Mouse'), extracting genes via hubs.get_genes((tissue, cell, hub_num)). Re-implement score_hub by binning all measured genes by mean expression level, sampling nctrl_size background genes matched per expression bin, binarizing the expression data (making normalization optional but requiring unscaled data), and weighting each signature gene by its number of connections in the hub network. Compute the per-cell score as the weighted signature expression minus the expected background expression from sampled control genes (Methods Eq. 11-15). Cross-check by calling sp.score_hub directly with defaults, and additionally run scanpy.score_genes with binarize=False and importance=False to mimic default scanpy scoring. Validate by comparing outlier-cell classifications from the reconstructed scores against the reference scored dataset; agreement in the outlier-cell sets indicates a faithful re-implementation.

## Related tools

- **SenePy** (Provides load_hubs, get_genes, and the reference score_hub whose output the re-implementation must agree with) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the single-cell expression matrix and obs-level score storage)
- **scanpy** (score_genes run with binarize=False and importance=False as a baseline scoring comparison)
- **Python** (Implementation language for the re-implemented score_hub and validation logic)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Outlier-cell classifications from the reconstructed scores match those from the reference scored dataset
- Direct call of sp.score_hub on the same AnnData and signature reproduces the re-implemented scores
- scanpy.score_genes with binarize=False, importance=False yields the expected baseline pattern mimicking default scanpy scoring
- Per-cell scores are written completely to senescence_scores.csv with one score per cell
- Background sampling respects the expression bins: control genes are matched to signature-gene expression levels within each bin

## Limitations

- Default score_hub binarizes data and weights genes by hub-network connection counts; deviating from these defaults changes scores and invalidates agreement with the reference
- senePy does not account for genes downregulated in senescent cells (no negative markers)
- Universal signatures from universal_signatures/ carry no importance weights; weights must be obtained via merge_hubs()
- High dropout and limited sequencing depth of single-cell data limit detection of classical markers like Cdkn2a and Cdkn1a, affecting score stability

## Evidence

- [methods] Cross-check reconstruction against direct package call: "We can mimic default scanpy.score_genes by removing both of these parameters"
- [readme] Default binarization and importance weighting: "By default, it binarizes the data, which makes normalization and transformation optional."
- [methods] Background gene matching by expression bins: [evidence span withheld — anchor status: not_found]
- [readme] Signature extraction entry point: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [readme] Scoring integration with AnnData: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"

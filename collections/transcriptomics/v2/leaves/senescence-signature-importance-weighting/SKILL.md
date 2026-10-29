---
name: senescence-signature-importance-weighting
description: Use when scoring single-cell transcriptomes with a SenePy senescence
  hub signature and you want per-gene contributions scaled by hub-network connectivity
  rather than treating all signature genes equally.
when_to_use_negative:
- When you deliberately want scanpy.score_genes-equivalent behavior (equal gene weighting
  on continuous, normalized, log-transformed expression) — then pass binarize=False,
  importance=False instead.
- When using the Universal_mouse.txt / Universal_human.txt universal signature files,
  which carry no importance weights; obtain weighted genes from merge_hubs instead.
- When the AnnData matrix has been scaled (z-scored), which violates score_hub's input
  expectation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides score_hub, which applies the per-gene importance weighting by hub-network
    connection count (importance=True by default) and binarizes the expression data
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the single-cell expression matrix being scored; scores are written
    back into adata.obs
- name: scanpy
  role: Alternative reference scoring behavior (score_genes) that can be mimicked
    by disabling binarize and importance; also used to normalize/log-transform data
    when those defaults are turned off
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:c9574e823ccf06167afdcd925e883ad26a38fe5eef5f21e64522f5ffcea25b09
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
yes_rate: 0.7
n_claims: 10
---

# senescence-signature-importance-weighting

## Summary

Weights each gene in a SenePy cellular senescence signature by its number of connections in the hub gene network, so that highly connected hub genes contribute more to per-cell senescence scores. This is the default behavior of senePy's score_hub and distinguishes SenePy scoring from plain scanpy.score_genes.

## When to use

Use when scoring single-cell transcriptomes with a SenePy senescence hub signature and you want per-gene contributions scaled by hub-network connectivity rather than treating all signature genes equally.

## When NOT to use

- When you deliberately want scanpy.score_genes-equivalent behavior (equal gene weighting on continuous, normalized, log-transformed expression) — then pass binarize=False, importance=False instead.
- When using the Universal_mouse.txt / Universal_human.txt universal signature files, which carry no importance weights; obtain weighted genes from merge_hubs instead.
- When the AnnData matrix has been scaled (z-scored), which violates score_hub's input expectation.

## Inputs

- AnnData single-cell gene expression matrix (.h5ad), raw or normalized but NOT scaled
- SenePy hub signature (list of genes from hubs.get_genes or hubs.hubs), e.g. ('Lung', 'fibroblast', 1)
- hubs.metadata dataframe with tissue, cell_type, hub_num, size, n_sen, hyp fields

## Outputs

- Per-cell senescence score column appended to adata.obs (e.g. adata.obs['sp_score'])
- Optionally, gene importance weights implied by hub network connection counts embedded in the score

## How to apply

Load the signature hub object with sp.load_hubs for the correct species (Human or Mouse) and extract the desired signature via hubs.get_genes((tissue, cell_type, hub_num)). Call sp.score_hub on a non-scaled AnnData object, leaving importance=True (the default) so each gene is weighted by its connection count in the hub network. Keep binarize=True (default) as well — binarization makes normalization and log-transformation optional since raw or normalized inputs both work; the only requirement is the data are not scaled. If you want to mimic scanpy.score_genes behavior instead, explicitly pass binarize=False, importance=False after running scanpy.pp.normalize_total and sc.pp.log1p. When building merged signatures (e.g., via merge_hubs with calculate_thresh=True, p_thres=0.01), note that universal-signature gene lists from the universal_signatures .txt files carry no importance weights — retrieve weights via merge_hubs if weighting is required.

## Related tools

- **SenePy** (Provides score_hub, which applies the per-gene importance weighting by hub-network connection count (importance=True by default) and binarizes the expression data) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the single-cell expression matrix being scored; scores are written back into adata.obs)
- **scanpy** (Alternative reference scoring behavior (score_genes) that can be mimicked by disabling binarize and importance; also used to normalize/log-transform data when those defaults are turned off)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Human'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- adata.obs contains a new numeric score column (e.g. 'sp_score') with one value per cell and no NaNs
- Score is computed on non-scaled data; verify max(abs(adata.X)) is not consistent with z-scored values before scoring
- Differential check: rerun with importance=False and binarize=False after normalize_total + log1p and confirm results resemble scanpy.score_genes output, confirming the weighting is the only difference
- Signature genes passed to score_hub match hubs.get_genes((tissue, cell_type, hub_num)) exactly (e.g. Lung-fibroblast-1)
- Hubs used carry expected metadata fields (tissue, cell_type, hub_num, size, n_sen, hyp) so weights derive from a valid hub network

## Limitations

- High dropout and limited sequencing depth in single-cell data make classical CS markers like Cdkn2a and Cdkn1a unreliable as sole indicators — hence reliance on weighted multi-gene hubs rather than single markers.
- The weighting scheme counts only positive connections in the hub network; negative gene-gene correlations were not computed because exhaustive pairwise correlation was computationally limiting.
- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging, and downregulated genes in senescent cells are not accounted for (no negative markers).
- Universal signature .txt files lack importance weights; weights are only available from load_hubs or merge_hubs outputs.

## Evidence

- [readme] score_hub weighs genes by hub-network connections: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network"
- [readme] score_hub default binarization makes preprocessing optional: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] mimicking scanpy.score_genes by removing both parameters: "We can mimic default scanpy.score_genes by removing both of these parameters"
- [readme] universal signature files have no weights: "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs():"
- [abstract] 64 human weighted signatures underpin the scoring platform: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."

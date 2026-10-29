---
name: mean-expression-binned-background-gene-sampling
description: Use when scoring single-cell transcriptomes (AnnData) against a gene
  signature and you need an expression-matched background control instead of naive
  whole-transcriptome averaging, e.g. re-implementing or validating SenePy score_hub.
when_to_use_negative:
- You want default scanpy.score_genes behavior without expression-matched binning
  — then call score_hub with binarize=False and importance=False on normalized, log1p-transformed
  data instead.
- Your input matrix is scaled (z-scored) — score_hub requires unscaled data even though
  binarization makes normalization optional.
- You need signatures that account for genes downregulated in senescent cells — the
  method accounts only for positively enriched (no negative) markers.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SenePy
  role: 'Provides load_hubs/get_genes/score_hub: the hub database, signature extraction,
    and the reference scoring implementation being reproduced'
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the single-cell expression matrix being scored; scores are stored
    in adata.obs
- name: scanpy
  role: score_genes (with binarize=False, importance=False) serves as the baseline
    cross-check for the re-implementation
- name: Python 3.9
  role: Runtime environment for the senepy package (conda env senepy)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:fc78f8629ae240eb9cc219d067225a16dac6d6924635c257bf96a09e4bad3041
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - Python 3.9
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mean-expression-binned-background-gene-sampling

## Summary

Background-gene control strategy underlying SenePy's score_hub: measured genes are binned by mean expression level, and expression-matched non-signature control genes are sampled within each bin so per-cell senescence scores reflect signature expression above expression-matched background expectations (Methods Eq. 11-15).

## When to use

Use when scoring single-cell transcriptomes (AnnData) against a gene signature and you need an expression-matched background control instead of naive whole-transcriptome averaging, e.g. re-implementing or validating SenePy score_hub.

## When NOT to use

- You want default scanpy.score_genes behavior without expression-matched binning — then call score_hub with binarize=False and importance=False on normalized, log1p-transformed data instead.
- Your input matrix is scaled (z-scored) — score_hub requires unscaled data even though binarization makes normalization optional.
- You need signatures that account for genes downregulated in senescent cells — the method accounts only for positively enriched (no negative) markers.

## Inputs

- Single-cell gene-expression AnnData object (raw or normalized, unscaled)
- Senescence signature gene list from senePy hubs (e.g. hubs.get_genes(('Lung', 'fibroblast', 1)))
- Background control size parameter nctrl_size

## Outputs

- Per-cell senescence scores (one value per cell), written e.g. to senescence_scores.csv or adata.obs['sp_score']
- Senescent-outlier cell classifications derived from score thresholds

## How to apply

Load the senescence hub with sp.load_hubs (e.g. species='Mouse') and extract genes via hubs.get_genes((tissue, cell, hub_num)). Bin all measured genes in the AnnData object by mean expression level, then within each bin sample a background set of nctrl_size non-signature genes matched to the binned expression so controls span the same dynamic range as the signature. Apply score_hub's default transformations: binarize the expression data (which makes normalization and transformation optional, as long as the matrix is not scaled) and weight each signature gene by its number of connections in the hub network. Compute the per-cell score as the weighted signature expression minus the expected background expression from the sampled control genes, following Methods Eq. 11-15. Cross-check the re-implementation by also calling sp.score_hub directly and by running scanpy.score_genes with binarize=False and importance=False to mimic default scanpy scoring.

## Related tools

- **SenePy** (Provides load_hubs/get_genes/score_hub: the hub database, signature extraction, and the reference scoring implementation being reproduced) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the single-cell expression matrix being scored; scores are stored in adata.obs)
- **scanpy** (score_genes (with binarize=False, importance=False) serves as the baseline cross-check for the re-implementation)
- **Python 3.9** (Runtime environment for the senepy package (conda env senepy))

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Per-cell scores from the re-implementation correlate with sp.score_hub output on the same AnnData and hub signature
- Outlier-cell classifications from reconstructed scores match those from the reference scored dataset
- score_hub run with binarize=False, importance=False reproduces scanpy.score_genes results on normalized, log1p-transformed data
- Sampled control genes are expression-matched: each signature gene's bin contains nctrl_size background genes of comparable mean expression
- Final scores written to senescence_scores.csv contain one row per cell with no missing values

## Limitations

- Binarization (the default) discards expression magnitude, so scores depend only on detection, not expression level.
- Data must not be scaled; score_hub with default settings tolerates raw or normalized input, but not z-scored matrices.
- Genes that are part of the CS program but overlap the transcriptional shift with age are discounted, and downregulated genes in senescent cells are not accounted for (no negative markers).
- High dropout and limited sequencing depth of single-cell data make classical markers like Cdkn2a and Cdkn1a unreliable as sole indicators, which is partly why hub-based scoring is used.
- Universal signatures from universal_signatures/Universal_mouse.txt / Universal_human.txt lack importance weights; weights must be obtained via merge_hubs().

## Evidence

- [methods] score_hub binarization and importance weighting: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] importance weighting by hub connectivity: "it weighs the importance of each gene based on the number of connections that gene has in the hub network"
- [readme] default scoring usage and unscaled-data requirement: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
- [readme] scanpy baseline cross-check: "We can mimic default scanpy.score_genes by removing both of these parameters"
- [discussion] no negative markers limitation: "We also do not account for genes that are downregulated in senescent cells."
- [discussion] dropout challenge motivating signature scoring: "the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators"

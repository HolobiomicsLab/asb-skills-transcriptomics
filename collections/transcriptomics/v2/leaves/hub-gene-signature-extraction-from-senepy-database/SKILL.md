---
name: hub-gene-signature-extraction-from-senepy-database
description: Use when a workflow needs a mouse or human cell-type-specific senescence
  gene signature (e.g., tongue keratinocyte type-A/type-B hubs) from SenePy to score
  single-cell transcriptomes.
when_to_use_negative:
- You need a truly universal senescence marker panel with weights — merge all hubs
  via merge_hubs instead of extracting a single cell-type hub
- The data are not single-cell gene expression (AnnData) or the species has no matching
  tissue/cell-type hub in SenePy metadata
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs/get_genes/merge_hubs to browse, filter, and extract weighted
    senescence hub gene signatures
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for single-cell expression data that the extracted signatures
    will later be scored against
- name: scanpy
  role: Single-cell ecosystem used alongside SenePy for loading and preprocessing
    the AnnData object
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:467f05b0d5b13f0389cd2ce76e5667c7c4328739c25d40b78e8c0445f588c887
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hub-gene-signature-extraction-from-senepy-database

## Summary

Extracts cell-type-specific senescence hub gene signatures from the SenePy database using sp.load_hubs and hubs.get_genes(tissue, cell, hub_num), yielding weighted gene lists for downstream single-cell senescence scoring.

## When to use

Use when a workflow needs a mouse or human cell-type-specific senescence gene signature (e.g., tongue keratinocyte type-A/type-B hubs) from SenePy to score single-cell transcriptomes.

## When NOT to use

- You need a truly universal senescence marker panel with weights — merge all hubs via merge_hubs instead of extracting a single cell-type hub
- The data are not single-cell gene expression (AnnData) or the species has no matching tissue/cell-type hub in SenePy metadata

## Inputs

- SenePy hub database (bundled with senepy; 75 mouse and 65 human signatures from 97 unique cell types)
- Hub selector tuple (tissue, cell type, hub_num), e.g. ('Tongue', 'keratinocyte', 0)
- hubs.metadata dataframe for browsing/filtering available hubs

## Outputs

- List of hub genes for the selected signature (with hub-network connection counts as importance weights)
- Optionally, merged signatures (e.g., tissue-specific or 'Universal') from merge_hubs

## How to apply

Install senepy (pip install senepy, optionally in a conda env with Python 3.9) and initialize the hub object with sp.load_hubs(species='Mouse') or 'Human'; hubs.metadata lists each hub's tissue, cell type, hub_num, size, n_sen, and hyp enrichment p-value. Retrieve a specific signature with hubs.get_genes((tissue, cell, hub_num)), e.g. ('Tongue', 'keratinocyte', 0) and 1 for the type-A/type-B tongue keratinocyte hubs. Signatures carry importance weights equal to each gene's number of connections in the hub network, which score_hub uses by default; if a merged or universal signature is needed, use hubs.merge_hubs with an overlap_threshold or calculate_thresh=True (permutation-based, p_thres=0.01). The extracted list is passed directly to sp.score_hub(adata, sig) on an AnnData object, where default binarization makes normalization optional as long as data are not scaled.

## Related tools

- **SenePy** (Provides load_hubs/get_genes/merge_hubs to browse, filter, and extract weighted senescence hub gene signatures) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for single-cell expression data that the extracted signatures will later be scored against)
- **scanpy** (Single-cell ecosystem used alongside SenePy for loading and preprocessing the AnnData object)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); sig = hubs.get_genes(('Tongue', 'keratinocyte', 0))
```

## Evaluation signals

- hubs.metadata contains the requested tissue/cell/hub_num row and hub size, n_sen, and hyp enrichment p-value are populated
- hubs.get_genes returns a non-empty gene list whose members match the selected hub in hubs.hubs[(tissue, cell, hub_num)]
- Genes in the signature match the species (Mouse genes like Cdkn2a/Cdkn1a, not human orthologs)
- If merge_hubs was used, each gene's value equals the number of input signatures in which it occurs, honoring overlap_threshold or the permutation-based calculate_thresh (p_thres=0.01)

## Limitations

- No importance weights are associated with the precomputed Universal_mouse.txt / Universal_human.txt signature lists; weights must come from .merge_hubs()
- Large overlap_threshold values drastically reduce gene counts when merging only a handful of signatures
- SenePy does not account for genes downregulated in senescent cells (no negative markers), and its methodology discounts CS genes overlapping with the organismal aging transcriptional shift

## Evidence

- [readme] Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse.: "Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse."
- [methods] get_genes extracts tissue/cell/hub_num signature: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [readme] score_hub weights genes by hub-network connections: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [readme] merge_hubs overlap_threshold reduces genes: "This number will drastically reduce the number of genes in the resulting signature if only merging a handful of input signatures."
- [readme] Universal signature lists lack importance weights: "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs():"

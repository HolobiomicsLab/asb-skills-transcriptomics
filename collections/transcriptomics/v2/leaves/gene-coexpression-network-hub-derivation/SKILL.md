---
name: gene-coexpression-network-hub-derivation
description: Use when deriving cell-type-specific senescence (or age-dynamic) gene
  signatures from aggregated single-cell RNA-seq spanning multiple tissues and ages,
  requiring weighted hub gene lists rather than simple marker lists.
when_to_use_negative:
- When the goal is only to score existing cells against already-published signatures
  — use SenePy's load_hubs/score_hub directly instead of re-deriving hubs
- When the transcriptomic data lack an age axis or multiple tissues, since the age-dynamic
  Equations 4-6 and cross-tissue hub merging require tissue-and-age-resolved data
- When a true universal signature without importance weights suffices — use the provided
  Universal_mouse.txt / Universal_human.txt lists instead
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs, get_genes, merge_hubs, search_hubs_by_genes, and score_hub
    functions; merge_hubs with calculate_thresh performs the permutation-based merge
    (5% threshold, FDR p < 0.01) that finalizes signatures; score_hub weights genes
    by hub-network connection count by default
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the aggregated single-cell RNA-seq object used in signature
    derivation and scoring
- name: scanpy
  role: Preprocessing (e.g., normalize_total, log1p) of the AnnData object; alternative
    unweighted scoring mode in score_hub mimics scanpy.score_genes
- name: Python 3.9
  role: Runtime environment for the signature-derivation workflow (conda create -n
    senepy python=3.9)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:843b6deb0764b69a3827fd1efc8a2cace2d89b494a7d7486473771d0e768a05b
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# gene-coexpression-network-hub-derivation

## Summary

Algorithmic derivation of cell-type-specific cellular senescence (CS) signatures by computing age-dynamic differential expression scores, building gene-gene co-expression hub networks, weighting hub genes by connection count, and merging redundant hubs — the method behind SenePy's 64 human and 72 mouse weighted signatures.

## When to use

Use when deriving cell-type-specific senescence (or age-dynamic) gene signatures from aggregated single-cell RNA-seq spanning multiple tissues and ages, requiring weighted hub gene lists rather than simple marker lists.

## When NOT to use

- When the goal is only to score existing cells against already-published signatures — use SenePy's load_hubs/score_hub directly instead of re-deriving hubs
- When the transcriptomic data lack an age axis or multiple tissues, since the age-dynamic Equations 4-6 and cross-tissue hub merging require tissue-and-age-resolved data
- When a true universal signature without importance weights suffices — use the provided Universal_mouse.txt / Universal_human.txt lists instead

## Inputs

- Aggregated single-cell RNA-seq AnnData object spanning multiple tissues and ages (human: 37 tissues, ages 1-92; mouse: Tabula Muris Senis, 328K cells, 19 tissues, 1-30 months)
- Cell-type and tissue annotations per cell
- 10-year age bin assignments for each cell

## Outputs

- Weighted cell-type-specific CS signature gene lists (64 human and 72 mouse signatures), genes weighted by hub-network connection count
- Signature metadata dataframe with fields: tissue, cell type, hub_num, size, n_sen, and enrichment p-value (hyp)
- Saved hub object (e.g., human_hubs.pkl)

## How to apply

Load an aggregated human or mouse single-cell RNA-seq AnnData object (human: 37 tissues, ages 1-92; mouse: Tabula Muris Senis, 19 tissues, 1-30 months) and aggregate cells by tissue, cell type, and 10-year age bins. Compute age-dynamic differential expression scores per cell type using the paper's Equations 4-6 to identify genes whose expression tracks senescence across ages. Build gene-gene co-expression networks from these age-dynamic scores and identify densely connected hub clusters per cell type; weight each hub gene by its number of connections in the hub network (senePy's default importance weighting), recording metadata (tissue, cell type, hub_num, size, n_sen, hyp). Merge redundant hubs per cell type with merge_hubs using a permutation-based threshold (calculate_thresh=True, ~5% random chance, FDR p < 0.01) to finalize signatures, then save the weighted gene lists and metadata (e.g., human_hubs.pkl). The connection-count weighting rationale is that genes with more network connections are more central to the senescence program, and the permutation threshold removes genes occurring below random chance.

## Related tools

- **SenePy** (Provides load_hubs, get_genes, merge_hubs, search_hubs_by_genes, and score_hub functions; merge_hubs with calculate_thresh performs the permutation-based merge (5% threshold, FDR p < 0.01) that finalizes signatures; score_hub weights genes by hub-network connection count by default) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the aggregated single-cell RNA-seq object used in signature derivation and scoring)
- **scanpy** (Preprocessing (e.g., normalize_total, log1p) of the AnnData object; alternative unweighted scoring mode in score_hub mimics scanpy.score_genes)
- **Python 3.9** (Runtime environment for the signature-derivation workflow (conda create -n senepy python=3.9))

## Examples

```
hubs = sp.load_hubs(species='Human'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal'); sig = hubs.get_genes(('Lung', 'fibroblast', 1))
```

## Evaluation signals

- Running the algorithm on the human data yields exactly 64 human putative CS cell-type signatures (72 for mouse)
- Each hub's metadata contains tissue, cell type, hub_num, size, n_sen, and enrichment p-value (hyp) fields
- Signature genes carry importance weights equal to their number of connections in the hub network
- Merged hubs discard genes occurring below the permutation-based threshold (~5% random chance, FDR p < 0.01)
- Extracted signatures (hubs.get_genes(('Lung', 'fibroblast', 1))) return the expected gene list for a given tissue-cell-hub_num triple

## Limitations

- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging
- Downregulated genes in senescent cells are not accounted for (no negative markers)
- Finding negative correlations in all pairwise gene combinations was computationally limiting
- Merged universal signatures from merge_hubs lack importance weights unless taken from merge_hubs output directly
- The overlap_threshold in merge_hubs drastically reduces gene count if merging only a handful of input signatures

## Evidence

- [abstract] Equations 4-6 age-dynamic scores and hub derivation: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [readme] Merge threshold: "We then use a 5% random chance threshold and remove all the genes that occur less than that number of times."
- [readme] Importance weighting rationale: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [readme] Hub metadata fields: "size: number of genes in the hub
- n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub."
- [discussion] Aging-overlap limitation: "this discounts genes that may be part of the CS program which overlap with the transcriptional shift with age"
- [discussion] No negative markers: "We also do not account for genes that are downregulated in senescent cells."

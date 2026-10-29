---
name: single-cell-rnaseq-age-binned-aggregation
description: Use when you have an aggregated multi-tissue, multi-age single-cell RNA-seq
  AnnData object and need to algorithmically derive cell-type-specific signatures
  of an age-associated cellular state (e.g., senescence) from age-dynamic expression
  changes.
when_to_use_negative:
- The goal is scoring existing cells for senescence with already-published signatures
  — use load_hubs + score_hub instead of re-deriving hubs
- You need a universal pan-tissue signature only — call merge_hubs over hubs.metadata
  with calculate_thresh rather than re-running the full derivation
- Your dataset lacks an age/time axis, so age-dynamic differential expression scores
  (Equations 4-6) cannot be computed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs, merge_hubs (with overlap_threshold and permutation-based
    calculate_thresh), and importance-weighted hub storage for finalizing signatures
  repo: https://github.com/jaleesr/senepy
- name: Python 3.9
  role: Runtime environment (conda senepy environment) in which the aggregation and
    hub derivation is executed
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the aggregated single-cell expression matrix that the derivation
    pipeline consumes
- name: scanpy
  role: Preprocessing of the AnnData object (e.g., normalize_total, log1p) prior to
    hub derivation and downstream scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:6df734855b63b1d7e27966d0024ab79624f334badf5e8f19b9b8ce788f9dd4b5
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-rnaseq-age-binned-aggregation

## Summary

Derives cell-type-specific cellular senescence (CS) signatures by aggregating single-cell RNA-seq cells by tissue, cell type, and 10-year age bins, computing age-dynamic differential expression scores (paper's Equations 4-6), and building gene-gene co-expression hub networks from those scores. This is the algorithmic basis of the 64 human and 72 mouse SenePy signatures.

## When to use

Use when you have an aggregated multi-tissue, multi-age single-cell RNA-seq AnnData object and need to algorithmically derive cell-type-specific signatures of an age-associated cellular state (e.g., senescence) from age-dynamic expression changes.

## When NOT to use

- The goal is scoring existing cells for senescence with already-published signatures — use load_hubs + score_hub instead of re-deriving hubs
- You need a universal pan-tissue signature only — call merge_hubs over hubs.metadata with calculate_thresh rather than re-running the full derivation
- Your dataset lacks an age/time axis, so age-dynamic differential expression scores (Equations 4-6) cannot be computed

## Inputs

- Aggregated single-cell RNA-seq AnnData object covering multiple tissues and ages (e.g., human data across 37 tissues, ages 1-92; mouse Tabula Muris Senis, 328K cells, 19 tissues, 1-30 months)
- Cell-type and age annotations (tissue, cell type, 10-year age bins)
- Curated list of prior known senescence markers (180 literature-curated markers; SenMayo union, n=125) for enrichment validation

## Outputs

- Putative cell-type-specific CS gene signatures: weighted gene lists per tissue/cell type/hub_num (e.g., 64 human signatures, 72 mouse signatures)
- Hub metadata dataframe with fields tissue, cell type, hub_num, size, n_sen, and enrichment p-value (hyp)
- Serialized hub object (e.g., human_hubs.pkl) loadable via sp.load_hubs for scoring with score_hub

## How to apply

Load the aggregated single-cell RNA-seq AnnData object (e.g., human data spanning 37 tissues and ages 1-92) into Python with scanpy/AnnData. Aggregate cells by tissue, cell type, and 10-year age bins, then compute age-dynamic differential expression scores per cell type using the paper's Equations 4-6 to find genes whose expression tracks senescence across ages. For each cell type, build gene-gene co-expression networks from the age-dynamic scores and identify densely connected hub gene clusters; weight each hub gene by its number of connections in the hub network (SenePy's default importance weighting). Merge redundant hubs per cell type using senePy merge_hubs with a permutation-based threshold (calculate_thresh=True, ~5% random chance, FDR p < 0.01) to finalize signatures, and save the resulting hubs and metadata (tissue, cell type, hub_num, size, n_sen, enrichment p-value) as a pickle (e.g., human_hubs.pkl).

## Related tools

- **SenePy** (Provides load_hubs, merge_hubs (with overlap_threshold and permutation-based calculate_thresh), and importance-weighted hub storage for finalizing signatures) — https://github.com/jaleesr/senepy
- **Python 3.9** (Runtime environment (conda senepy environment) in which the aggregation and hub derivation is executed) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the aggregated single-cell expression matrix that the derivation pipeline consumes)
- **scanpy** (Preprocessing of the AnnData object (e.g., normalize_total, log1p) prior to hub derivation and downstream scoring)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Human'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal')
```

## Evaluation signals

- Exactly 64 putative human CS cell-type signatures (and 72 mouse signatures) are produced when run on the full human/mouse atlases
- Every hub's metadata contains tissue, cell type, hub_num, size, n_sen, and enrichment p-value fields
- Hub genes are importance-weighted by their number of connections in the hub network (default score_hub importance weighting)
- Merged hubs use the permutation-based threshold at a ~5% random chance with FDR p < 0.01, matching the universal-signature procedure
- Extracted signatures are retrievable via hubs.get_genes((tissue, cell_type, hub_num)) and contain the expected gene lists

## Limitations

- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging
- Downregulated genes in senescent cells are not accounted for (no negative markers)
- Finding negative correlations in all pairwise gene combinations was computationally limiting
- Cell-specific heterogeneity of CS and the high dropout rate / limited sequencing depth of single-cell data (classical markers Cdkn2a/Cdkn1a poorly detectable) constrain signature generality

## Evidence

- [other] Running the signature-derivation algorithm on the human data yielded 64 putative human CS cell-type signatures in total (alongside 72 mouse signatures).: [evidence span withheld — anchor status: not_found]
- [abstract] We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform.: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [intro] In this study, we aggregate and interrogate large-scale single-cell RNA-sequencing datasets across tissues and ages in mice and humans to define in vivo CS heterogeneity.: "we aggregate and interrogate large-scale single-cell RNA-sequencing datasets across tissues and ages in mice and humans to define in vivo CS heterogeneity"
- [readme] By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network.: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [discussion] this discounts genes that may be part of the CS program which overlap with the transcriptional shift with age: "this discounts genes that may be part of the CS program which overlap with the transcriptional shift with age"

---
name: cross-species-senescence-signature-comparison
description: Use when the goal is to derive or validate species-level 'universal'
  senescence gene panels from SenePy's 72 mouse / 64 human cell-type signatures, or
  to compare a derived merged panel against the distributed Universal_mouse.txt /
  Universal_human.txt references.
when_to_use_negative:
- The goal is tissue-specific (e.g., Lung-only) merging with a hard overlap_threshold
  rather than a species-wide universal panel with a permutation-based threshold.
- The input is a raw single-cell AnnData to be scored for senescence — use score_hub
  on an existing signature instead of re-deriving one.
- Only the gene lists without weights are needed and the distributed Universal_mouse.txt
  / Universal_human.txt files already suffice.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs to retrieve species-specific cell-type signature hubs and
    merge_hubs with calculate_thresh/p_thres to compute the permutation-based overrepresentation
    threshold and gene occurrence-count weights
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Execution environment for the SenePy package (installed via pip install senepy
    in a Python 3.9 conda environment)
  repo: https://github.com/jaleesr/senepy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:f202a68c2338a6d3c8cef04e380611a6879d4806fdd192836e9a27f74f5dbc27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - Python
  - SenePy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-species-senescence-signature-comparison

## Summary

Reproduces SenePy's universal senescence gene panels by merging all cell-type-specific signatures per species with a permutation-based overrepresentation threshold (FDR p-value < 0.01), yielding 635 universal mouse and 734 universal human genes.

## When to use

Use when the goal is to derive or validate species-level 'universal' senescence gene panels from SenePy's 72 mouse / 64 human cell-type signatures, or to compare a derived merged panel against the distributed Universal_mouse.txt / Universal_human.txt references.

## When NOT to use

- The goal is tissue-specific (e.g., Lung-only) merging with a hard overlap_threshold rather than a species-wide universal panel with a permutation-based threshold.
- The input is a raw single-cell AnnData to be scored for senescence — use score_hub on an existing signature instead of re-deriving one.
- Only the gene lists without weights are needed and the distributed Universal_mouse.txt / Universal_human.txt files already suffice.

## Inputs

- SenePy hub object with hubs.metadata dataframe (tissue, cell, hub_num, size, n_sen, hyp) for species='Mouse', sig_type='cell_type'
- SenePy hub object with hubs.metadata for species='Human'
- Distributed reference files universal_signatures/Universal_mouse.txt and universal_signatures/Universal_human.txt (for validation only)

## Outputs

- universal_mouse_panel.txt — 635 universal mouse genes with occurrence-count importance weights
- universal_human_panel.txt — 734 universal human genes with occurrence-count importance weights
- Merged 'Universal' signature stored in hubs.hubs['Universal']

## How to apply

Load the signature hubs with sp.load_hubs(species='Mouse', sig_type='cell_type') and read hubs.metadata, which lists tissue, cell type, hub_num, size, n_sen (curated-marker count), and hyp enrichment p-value for each hub. Merge all mouse signatures in one call via hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal'); merge_hubs takes the union of genes, weighs each gene by its occurrence count across input signatures (importance weight), and, when calculate_thresh=True, computes a permutation-based occurrence threshold and retains genes significantly overrepresented at FDR p-value < 0.01. Export the merged signature (genes plus occurrence-count weights) as universal_mouse_panel.txt, then repeat the identical procedure with sp.load_hubs(species='Human') to produce universal_human_panel.txt. Validate by confirming the mouse panel contains 635 genes and the human panel 734 genes, matching the distributed universal_signatures/Universal_mouse.txt and Universal_human.txt reference files (which lack importance weights; weights come only from merge_hubs).

## Related tools

- **SenePy** (Provides load_hubs to retrieve species-specific cell-type signature hubs and merge_hubs with calculate_thresh/p_thres to compute the permutation-based overrepresentation threshold and gene occurrence-count weights) — https://github.com/jaleesr/senepy
- **Python** (Execution environment for the SenePy package (installed via pip install senepy in a Python 3.9 conda environment)) — https://github.com/jaleesr/senepy

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse', sig_type='cell_type'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal')
```

## Evaluation signals

- Recovered mouse panel contains exactly 635 genes, matching universal_signatures/Universal_mouse.txt
- Recovered human panel contains exactly 734 genes, matching universal_signatures/Universal_human.txt
- Each gene in the merged 'Universal' signature occurs at or above the permutation-derived occurrence threshold and passes FDR p-value < 0.01 (p_thres=0.01)
- Each retained gene carries an integer occurrence-count importance weight reflecting how many input signatures contain it
- Gene members are drawn from the union of all cell-type hub signatures (9645-gene union for mouse), not from an arbitrary subset

## Limitations

- The distributed Universal_mouse.txt / Universal_human.txt files carry no importance weights; weights are only obtainable via merge_hubs.
- merge_hubs uses a 5% random chance threshold by default; the universal panels require explicitly setting calculate_thresh=True with p_thres=0.01.
- The approach retains only overrepresented genes and does not account for genes downregulated in senescent cells or genes overlapping the organismal-aging transcriptional shift.
- Setting overlap_threshold too high drastically reduces the number of genes when merging only a handful of signatures.

## Evidence

- [readme] Universal signatures overrepresentation threshold: "These are lists of genes that are overrepresented in the senePy signatures (FDR p-value < 0.01), i.e., genes that are more universal across cell types."
- [readme] merge_hubs canonical invocation: "hubs = sp.load_hubs(species = 'Mouse', sig_type = 'cell_type')
result = hubs.merge_hubs(hubs.metadata, calculate_thresh = True, p_thres = 0.01, new_name = 'Universal')"
- [readme] permutation-based threshold and occurrence counting: "We can also calculate a threshold automatically using a permutation-based method. In the example below, we merge all signatures from all tissues and cells."
- [readme] gene weighting by occurrence count: "By default, the union of all genes is taken. The respective value for each gene is the number of times that gene is found in the input signatures."
- [abstract] finding on universal signature derivation: "Here we developed an analytical algorithm that defines cell-type-specific and universal signatures of cellular senescence across a wide range of cell types and tissues."

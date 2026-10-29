---
name: permutation-based-signature-thresholding
description: Use when merging multiple cell-type-specific senescence hub signatures
  (e.g., all human or mouse hubs in hubs.metadata) into a universal signature and
  you need an automatic, statistically grounded overlap_threshold instead of hand-picking
  one.
when_to_use_negative:
- When merging only a handful of signatures, where a fixed overlap_threshold of 2
  already removes most genes and permutation-based thresholding would strip the merged
  signature to near-empty
- When you want the precomputed universal signature files (Universal_mouse.txt / Universal_human.txt),
  which lack importance weights and require no merging step
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SenePy
  role: Provides the merge_hubs function with the calculate_thresh permutation-based
    thresholding used to set the gene overlap cutoff when merging senescence signatures
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Runtime environment (Python 3.9 via conda) in which senePy and its merge_hubs
    permutation routine are executed
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Format of the single-cell expression objects that the derived merged signatures
    are used to score after thresholding
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:986a50f16be30ac86c5e6d7e9658fafebe73c24f4a485841a51d38f4c26571ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# permutation-based-signature-thresholding

## Summary

A technique from SenePy that uses a permutation-based method to automatically set the gene overlap threshold when merging senescence hub signatures, retaining only genes that occur more often than a 5% random chance expectation.

## When to use

Use when merging multiple cell-type-specific senescence hub signatures (e.g., all human or mouse hubs in hubs.metadata) into a universal signature and you need an automatic, statistically grounded overlap_threshold instead of hand-picking one.

## When NOT to use

- When merging only a handful of signatures, where a fixed overlap_threshold of 2 already removes most genes and permutation-based thresholding would strip the merged signature to near-empty
- When you want the precomputed universal signature files (Universal_mouse.txt / Universal_human.txt), which lack importance weights and require no merging step

## Inputs

- hubs.metadata dataframe of senescence hub signatures (tissue, cell type, hub_num, size, n_sen, hyp columns) from sp.load_hubs
- Optional manual overlap_threshold integer (e.g., 2) if not using the permutation-based calculation

## Outputs

- A merged named signature (e.g., 'universal', 'Lung_merged') in hubs.hubs with genes weighted by occurrence count across input signatures
- Updated hubs.metadata including the new merged hub entry

## How to apply

Load the hub object with sp.load_hubs(species='Human' or 'Mouse'), then call hubs.merge_hubs on the metadata subset to be merged with calculate_thresh=True. SenePy computes, via permutation, the overlap frequency expected by chance (a 5% random chance level) and removes genes that occur less than that number of times in the input signatures. Genes retained get an importance weight equal to their occurrence count across the input signatures. Use p_thres=0.01 (FDR < 0.01) when deriving universal signatures, matching the paper's derivation of 64 human and 72 mouse signatures. Hand-set overlap_threshold (e.g., 2) is an alternative but drastically reduces gene counts when only a few signatures are merged.

## Related tools

- **SenePy** (Provides the merge_hubs function with the calculate_thresh permutation-based thresholding used to set the gene overlap cutoff when merging senescence signatures) — https://github.com/jaleesr/senepy
- **Python** (Runtime environment (Python 3.9 via conda) in which senePy and its merge_hubs permutation routine are executed) — https://github.com/jaleesr/senepy
- **AnnData** (Format of the single-cell expression objects that the derived merged signatures are used to score after thresholding)

## Examples

```
hubs = sp.load_hubs(species='Human', sig_type='cell_type'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal')
```

## Evaluation signals

- With calculate_thresh=True, no merged gene should occur fewer times than the 5% random-chance permutation threshold
- Each retained gene's value in the merged hub equals the number of input signatures in which it appears (occurrence-count weighting)
- Deriving universal human signatures with p_thres=0.01 should reproduce gene sets consistent with the published universal_signatures/Universal_human.txt
- The merged hub appears in hubs.metadata with the new_name supplied (e.g., 'universal') and retrievable via hubs.hubs[new_name]
- Signature counts remain consistent with the paper's totals (64 human and 72 mouse weighted signatures) when applied in the full atlas workflow

## Limitations

- The permutation threshold is tuned to a 5% random chance level; using it when merging few input signatures drastically reduces the number of genes in the resulting signature
- Genes removed by the threshold may still be part of the CS program — the paper's methodology discounts genes overlapping the transcriptional shift of organismal aging and excludes downregulated genes
- Universal signature text files derived this way carry no importance weights; weights must be obtained through merge_hubs

## Evidence

- [abstract] abstract: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [readme] readme: "We can also calculate a threshold automatically using a permutation-based method"

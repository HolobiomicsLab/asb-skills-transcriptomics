---
name: overrepresentation-enrichment-testing
description: Use when deriving a universal senescence gene panel from many cell-type-specific
  signatures and you need a statistical (rather than arbitrary overlap-count) cutoff
  for which genes are overrepresented across signatures.
when_to_use_negative:
- When merging only a handful of input signatures (e.g. one tissue's Lung hubs), where
  a manual overlap_threshold is more appropriate than the permutation-based threshold
- When you need cell-type- or tissue-specific signatures rather than genes universal
  across all 97 cell types
- When the input is already a universal signature list (e.g. universal_signatures/Universal_mouse.txt),
  which contains genes but no importance weights
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SenePy
  role: Provides load_hubs and merge_hubs (with calculate_thresh, permutation-based
    thresholding and p_thres=0.01) used to test gene overrepresentation across senescence
    signatures and output the universal panel
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Language in which SenePy is implemented and invoked to run the overrepresentation
    test
  repo: https://github.com/jaleesr/senepy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:1147ab80770b954b4a485fc84bf2205e125b70a8ffe4e191ad24846ecdc1fefd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - Python
  - SenePy
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# overrepresentation-enrichment-testing

## Summary

Identifies genes statistically overrepresented across a collection of SenePy cell-type senescence signatures by merging hubs with a permutation-based occurrence threshold, retaining genes significant at FDR p-value < 0.01 to form universal mouse (635 genes) and human (734 genes) panels.

## When to use

Use when deriving a universal senescence gene panel from many cell-type-specific signatures and you need a statistical (rather than arbitrary overlap-count) cutoff for which genes are overrepresented across signatures.

## When NOT to use

- When merging only a handful of input signatures (e.g. one tissue's Lung hubs), where a manual overlap_threshold is more appropriate than the permutation-based threshold
- When you need cell-type- or tissue-specific signatures rather than genes universal across all 97 cell types
- When the input is already a universal signature list (e.g. universal_signatures/Universal_mouse.txt), which contains genes but no importance weights

## Inputs

- SenePy hubs metadata dataframe (hubs.metadata) for a species, listing all cell-type senescence signature hubs (tissue, cell type, hub_num, size, n_sen, hyp)
- Optional manual overlap_threshold integer for signature-occurrence cutoffs

## Outputs

- Merged universal signature (dictionary of genes with occurrence-count importance weights), e.g. hubs.hubs['Universal']
- Universal_mouse_panel.txt with 635 overrepresented mouse genes (FDR p-value < 0.01)
- Universal_human_panel.txt with 734 overrepresented human genes (FDR p-value < 0.01)

## How to apply

Load the signature hubs with sp.load_hubs (e.g., species='Mouse', sig_type='cell_type') and take hubs.metadata as the input signature collection. Call hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal'), which merges the union of all genes (9645 union genes in the mouse case), computes a permutation-based occurrence threshold, and retains only genes whose occurrence across signatures is significantly overrepresented at FDR p-value < 0.01. Each retained gene carries an importance weight equal to its occurrence count (number of input signatures containing it). Export the merged signature and verify it against the distributed reference files universal_signatures/Universal_mouse.txt and universal_signatures/Universal_human.txt, expecting 635 mouse and 734 human genes. Use an FDR-adjusted p-value rather than a raw occurrence cutoff so that the threshold scales to the number of input signatures; only use a manual overlap_threshold when merging a small number of signatures where the permutation threshold is not informative.

## Related tools

- **SenePy** (Provides load_hubs and merge_hubs (with calculate_thresh, permutation-based thresholding and p_thres=0.01) used to test gene overrepresentation across senescence signatures and output the universal panel) — https://github.com/jaleesr/senepy
- **Python** (Language in which SenePy is implemented and invoked to run the overrepresentation test) — https://github.com/jaleesr/senepy

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse', sig_type='cell_type'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal')
```

## Evaluation signals

- The recovered universal mouse panel contains exactly 635 genes, matching universal_signatures/Universal_mouse.txt
- The recovered universal human panel contains exactly 734 genes, matching universal_signatures/Universal_human.txt
- All retained genes satisfy FDR p-value < 0.01 under the permutation-based threshold (p_thres=0.01)
- Each retained gene's weight equals its occurrence count across the input signatures (the number of hubs containing it)
- The union of all input mouse signatures totals 9645 genes before overrepresentation filtering, which should shrink to 635 after filtering

## Limitations

- The permutation-based threshold requires a sufficiently large number of input signatures; merging a handful of hubs with overlap thresholds drastically reduces the resulting gene set
- The distributed universal_signatures files contain no importance weights; weights must be recomputed via merge_hubs
- SenePy's approach discounts genes that overlap with the transcriptional shift of organismal aging and does not account for downregulated genes (no negative markers) in senescent cells
- Threshold is FDR-controlled at p-value < 0.01, so weakly shared genes that fail significance are excluded from the universal panel

## Evidence

- [full_text] 635 universal mouse genes / 734 human genes via FDR < 0.01: [evidence span withheld — anchor status: not_found]
- [readme] README universal signatures definition: "These are lists of genes that are overrepresented in the senePy signatures (FDR p-value < 0.01), i.e., genes that are more universal across cell types."
- [readme] README merge_hubs permutation threshold: "We can also calculate a threshold automatically using a permutation-based method."
- [readme] README merge_hubs occurrence weights: "By default, the union of all genes is taken. The respective value for each gene is the number of times that gene is found in the input signatures."
- [readme] README no weights in distributed files: "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs():"

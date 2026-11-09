---
name: gene-signature-merging
description: Use when you have multiple cell-type- or tissue-specific gene signatures
  (e.g., SenePy hub signatures) and need to derive a consolidated universal or tissue-level
  signature with statistically filtered, weighted genes.
when_to_use_negative:
- You only need the precomputed universal gene lists without importance weights —
  use the distributed universal_signatures/Universal_mouse.txt or Universal_human.txt
  files directly instead of merging.
- You want to score cells with a single existing signature (use sp.score_hub rather
  than merging).
- The merge involves only a handful of input signatures and you rely on calculate_thresh
  — the permutation threshold can drastically reduce the gene set; specify overlap_threshold
  manually in that case.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs to initialize hub signatures and metadata, and merge_hubs
    to take the union of signatures, compute a permutation-based occurrence threshold,
    and retain overrepresented genes with occurrence-count weights.
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Runtime environment (Python 3.9 via conda; pip install senepy) for executing
    the SenePy merge workflow and exporting signature files.
  repo: https://github.com/jaleesr/senepy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:52e75eaec87c600f69d0fe3351ced19647707fb53baeb1f2e0718f133106fea6
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
yes_rate: 0.6
n_claims: 5
---

# gene-signature-merging

## Summary

Merge multiple cell-type-specific senescence gene signatures from the SenePy hub database into a single universal or tissue-specific signature, using a permutation-based occurrence threshold to retain only statistically overrepresented genes (FDR p-value < 0.01) and importance weights equal to each gene's occurrence count across input signatures.

## When to use

Use when you have multiple cell-type- or tissue-specific gene signatures (e.g., SenePy hub signatures) and need to derive a consolidated universal or tissue-level signature with statistically filtered, weighted genes.

## When NOT to use

- You only need the precomputed universal gene lists without importance weights — use the distributed universal_signatures/Universal_mouse.txt or Universal_human.txt files directly instead of merging.
- You want to score cells with a single existing signature (use sp.score_hub rather than merging).
- The merge involves only a handful of input signatures and you rely on calculate_thresh — the permutation threshold can drastically reduce the gene set; specify overlap_threshold manually in that case.

## Inputs

- SenePy hub object with metadata dataframe of signatures (tissue, cell type, hub_num) from sp.load_hubs(species='Human' or 'Mouse', sig_type='cell_type')
- Threshold parameters: p_thres=0.01 (FDR) or an integer overlap_threshold (e.g., 2)
- Optional subset of hubs.metadata rows to restrict the merge to a tissue (e.g., hubs.metadata[hubs.metadata.tissue == 'Lung'])

## Outputs

- Merged signature stored in hubs.hubs[new_name] with genes weighted by occurrence count across input signatures
- Exported universal signature panels, e.g., universal_mouse_panel.txt (635 genes) and universal_human_panel.txt (734 genes)

## How to apply

Load the hub object with sp.load_hubs(species='Mouse' or 'Human', sig_type='cell_type') and inspect hubs.metadata, which lists each hub's tissue, cell type, hub_num, size, and enrichment p-value. Merge signatures with hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal'), which takes the union of all genes, computes a permutation-based occurrence threshold, and keeps only genes significantly overrepresented across signatures (FDR p-value < 0.01); for small merges of a handful of signatures, set overlap_threshold manually (e.g., 2) since automatic thresholds can drastically shrink output. The resulting signature (hubs.hubs['Universal']) stores each gene weighted by the number of input signatures in which it occurs, which can be exported (e.g., universal_mouse_panel.txt) and compared against the distributed reference files universal_signatures/Universal_mouse.txt (635 genes) and Universal_human.txt (734 genes) to verify correctness.

## Related tools

- **SenePy** (Provides load_hubs to initialize hub signatures and metadata, and merge_hubs to take the union of signatures, compute a permutation-based occurrence threshold, and retain overrepresented genes with occurrence-count weights.) — https://github.com/jaleesr/senepy
- **Python** (Runtime environment (Python 3.9 via conda; pip install senepy) for executing the SenePy merge workflow and exporting signature files.) — https://github.com/jaleesr/senepy

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse', sig_type='cell_type'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal')
```

## Evaluation signals

- The merged mouse 'Universal' signature contains 635 genes that are significantly overrepresented (FDR p-value < 0.01) out of the 9645-gene union.
- The merged human 'Universal' signature contains 734 genes matching the reference universal_signatures/Universal_human.txt.
- Each gene in the merged signature carries an importance weight equal to its occurrence count across the input signatures.
- Retrieved panels match the distributed reference files Universal_mouse.txt and Universal_human.txt on gene-set comparison.
- Lowering p_thres from 0.01 or removing calculate_thresh changes the retained gene count, confirming the threshold filter is active.

## Limitations

- The distributed universal signature files (Universal_mouse.txt, Universal_human.txt) have no importance weights; weights must be regenerated via .merge_hubs().
- Setting an overlap_threshold that is too high drastically reduces the number of genes when merging only a handful of input signatures.
- The workflow keeps only overrepresented genes and does not account for genes downregulated in senescent cells (no negative markers).

## Evidence

- [readme] Permutation-based threshold and FDR filtering of merged universal signatures: "These are lists of genes that are overrepresented in the senePy signatures (FDR p-value < 0.01), i.e., genes that are more universal across cell types."
- [readme] merge_hubs call with calculate_thresh and p_thres: "hubs = sp.load_hubs(species = 'Mouse', sig_type = 'cell_type')
result = hubs.merge_hubs(hubs.metadata, calculate_thresh = True, p_thres = 0.01, new_name = 'Universal')"
- [readme] Union of genes with occurrence-count weights: "By default, the union of all genes is taken. The respective value for each gene is the number of times that gene is found in the input signatures."
- [results] 635 universal mouse genes from statistical overrepresentation testing: [evidence span withheld — anchor status: not_found]
- [readme] overlap_threshold behavior for small merges: "This number will drastically reduce the number of genes in the resulting signature if only merging a handful of input signatures."

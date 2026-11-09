---
name: permutation-based-threshold-calculation
description: Use when merging multiple SenePy senescence signatures (e.g., all mouse
  or human cell-type hubs) into a universal signature and you need an automatic, statistically
  grounded overlap threshold instead of an arbitrary overlap_threshold count.
when_to_use_negative:
- You only need the precomputed universal gene lists without importance weights —
  the distributed universal_signatures/Universal_mouse.txt and Universal_human.txt
  files already provide these
- You are merging only a handful of signatures (e.g., Lung tissue only) and a simple
  explicit overlap_threshold (e.g., 2) is sufficient; the permutation threshold is
  designed for merging many signatures across all tissues and cell types
- The input is a single-cell expression matrix to be scored — use sp.score_hub instead
  of merge_hubs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs to initialize the hub object and merge_hubs with calculate_thresh=True
    to perform the permutation-based threshold calculation and gene overrepresentation
    filtering
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Execution environment for the SenePy package (installed via pip or conda,
    python=3.9)
  repo: https://github.com/jaleesr/senepy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:9c5ef527b5c36bc822e102e7cde8451cbc885a85fed3fc985b2aa9576f38e102
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - Python
  - SenePy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# permutation-based-threshold-calculation

## Summary

SenePy's merge_hubs() can automatically compute a permutation-based occurrence threshold so that, when many cell-type senescence signatures are merged, only genes statistically overrepresented across signatures (FDR p-value < 0.01) are retained in the resulting universal signature.

## When to use

Use when merging multiple SenePy senescence signatures (e.g., all mouse or human cell-type hubs) into a universal signature and you need an automatic, statistically grounded overlap threshold instead of an arbitrary overlap_threshold count.

## When NOT to use

- You only need the precomputed universal gene lists without importance weights — the distributed universal_signatures/Universal_mouse.txt and Universal_human.txt files already provide these
- You are merging only a handful of signatures (e.g., Lung tissue only) and a simple explicit overlap_threshold (e.g., 2) is sufficient; the permutation threshold is designed for merging many signatures across all tissues and cell types
- The input is a single-cell expression matrix to be scored — use sp.score_hub instead of merge_hubs

## Inputs

- SenePy hubs.metadata dataframe of cell-type senescence signatures (from sp.load_hubs for Mouse or Human)
- p_thres parameter value (0.01) for the FDR cutoff
- species selection ('Mouse' or 'Human')

## Outputs

- A merged universal signature stored in hubs.hubs under new_name (e.g., 'Universal') containing overrepresented genes with occurrence-count importance weights
- Universal mouse panel of 635 genes and universal human panel of 734 genes (comparable to universal_signatures/Universal_mouse.txt and Universal_human.txt)

## How to apply

Load the species-specific hub object with sp.load_hubs (e.g., species='Mouse', sig_type='cell_type') and pass hubs.metadata into hubs.merge_hubs with calculate_thresh=True, p_thres=0.01, and a new_name such as 'Universal'. The function takes the union of all input signature genes, weighs each gene by the number of input signatures it occurs in, computes a permutation-based threshold corresponding to a random-occurrence expectation, and retains only genes whose occurrence significantly exceeds it at FDR p-value < 0.01. The resulting signature (e.g., 635 genes for mouse, 734 genes for human out of 9645 union genes in mouse) is stored under the new name in hubs.hubs, with each gene's occurrence count serving as an importance weight. Apply the procedure separately per species, then compare the recovered gene lists against the distributed universal_signatures/Universal_mouse.txt and Universal_human.txt reference files to verify correctness.

## Related tools

- **SenePy** (Provides load_hubs to initialize the hub object and merge_hubs with calculate_thresh=True to perform the permutation-based threshold calculation and gene overrepresentation filtering) — https://github.com/jaleesr/senepy
- **Python** (Execution environment for the SenePy package (installed via pip or conda, python=3.9)) — https://github.com/jaleesr/senepy

## Examples

```
hubs = sp.load_hubs(species='Mouse', sig_type='cell_type'); result = hubs.merge_hubs(hubs.metadata, calculate_thresh=True, p_thres=0.01, new_name='Universal')
```

## Evaluation signals

- The merged mouse universal signature contains 635 genes significantly overrepresented at FDR p-value < 0.01 out of 9645 union genes
- The merged human universal signature contains 734 genes
- Recovered panels match the distributed reference files universal_signatures/Universal_mouse.txt and universal_signatures/Universal_human.txt gene-for-gene
- Each retained gene carries an occurrence-count importance weight (number of input signatures it appears in), unlike the weight-less distributed .txt files
- Genes occurring at or below the permutation-derived random-occurrence threshold are excluded from the merged signature

## Limitations

- The distributed universal signature .txt files carry no importance weights; weights must be obtained by rerunning merge_hubs with calculate_thresh=True
- The permutation threshold reflects a random-occurrence expectation (e.g., 5% random chance); merging only a few input signatures can drastically reduce the resulting signature size, so the method is best suited to merging many signatures
- The method uses a p_thres of 0.01 on FDR; changing this cutoff will change the number of retained genes and break comparability with the published 635/734-gene panels

## Evidence

- [methods] calculate_thresh (permutation-based threshold): "We can also calculate a threshold automatically using a permutation-based method"
- [readme] universal signature merging with FDR cutoff: "These are lists of genes that are overrepresented in the senePy signatures (FDR p-value < 0.01), i.e., genes that are more universal across cell types."
- [readme] importance weights via merge_hubs: "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs()"
- [other] expected gene counts for universal panels: [evidence span withheld — anchor status: not_found]
- [readme] gene importance by occurrence count: "By default, the union of all genes is taken. The respective value for each gene is the number of times that gene is found in the input signatures."

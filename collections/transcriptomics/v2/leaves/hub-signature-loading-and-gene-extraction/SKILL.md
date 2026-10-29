---
name: hub-signature-loading-and-gene-extraction
description: Use when a senescence-scoring task needs cell-type-specific gene signatures
  from the SenePy database — e.g. selecting mouse heart hubs to score Visium spots
  or single cells — rather than ad-hoc marker lists like Cdkn2a/Cdkn1a alone.
when_to_use_negative:
- You need a senescence signature with importance weights but plan to use the universal
  signature text files (Universal_mouse.txt / Universal_human.txt) — the README states
  these lists carry no importance weights; derive weights via merge_hubs instead
- Your goal is cell-level senescence scoring itself rather than signature retrieval
  — that is handled by sp.score_hub on an AnnData object, not by load_hubs/get_genes
- You require negative (downregulated) senescence markers — the paper notes SenePy
  does not account for genes downregulated in senescent cells
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs, get_genes, merge_hubs, and search_hubs_by_genes for retrieving
    and combining cell-type-specific senescence hub signatures
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Runtime environment (Python 3.9 via conda) in which the senepy package is
    installed and hub extraction is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:a0fb9697e3e420ea61cb487b15395dd99ca31d6bd8484003173e8363eadbf507
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - Python
  - SenePy
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hub-signature-loading-and-gene-extraction

## Summary

Load the SenePy senescence hub database for a species of interest and extract cell-type-specific gene signatures (hubs) for downstream senescence scoring. Each hub is a weighted gene set derived from in vivo single-cell transcriptomics, with metadata on tissue, cell type, hub number, size, number of curated senescence markers (n_sen), and enrichment p-value (hyp).

## When to use

Use when a senescence-scoring task needs cell-type-specific gene signatures from the SenePy database — e.g. selecting mouse heart hubs to score Visium spots or single cells — rather than ad-hoc marker lists like Cdkn2a/Cdkn1a alone.

## When NOT to use

- You need a senescence signature with importance weights but plan to use the universal signature text files (Universal_mouse.txt / Universal_human.txt) — the README states these lists carry no importance weights; derive weights via merge_hubs instead
- Your goal is cell-level senescence scoring itself rather than signature retrieval — that is handled by sp.score_hub on an AnnData object, not by load_hubs/get_genes
- You require negative (downregulated) senescence markers — the paper notes SenePy does not account for genes downregulated in senescent cells

## Inputs

- No input required to browse/retrieve signatures (database ships with the senepy package)
- Optional: pandas-style metadata filter values (tissue, cell type, hub_num)
- Optional: gene symbol list for hub search (e.g. ['Cdkn2a', 'Cdkn1a', 'Il6', 'Cxcl13'])

## Outputs

- hubs.metadata dataframe (tissue, cell type, hub_num, size, n_sen, hyp enrichment p-value)
- Per-hub gene lists from hubs.get_genes((tissue, cell, hub_num))
- Merged signatures with per-gene importance weights from hubs.merge_hubs (e.g. 'Universal', tissue-merged)
- Dataframe of hub matches and statistics from hubs.search_hubs_by_genes

## How to apply

Initialize the hub object with senepy.load_hubs(species='Mouse' or 'Human'); the returned hubs.metadata dataframe lists each hub's tissue, cell type, hub_num, size, n_sen, and enrichment p-value. Filter metadata to the tissue and cell type of interest (e.g. hubs.metadata[hubs.metadata.tissue == 'Heart']) and extract the gene list for a specific hub with hubs.get_genes((tissue, cell, hub_num)), e.g. hubs.get_genes(('Lung', 'fibroblast', 1)). To build tissue-level or universal signatures from multiple hubs, use hubs.merge_hubs(filt_meta, new_name=...), optionally with overlap_threshold (e.g. 2, keep genes occurring in at least 2 signatures) or calculate_thresh=True with p_thres=0.01 for a permutation-based threshold; merged hub values give per-gene importance weights (number of signatures the gene occurs in). Hubs can also be searched by genes of interest (e.g. Cdkn2a, Cdkn1a, Il6, Cxcl13) via hubs.search_hubs_by_genes. The extracted hub dictionaries (hubs.hubs[key]) feed directly into sp.score_hub on an AnnData object.

## Related tools

- **SenePy** (Provides load_hubs, get_genes, merge_hubs, and search_hubs_by_genes for retrieving and combining cell-type-specific senescence hub signatures) — https://github.com/jaleesr/senepy
- **Python** (Runtime environment (Python 3.9 via conda) in which the senepy package is installed and hub extraction is executed)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); heart_meta = hubs.metadata[hubs.metadata.tissue == 'Heart']; genes = hubs.get_genes(tuple(heart_meta.iloc[0][['tissue','cell','hub_num']]))
```

## Evaluation signals

- hubs.metadata returns a dataframe whose rows include tissue, cell type, hub_num, size, n_sen, and hyp columns; confirm the expected number of hubs for the species (75 mouse / 65 human signatures from 97 cell types per the database description)
- hubs.get_genes((tissue, cell, hub_num)) returns a gene list matching the hub's 'size' count in metadata
- hubs.merge_hubs output values equal the number of input signatures each gene occurs in; with overlap_threshold=2, no gene occurs in fewer than 2 signatures
- Extracted gene symbols are valid for the chosen species (e.g. mouse symbols like Cdkn2a) and, where spot-checked, correspond to curated senescence markers counted in n_sen

## Limitations

- Universal signature text files (Universal_mouse.txt, Universal_human.txt) lack importance weights; weights must be obtained from merge_hubs
- High overlap_threshold on merge_hubs drastically reduces gene counts when merging only a handful of input signatures
- SenePy signatures do not include genes downregulated in senescent cells and discount CS genes overlapping the transcriptional shift of organismal aging

## Evidence

- [readme] Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse.: "Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse."
- [readme] get_genes returns a list of genes for a hub identified by tissue, cell, and hub_num: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [readme] hub metadata columns include size, n_sen, and enrichment p-value: "- size: number of genes in the hub
- n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub."
- [readme] merge_hubs threshold rationale: "This number will drastically reduce the number of genes in the resulting signature if only merging a handful of input signatures."
- [readme] universal signatures lack importance weights: "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs():"
- [methods] database size description: "Database of 75 **mouse** and 65 **human** signatures of cellular senescence from 97 unique cell types"

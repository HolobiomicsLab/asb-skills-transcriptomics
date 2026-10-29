---
name: senescence-marker-enrichment-analysis
description: Use when you have cell-type-specific senescence hub gene networks (e.g.
  from SenePy's Tabula Muris Senis analysis) and need to quantify enrichment of the
  180 literature-curated known CS markers to validate each hub.
when_to_use_negative:
- You want to identify novel senescence genes de novo from raw single-cell data —
  this skill only annotates/validates already-derived hub signatures against a fixed
  180-marker literature panel
- Your gene set is not derived as a co-positivity hub (e.g. a plain differential-expression
  gene list) — the hypergeometric n_sen/hyp annotation is defined for SenePy hub objects
- You need a species other than mouse or human — load_hubs only provides Human and
  Mouse signature databases
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: SenePy
  role: Provides load_hubs, merge_hubs, search_hubs_by_genes, and the hubs.metadata
    fields (n_sen, hyp, size) used to compute and inspect marker enrichment of senescence
    signatures
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Language in which the SenePy package is implemented and invoked (Python 3.9
    conda environment recommended)
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Integration format for scoring signature-bearing hubs against single-cell
    expression data alongside the enrichment analysis
- name: scanpy
  role: Single-cell workflow companion; score_hub defaults can mimic scanpy.score_genes
    when binarize and importance are disabled
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:84179b2397e4ff8ca340ac114a6b2b16f8ab577a50ea287cdb6196ff63b19d33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# senescence-marker-enrichment-analysis

## Summary

Annotate SenePy senescence hub gene signatures with the number of literature-curated known senescence markers they contain (n_sen) and their hypergeometric enrichment p-value (hyp), validating that derived hubs are genuine senescence programs rather than generic age-dynamic gene sets.

## When to use

Use when you have cell-type-specific senescence hub gene networks (e.g. from SenePy's Tabula Muris Senis analysis) and need to quantify enrichment of the 180 literature-curated known CS markers to validate each hub.

## When NOT to use

- You want to identify novel senescence genes de novo from raw single-cell data — this skill only annotates/validates already-derived hub signatures against a fixed 180-marker literature panel
- Your gene set is not derived as a co-positivity hub (e.g. a plain differential-expression gene list) — the hypergeometric n_sen/hyp annotation is defined for SenePy hub objects
- You need a species other than mouse or human — load_hubs only provides Human and Mouse signature databases

## Inputs

- SenePy hub object (hubs.metadata dataframe with tissue, cell type, hub_num, size, n_sen, hyp columns) from sp.load_hubs(species='Mouse'|'Human')
- List of literature-curated known senescence markers (180 prior curated markers)
- Optional: query gene list for search_hubs_by_genes (e.g. Cdkn2a, Cdkn1a, Il6, Cxcl13)
- Optional: filtered hubs.metadata subsets for merge_hubs (e.g. hubs.metadata[hubs.metadata.tissue == 'Lung'])

## Outputs

- Per-hub annotation: n_sen (count of known markers) and hyp (hypergeometric enrichment p-value) in hubs.metadata
- Dataframe of hubs containing genes of interest from search_hubs_by_genes with metadata and statistics
- Merged signatures (e.g. 'Lung_merged', 'universal') with gene connection-count weights and re-evaluated enrichment
- Validated set of 72 mouse (and 64-65 human) cell-type-specific senescence signatures matching Supplementary Data

## How to apply

Load the hub object with sp.load_hubs(species='Mouse' or 'Human'), which exposes hubs.metadata containing the n_sen field (number of the 180 prior literature-curated senescence markers found in the hub), the hyp field (hypergeometric enrichment p-value of known vs novel hub genes), and size (number of genes in the hub). Each hub is keyed by (tissue, cell type, hub_num), e.g. ('Lung', 'fibroblast', 1), and hub_num distinguishes multiple senescence hubs within a single cell type. Use search_hubs_by_genes with markers such as Cdkn2a, Cdkn1a, Il6, and Cxcl13 to find which hubs contain genes of interest, returning a dataframe with relevant metadata and statistics. When merging hubs with merge_hubs, apply the overlap_threshold parameter (e.g. require genes to occur at least 2 times) or a permutation-based calculate_thresh (e.g. p_thres=0.01 for the universal signatures) before re-assessing enrichment. Confirm each candidate hub shows statistically significant enrichment of known markers — this distinguishes senescence hubs from hubs of generic aging-shift genes — and cross-check the derived signature set against the database of 75 mouse and 65 human signatures from 97 unique cell types.

## Related tools

- **SenePy** (Provides load_hubs, merge_hubs, search_hubs_by_genes, and the hubs.metadata fields (n_sen, hyp, size) used to compute and inspect marker enrichment of senescence signatures) — https://github.com/jaleesr/senepy
- **Python** (Language in which the SenePy package is implemented and invoked (Python 3.9 conda environment recommended)) — https://github.com/jaleesr/senepy
- **AnnData** (Integration format for scoring signature-bearing hubs against single-cell expression data alongside the enrichment analysis)
- **scanpy** (Single-cell workflow companion; score_hub defaults can mimic scanpy.score_genes when binarize and importance are disabled)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); result = hubs.search_hubs_by_genes(['Cdkn2a', 'Cdkn1a', 'Il6', 'Cxcl13'])
```

## Evaluation signals

- Exactly 72 mouse putative CS signatures are produced (64-65 human), consistent with the SenePy database of 75 mouse and 65 human signatures from 97 unique cell types
- Each hub's hubs.metadata row carries a non-null n_sen count and a hyp hypergeometric enrichment p-value, with hubs showing statistically significant enrichment of the 180 known markers
- hub metadata resolves to a valid (tissue, cell type, hub_num) key, e.g. ('Lung', 'fibroblast', 1), and get_genes returns the gene list for that hub
- Merged signatures respect the overlap_threshold (genes occurring at least N times) or permutation-based threshold (p_thres=0.01), and gene values equal the number of input signatures containing that gene
- Universal merged signatures match the gene lists distributed in universal_signatures/Universal_mouse.txt and Universal_human.txt (FDR p-value < 0.01)

## Limitations

- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging, so hubs may miss senescence genes shared with general aging programs
- Downregulated genes in senescent cells are not accounted for — there are no negative markers in the signatures
- Finding negative correlations in all pairwise gene combinations was computationally limiting, restricting the network construction underlying the hubs
- Classical CS markers such as Cdkn2a and Cdkn1a suffer from high dropout and limited sequencing depth in single-cell data, so enrichment statistics should not rely on single-marker detection

## Evidence

- [readme] n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub. hyp: enrichment p-value of the "known" and novel hub genes: "n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub. - hyp: enrichment p-value of the "known" and novel hub genes"
- [abstract] We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform.: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [readme] Function to find hubs that contain genes of interest. Returns a dataframe with relevant metadata and statistics.: "Function to find hubs that contain genes of interest. Returns a dataframe with relevant metadata and statistics."
- [methods] hubs.search_hubs_by_genes(['Cdkn2a', 'Cdkn1a', 'Il6', 'Cxcl13']): "hubs.search_hubs_by_genes(['Cdkn2a', 'Cdkn1a', 'Il6', 'Cxcl13'])"
- [discussion] the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators: "the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators"

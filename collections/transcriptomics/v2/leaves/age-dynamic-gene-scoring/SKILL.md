---
name: age-dynamic-gene-scoring
description: Use when you have single-cell RNA-seq data spanning multiple organismal
  ages and need to identify, per cell type, genes whose expression changes with age
  to build cell-type-specific senescence signatures.
when_to_use_negative:
- Your single-cell data contain only one age point or lack replicate age structure,
  since GAD scoring and the age-label permutation null cannot be computed
- You only need to score cells for senescence with existing signatures — use senePy's
  score_hub/load_hubs instead of re-deriving age-dynamic genes
- Your goal is a universal senescence signature from merged existing hubs (merge_hubs
  with calculate_thresh=True) rather than de novo per-cell-type gene discovery
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SenePy
  role: Open-source Python package whose age-dynamic scoring algorithm and downstream
    hub functions (load_hubs, merge_hubs, get_genes, score_hub) consume the significant
    age-dynamic genes; hub metadata fields n_sen and hyp quantify enrichment of the
    180 known markers
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the single-cell expression matrix, per-cell tissue/cell-type/age
    annotations passed to the GAD scoring workflow
  repo: https://github.com/jaleesr/senepy
- name: scanpy
  role: Used to normalize and log-transform AnnData (normalize_total, log1p) when
    scoring beyond the binarized default of score_hub
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:e1dfd1d0bb2768906590114c22fd5dd7989c63fdbedb224a2dc083940a8ed564
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# age-dynamic-gene-scoring

## Summary

Scores candidate senescence-associated genes for age-dynamic (GAD) behavior within each cell type of a single-cell atlas (e.g., Tabula Muris Senis, 328K cells, 19 tissues, ages 1–30 months), testing significance against null permutations of age labels so that genes tracking senescence across ages can seed co-positivity hub construction.

## When to use

Use when you have single-cell RNA-seq data spanning multiple organismal ages and need to identify, per cell type, genes whose expression changes with age to build cell-type-specific senescence signatures.

## When NOT to use

- Your single-cell data contain only one age point or lack replicate age structure, since GAD scoring and the age-label permutation null cannot be computed
- You only need to score cells for senescence with existing signatures — use senePy's score_hub/load_hubs instead of re-deriving age-dynamic genes
- Your goal is a universal senescence signature from merged existing hubs (merge_hubs with calculate_thresh=True) rather than de novo per-cell-type gene discovery

## Inputs

- Single-cell RNA-seq AnnData object spanning multiple ages and tissues (e.g., Tabula Muris Senis: 328K cells, 19 tissues, ages 1–30 months; or aggregated human atlas: 37 tissues, ages 1–92)
- Candidate senescence-associated gene list (180 literature-curated known senescence markers; SenMayo union, n = 125)
- Per-cell annotations: tissue, cell type, and age (or 10-year age bin)

## Outputs

- Per-cell-type tables of age-dynamic (GAD) gene scores with permutation-based significance labels
- Set of significant age-dynamic genes per cell type, ready for co-positivity network / hub construction

## How to apply

Load the single-cell dataset into an AnnData object and aggregate cells by tissue, cell type, and age (e.g., 10-year age bins for the human atlas, 1–30 month ages for Tabula Muris Senis). For each cell type, compute the age-dynamic (GAD) differential expression score for candidate senescence-associated genes using the paper's Equations 4–6, and assess significance against null permutations of age labels rather than raw p-values. Restrict the gene universe to candidate senescence-associated genes (e.g., the 180 literature-curated known markers and the SenMayo union, n = 125) so downstream co-positivity networks are senescence-focused. Keep only significant age-dynamic genes for each cell type as the input edges/genes for the next hub-construction step; genes merely part of the general organismal aging transcriptional shift are handled by the permutation null rather than discarded outright.

## Related tools

- **SenePy** (Open-source Python package whose age-dynamic scoring algorithm and downstream hub functions (load_hubs, merge_hubs, get_genes, score_hub) consume the significant age-dynamic genes; hub metadata fields n_sen and hyp quantify enrichment of the 180 known markers) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the single-cell expression matrix, per-cell tissue/cell-type/age annotations passed to the GAD scoring workflow) — https://github.com/jaleesr/senepy
- **scanpy** (Used to normalize and log-transform AnnData (normalize_total, log1p) when scoring beyond the binarized default of score_hub)

## Examples

```
hubs = sp.load_hubs(species = 'Mouse'); extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num
```

## Evaluation signals

- Applying the full pipeline (GAD scoring + hub construction) to Tabula Muris Senis yields exactly 72 mouse putative CS signatures spanning 43 mouse cell types, consistent with the database of 75 mouse signatures from 97 unique cell types
- The human atlas pipeline (Equations 4–6 + hub merging with calculate_thresh=True, ~5% random chance, FDR p < 0.01) yields exactly 64 human putative CS signatures
- Each derived hub shows statistically significant hypergeometric enrichment (hubs.metadata 'hyp') of the 180 literature-curated known senescence markers, with non-zero 'n_sen' counts
- Permuting age labels within a cell type collapses GAD significance, confirming scores track real age structure rather than batch or cell-count artifacts
- Resulting signature files (e.g., mouse_cs_signatures.csv matching Supplementary Data 4–5; human_hubs.pkl) carry tissue, cell type, hub_num, size, n_sen, and hyp metadata fields

## Limitations

- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging, so aging-general genes are penalized by design
- Downregulated genes in senescent cells are not accounted for — the approach yields no negative markers
- Finding negative correlations in all pairwise gene combinations was computationally limiting, restricting the co-positivity analysis
- High dropout rate and limited sequencing depth make classical CS markers such as Cdkn2a and Cdkn1a unreliable as sole indicators; transient p16 expression combined with dropout yields low Cdkn2a detectability
- Cell-type-specific heterogeneity of CS means no universal signature gene panel fully captures all senescent cell states

## Evidence

- [methods] Compute age-dynamic (GAD) gene scores for candidate senescence-associated genes within each cell type, assessing significance against null permutations of age labels.: [evidence span withheld — anchor status: not_found]
- [methods] Equations 4-6 GAD scores: [evidence span withheld — anchor status: not_found]
- [discussion] Aging-shift discount caveat: "this discounts genes that may be part of the CS program which overlap with the transcriptional shift with age"
- [intro] Tabula Muris Senis dataset description: "For mouse data, we utilized the Tabula Muris Senis22 resource, which is comprised of 328 K cells from 19 tissues collected between 1 and 30 months of age"
- [readme] Hub metadata fields supporting validation: "size: number of genes in the hub - n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub."

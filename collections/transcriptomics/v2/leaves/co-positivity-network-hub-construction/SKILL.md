---
name: co-positivity-network-hub-construction
description: Use when deriving cell-type-specific senescence signatures de novo from
  in vivo single-cell transcriptomic data spanning multiple ages, because classical
  markers (Cdkn2a/Cdkn1a) and differential expression poorly capture in vivo CS heterogeneity.
when_to_use_negative:
- Input is already a curated signature database to be applied for scoring cells —
  use sp.score_hub / SenePy scoring instead of de novo hub construction
- Goal is to detect downregulated genes or negative senescence markers — the algorithm
  explicitly does not account for genes downregulated in senescent cells
- Goal is to capture genes whose signal is part of the general transcriptional shift
  with age — the methodology discounts such genes
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SenePy
  role: Constructs and stores co-positivity hubs, provides hubs.metadata (size, n_sen,
    hyp), get_genes, merge_hubs, and scoring of the derived signatures
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the single-cell Tabula Muris Senis dataset processed
    by the hub-construction workflow
- name: scanpy
  role: Preprocessing (e.g., normalize_total, log1p) of the AnnData object integrated
    with SenePy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:cb2291510760b6c420b2bfb58da862b3ef3c57264b270fc3e440ac955724b269
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# co-positivity-network-hub-construction

## Summary

An algorithmic approach that scores age-dynamic genes per cell type in large single-cell datasets (e.g., Tabula Muris Senis), builds co-positivity gene networks, and extracts densely connected hubs as cell-type-specific cellular senescence (CS) signatures, yielding 72 mouse putative CS signatures across 43 cell types.

## When to use

Use when deriving cell-type-specific senescence signatures de novo from in vivo single-cell transcriptomic data spanning multiple ages, because classical markers (Cdkn2a/Cdkn1a) and differential expression poorly capture in vivo CS heterogeneity.

## When NOT to use

- Input is already a curated signature database to be applied for scoring cells — use sp.score_hub / SenePy scoring instead of de novo hub construction
- Goal is to detect downregulated genes or negative senescence markers — the algorithm explicitly does not account for genes downregulated in senescent cells
- Goal is to capture genes whose signal is part of the general transcriptional shift with age — the methodology discounts such genes

## Inputs

- Single-cell AnnData object of an aging dataset (Tabula Muris Senis: 328K cells, 19 tissues, ages 1–30 months)
- Candidate senescence-associated gene list (e.g., union of SenMayo, n=125, and other curated marker sets)
- 180 literature-curated known senescence markers used for hub enrichment annotation

## Outputs

- mouse_cs_signatures.csv with 72 mouse putative CS signatures (72 signatures across 43 mouse cell types) including tissue, cell type, and hub_num metadata
- hubs.metadata dataframe with hub size, n_sen (count of known markers in hub), and hyp (hypergeometric enrichment p-value)
- Per-hub gene signatures retrievable via hubs.get_genes((tissue, cell, hub_num))

## How to apply

Load the Tabula Muris Senis dataset (328K cells, 19 tissues, ages 1–30 months) into an AnnData object. Within each cell type, compute age-dynamic (GAD) gene scores for candidate senescence-associated genes, testing significance against null permutations of age labels. For significant genes, build per-cell-type co-positivity networks and extract densely connected hubs using SenePy hub functions. Annotate each hub with the count of the 180 literature-curated known senescence markers it contains (hubs.metadata field n_sen) and the hypergeometric enrichment p-value (hyp), then save the resulting 72 mouse cell-type signatures with tissue, cell type, and hub_num metadata (mouse_cs_signatures.csv, matching Supplementary Data 4 and 5).

## Related tools

- **SenePy** (Constructs and stores co-positivity hubs, provides hubs.metadata (size, n_sen, hyp), get_genes, merge_hubs, and scoring of the derived signatures) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the single-cell Tabula Muris Senis dataset processed by the hub-construction workflow)
- **scanpy** (Preprocessing (e.g., normalize_total, log1p) of the AnnData object integrated with SenePy)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1))
```

## Evaluation signals

- Exactly 72 mouse putative CS signatures spanning 43 mouse cell types are produced
- Each hub shows statistically significant hypergeometric enrichment (hyp) of the 180 known senescence markers (n_sen field is non-zero and p-value significant)
- Signature contents and metadata match Supplementary Data 4 and 5
- Resulting database is consistent with the reported 75 mouse and 65 human signatures from 97 unique cell types in the SenePy database
- hubs.metadata rows carry expected fields (tissue, cell type, hub_num, size, n_sen, hyp)

## Limitations

- Genes that overlap with the transcriptional shift of organismal aging are discounted, potentially excluding bona fide CS genes
- Downregulated genes in senescent cells are not accounted for (no negative markers); finding negative correlations in all pairwise gene combinations was computationally limiting
- High dropout rate and limited sequencing depth of single-cell data limit use of classical CS markers such as Cdkn2a and Cdkn1a as sole indicators; transient p16 expression combined with dropout yields low Cdkn2a detectability

## Evidence

- [full_text] Applying an algorithm that scores age-dynamic genes and constructs co-positivity networks with hubs across Tabula Muris Senis mouse cell populations, the authors derived 72 mouse putative CS cell-type signatures: [evidence span withheld — anchor status: not_found]
- [abstract] We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform.: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [readme] Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse.: "Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse."
- [readme] n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub.: "n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub."
- [readme] it weighs the importance of each gene based on the number of connections that gene has in the hub network: "it weighs the importance of each gene based on the number of connections that gene has in the hub network"

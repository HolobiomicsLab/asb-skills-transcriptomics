---
name: evolutionary-index-visualization
description: Use when scRNA-seq AnnData objects need gene age-weighted evolutionary
  index (TEI) scores per cell and stage-wise plots, e.g. to reproduce hourglass patterns
  with lowest TEI at 12 hpf in zebrafish.
when_to_use_negative:
- When the goal is to build the orthomap itself (gene ages per orthogroup) rather
  than apply it to scRNA data — use the of2orthomap/plaza2orthomap/eggnog extraction
  subcommands instead
- When expression data is bulk or non-cell-resolved and no per-cell evolutionary index
  is needed
- When the feature IDs cannot be matched to any orthogroup gene age class (no orthomap
  exists for the query species)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3170
tools:
- name: oggmap
  role: Provides orthomap2tei helpers (get_tei) to match gene age classes to scRNA
    data and calculate/visualize TEI and partial TEI profiles
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Python toolkit used to load and manipulate the scRNA sequencing (AnnData)
    objects into which TEI values are written
- name: gtf2t2g
  role: oggmap subcommand to extract a transcript-to-gene table from a GTF file for
    mapping transcript-level IDs to gene-level IDs
  repo: https://github.com/kullrich/oggmap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:9ac0330b3c265d5361b73e8f59bf3580c7c9ff632a848086357934eecb2d2555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - gtf2t2g
  - oggmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# evolutionary-index-visualization

## Summary

Compute and visualize a gene age-weighted transcriptome evolutionary index (TEI) on single-cell RNA data using oggmap, linking orthomap gene age classes to per-cell expression to reveal evolutionary patterns such as the hourglass model across developmental stages.

## When to use

Use when scRNA-seq AnnData objects need gene age-weighted evolutionary index (TEI) scores per cell and stage-wise plots, e.g. to reproduce hourglass patterns with lowest TEI at 12 hpf in zebrafish.

## When NOT to use

- When the goal is to build the orthomap itself (gene ages per orthogroup) rather than apply it to scRNA data — use the of2orthomap/plaza2orthomap/eggnog extraction subcommands instead
- When expression data is bulk or non-cell-resolved and no per-cell evolutionary index is needed
- When the feature IDs cannot be matched to any orthogroup gene age class (no orthomap exists for the query species)

## Inputs

- AnnData scRNA-seq object (~70,000 zebrafish cells) with transcript-level or gene-level var_names
- orthomap gene age table (gene ages per orthogroup) for the query species
- GTF annotation file (e.g. Danio rerio GRCz11) for the transcript-to-gene table

## Outputs

- AnnData obs column with per-cell TEI values (saved as zebrafish_tei.h5ad)
- Mean TEI per developmental stage summary table
- Stage-wise TEI plot (matching Fig. 1d) and optional partial TEI profiles per gene age class

## How to apply

First build a transcript-to-gene table with oggmap gtf2t2g from a GTF annotation (e.g. Danio rerio GRCz11) and use it to collapse/map transcript-level IDs in the AnnData to gene-level IDs. Match the orthomap gene age classes to the AnnData var_names with the oggmap.orthomap2tei helper, keeping unannotated genes out of the weighted calculation. Compute per-cell TEI with orthomap2tei get_tei and store the resulting vector in the AnnData obs (saved as e.g. zebrafish_tei.h5ad). Finally aggregate mean TEI per developmental stage into a summary table and generate a stage-wise TEI plot (matching Fig. 1d); partial TEI profiles per gene age class can also be calculated and visualized. Rationale: weighting expression by gene age class exposes evolutionary signatures in cell-type and stage-resolved scRNA data.

## Related tools

- **oggmap** (Provides orthomap2tei helpers (get_tei) to match gene age classes to scRNA data and calculate/visualize TEI and partial TEI profiles) — https://github.com/kullrich/oggmap
- **Scanpy** (Python toolkit used to load and manipulate the scRNA sequencing (AnnData) objects into which TEI values are written)
- **gtf2t2g** (oggmap subcommand to extract a transcript-to-gene table from a GTF file for mapping transcript-level IDs to gene-level IDs) — https://github.com/kullrich/oggmap

## Examples

```
from oggmap import orthomap2tei; adata.obs['TEI'] = orthomap2tei.get_tei(dataset=adata.X, gene_names=adata.var_names.tolist(), orthomap=orthomap, gene_set_len=False)
```

## Evaluation signals

- Stage-level summary table exists and the developmental stage with the minimum TEI equals the 12-hpf stage (hourglass reproduction)
- Per-cell TEI vector is stored in AnnData obs and the object round-trips as zebrafish_tei.h5ad
- Genes without an assigned age class are excluded from the weighted TEI calculation rather than silently zero-weighted
- var_names after mapping are gene-level IDs consistent with the orthomap keys (via gtf2t2g transcript-to-gene table)

## Limitations

- oggmap v0.0.2 requires Python 3.10; Python 2.7x or <=3.10 (older) is not supported
- Only genes matching orthomap age classes contribute to TEI; unannotated genes are dropped, which can bias the index if coverage is low
- No dataset accessions or data availability statements are provided in the discussed sections, so input scRNA objects must be sourced separately (e.g. GEO-deposited)

## Evidence

- [methods] calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [methods] match gene/transcript IDs with scRNA data: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [methods] gtf2t2g subcommand: "gtf2t2g             extract transcript to gene table from GTF <gtf2t2g -h>"
- [abstract] Scanpy-based weighted expression: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [abstract] oggmap visualizes gene category expression profiles in evolutionary context: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles to investigate scRNA-based cell type assignments in an"

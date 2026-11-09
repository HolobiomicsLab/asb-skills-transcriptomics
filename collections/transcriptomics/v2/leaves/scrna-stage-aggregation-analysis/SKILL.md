---
name: scrna-stage-aggregation-analysis
description: Use when a scRNA-seq AnnData object already contains per-cell TEI (or
  similar gene age-weighted index) in obs and the goal is to summarize and plot mean
  index per developmental stage to detect an hourglass pattern.
when_to_use_negative:
- The input object has no per-cell gene age-weighted index in obs — compute TEI first
  with orthomap2tei rather than aggregating raw expression
- The analysis targets single-cell type-level TEI profiles rather than global/stage-level
  aggregation
- Gene IDs in var_names are transcript-level and have not yet been collapsed to gene
  level (e.g. via gtf2t2g from a GRCz11 GTF)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: oggmap
  role: Provides orthomap2tei helpers to match gene age classes to scRNA data and
    calculate per-cell TEI before stage aggregation
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Toolkit for handling the scRNA AnnData object, storing TEI in obs, and aggregating/plotting
    per developmental stage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:3f25f6bb5a90d7c55cc5b0dc14f6579d06bc006289a1ecc92aa644a9175b4d6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scrna-stage-aggregation-analysis

## Summary

Aggregates per-cell transcriptome evolutionary index (TEI) values from a Scanpy AnnData object into per-developmental-stage summaries and stage-wise plots, to test whether the hourglass pattern (lowest global TAI/TEI at a mid-developmental stage, e.g. 12 hpf in zebrafish) is reproduced.

## When to use

Use when a scRNA-seq AnnData object already contains per-cell TEI (or similar gene age-weighted index) in obs and the goal is to summarize and plot mean index per developmental stage to detect an hourglass pattern.

## When NOT to use

- The input object has no per-cell gene age-weighted index in obs — compute TEI first with orthomap2tei rather than aggregating raw expression
- The analysis targets single-cell type-level TEI profiles rather than global/stage-level aggregation
- Gene IDs in var_names are transcript-level and have not yet been collapsed to gene level (e.g. via gtf2t2g from a GRCz11 GTF)

## Inputs

- AnnData scRNA-seq object (.h5ad) with per-cell TEI values in obs and developmental stage annotations (e.g. hpf stages)
- Orthomap gene age class table from oggmap
- Per-cell TEI vector computed via orthomap2tei get_tei

## Outputs

- Stage-level summary table of mean TEI per developmental stage
- Stage-wise TEI plot (analogous to Fig. 1d)
- Annotated AnnData object saved as e.g. zebrafish_tei.h5ad

## How to apply

After computing per-cell TEI with oggmap's orthomap2tei get_tei and storing it in AnnData obs, group cells by their developmental stage annotation (e.g. hours postfertilization for the ~70,000-cell zebrafish dataset) and aggregate mean TEI per stage into a summary table. Exclude unannotated genes from the weighted calculation when matching orthomap gene age classes to AnnData var_names via orthomap2tei. Visualize the stage-wise TEI profile to compare against the published Fig. 1d hourglass curve. Validate by confirming the summary table exists and that the developmental stage with the minimum TEI equals the expected stage (12 hpf in the zebrafish re-analysis). Save the annotated object (e.g. zebrafish_tei.h5ad) so the per-cell vector and stage aggregation remain reproducible.

## Related tools

- **oggmap** (Provides orthomap2tei helpers to match gene age classes to scRNA data and calculate per-cell TEI before stage aggregation) — https://github.com/kullrich/oggmap
- **Scanpy** (Toolkit for handling the scRNA AnnData object, storing TEI in obs, and aggregating/plotting per developmental stage)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('zebrafish_tei.h5ad'); stage_tei = adata.obs.groupby('stage')['TEI'].mean(); stage_tei.idxmin()
```

## Evaluation signals

- A stage-level summary table exists with one mean-TEI row per developmental stage
- The developmental stage with the minimum mean TEI equals the 12-hpf stage (hourglass validation criterion from the source task)
- Per-cell TEI vector is present in AnnData obs and the object persists as a valid .h5ad file
- Stage-wise plot reproduces the shape of the published Fig. 1d zebrafish TEI profile
- Unannotated genes were excluded from the TEI weighted calculation (only orthomap-matched var_names contribute)

## Limitations

- Genes lacking an orthomap gene age class must be kept out of the weighted TEI calculation, so the index covers only orthogroup-annotated genes
- oggmap requires Python 3.10 (Python 2.7x and <=3.10 legacy versions are not supported)
- Matching depends on correct transcript-to-gene ID mapping (e.g. Danio rerio GRCz11 GTF via gtf2t2g); mismatched IDs silently reduce coverage

## Evidence

- [methods] calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [methods] match gene/transcript IDs and age classes with scRNA data: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [abstract] weighted expression from scRNA objects via Scanpy: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [readme] README zebrafish TEI example figure: "Oggmap results (gene ages per orthologous group) can be further used to calculate and visualize weighted expression data
(transcriptome evolutionary index) from scRNA sequencing objects."

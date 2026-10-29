---
name: gene-category-expression-profiling
description: Use when a per-gene scalar metric (e.g. nucleotide diversity, Tajima's
  D, F-statistics) is available and you want a cell-level weighted expression index
  using those bins as weights in place of evolutionary gene age classes.
when_to_use_negative:
- The weights you want are evolutionary gene ages per orthogroup — then use oggmap's
  standard orthomap/TEI workflow (of2orthomap/plaza2orthomap) rather than binning
  an arbitrary metric.
- The gene metric is not available per gene (e.g. only per-cell or per-sample measurements)
  — binning requires a gene-level value for each gene to be mapped to a bin group.
- The data are bulk RNA-seq rather than scRNA-seq AnnData objects, since oggmap's
  TEI calculation is implemented on Scanpy objects.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0194
tools:
- name: oggmap
  role: Provides get_bins to bin arbitrary gene-based metrics into gene groups and
    the TEI calculation used to weigh expression by those bin groups on Scanpy objects
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Toolkit through which oggmap calculates the gene category-weighted expression
    data from scRNA sequencing objects
- name: Python
  role: Implementation language; oggmap v0.0.2 was developed using Python 3.10 and
    is installed via PyPI or docker
  repo: https://github.com/kullrich/oggmap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:3fdc3b0b21de26b886808fd31d93eb3dd0fb0b9b72afe32434c44bada14b4fee
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-category-expression-profiling

## Summary

Weight single-cell expression by arbitrary per-gene categories (e.g. nucleotide diversity or Tajima's D bins) instead of gene age classes, using oggmap's get_bins to bin gene-level metrics and its Scanpy-based TEI machinery to compute the resulting weighted expression index per cell.

## When to use

Use when a per-gene scalar metric (e.g. nucleotide diversity, Tajima's D, F-statistics) is available and you want a cell-level weighted expression index using those bins as weights in place of evolutionary gene age classes.

## When NOT to use

- The weights you want are evolutionary gene ages per orthogroup — then use oggmap's standard orthomap/TEI workflow (of2orthomap/plaza2orthomap) rather than binning an arbitrary metric.
- The gene metric is not available per gene (e.g. only per-cell or per-sample measurements) — binning requires a gene-level value for each gene to be mapped to a bin group.
- The data are bulk RNA-seq rather than scRNA-seq AnnData objects, since oggmap's TEI calculation is implemented on Scanpy objects.

## Inputs

- scRNA-seq count object as AnnData (handled via Scanpy), e.g. nematode developmental scRNA data
- per-gene metric table (e.g. nucleotide diversity, Tajima's D, or F-statistics per gene) loadable with pandas
- gene/transcript ID definitions from the scRNA data set for ID matching (extractable via oggmap gtf2t2g from GTF files)

## Outputs

- binning_map.tsv — gene-to-bin-group mapping table from get_bins
- per-cell weighted expression index (TEI analogue) computed with bin groups as weights, stored on the Scanpy object
- binned_index_per_stage.tsv — binned index aggregated per developmental stage

## How to apply

Load the scRNA count object (AnnData, handled via Scanpy) and the per-gene metric table (e.g. nucleotide diversity or Tajima's D per gene) with pandas. Use oggmap's get_bins function to partition genes into discrete bins by their metric values, producing a gene-to-bin-group mapping that is written out as binning_map.tsv. Match gene and transcript IDs in the binned mapping with the identifiers in the scRNA data set, following the same oggmap orthomap-to-scRNA matching step used for gene ages. Then compute the weighted expression index (the TEI analogue, with bin groups serving as weights instead of gene age classes) per cell using oggmap's TEI calculation on Scanpy objects, and aggregate the binned index per developmental stage into binned_index_per_stage.tsv. The rationale is that oggmap's weighting framework generalizes beyond evolutionary age: any discretized gene-based metric can supply the weights.

## Related tools

- **oggmap** (Provides get_bins to bin arbitrary gene-based metrics into gene groups and the TEI calculation used to weigh expression by those bin groups on Scanpy objects) — https://github.com/kullrich/oggmap
- **Scanpy** (Toolkit through which oggmap calculates the gene category-weighted expression data from scRNA sequencing objects)
- **Python** (Implementation language; oggmap v0.0.2 was developed using Python 3.10 and is installed via PyPI or docker) — https://github.com/kullrich/oggmap

## Examples

```
from oggmap import get_bins; bins = get_bins(dataset=adata, gene_id_col='gene_id', binning_map=binning_map.tsv)
```

## Evaluation signals

- binning_map.tsv contains a gene-to-bin-group mapping covering the genes in the metric table, with each gene assigned to exactly one discrete bin by its metric value
- Gene/transcript IDs in the binned mapping successfully match identifiers present in the scRNA AnnData object after the ID-matching step
- Per-cell binned index values are finite and within the numeric range spanned by the bin labels used as weights
- binned_index_per_stage.tsv contains one aggregated index value per developmental stage and no missing stage entries
- The weighted index is reproducible when recomputed with the same bins and AnnData object via oggmap's TEI calculation

## Limitations

- oggmap v0.0.2 was developed using Python 3.10; Python 2.7x and Python <=3.10 are not supported.
- Genes lacking a metric value cannot be assigned to a bin, so the gene-to-bin mapping must cover the genes to be weighted.
- Gene/transcript ID mismatches between the metric table and the scRNA data set require the same ID-matching handling as the orthomap workflow.

## Evidence

- [results] get_bins bins arbitrary gene-based metrics into gene groups usable as weights: [evidence span withheld — anchor status: not_found]
- [abstract] oggmap supports visualization of categories like nucleotide diversity bins: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles to investigate scRNA-based cell type assignments in an"
- [abstract] TEI calculated on Scanpy objects: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] ID matching between gene age class and scRNA data sets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [methods] Python version constraint: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."

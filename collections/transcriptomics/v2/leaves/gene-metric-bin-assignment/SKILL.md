---
name: gene-metric-bin-assignment
description: Use when you have a per-gene metric table (e.g. Tajima's D or nucleotide
  diversity per gene) and a nematode/zebrafish scRNA count object (AnnData via Scanpy),
  and want a binned gene-category-weighted expression index instead of a gene-age-weighted
  one.
when_to_use_negative:
- The goal is a standard evolutionary gene-age orthomap — use oggmap's of2orthomap/plaza2orthomap
  workflow instead of metric binning.
- No per-gene metric table exists for the query species, so bins cannot be computed
  and only orthogroup-based gene ages are available.
- The gene-to-bin mapping already exists as a finished group table — skip get_bins
  and go directly to ID matching and TEI calculation.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: oggmap
  role: Provides get_bins to partition genes into discrete bins by metric value, and
    the TEI calculation to compute the bin-weighted expression index on Scanpy objects
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Toolkit through which the AnnData scRNA sequencing object is handled and the
    gene category-weighted expression data are calculated
- name: Python
  role: Runtime for oggmap v0.0.2; oggmap was developed using Python 3.10
- name: pandas
  role: Loads the per-gene metric table and writes the binning_map.tsv gene-to-bin-group
    mapping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:7a6c88794ca63b895abc73460ba55be151e0bce270f711f4dcfe028eb5ba910a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-metric-bin-assignment

## Summary

Partition genes into discrete bins by an arbitrary per-gene metric (e.g. nucleotide diversity, Tajima's D, F-statistics) using oggmap's get_bins, and use the resulting bin groups as expression weights in place of evolutionary gene age classes to compute a TEI-analogue index per cell.

## When to use

Use when you have a per-gene metric table (e.g. Tajima's D or nucleotide diversity per gene) and a nematode/zebrafish scRNA count object (AnnData via Scanpy), and want a binned gene-category-weighted expression index instead of a gene-age-weighted one.

## When NOT to use

- The goal is a standard evolutionary gene-age orthomap — use oggmap's of2orthomap/plaza2orthomap workflow instead of metric binning.
- No per-gene metric table exists for the query species, so bins cannot be computed and only orthogroup-based gene ages are available.
- The gene-to-bin mapping already exists as a finished group table — skip get_bins and go directly to ID matching and TEI calculation.

## Inputs

- scRNA count object as AnnData (handled via Scanpy), e.g. nematode developmental-stage scRNA data
- per-gene metric table (e.g. nucleotide diversity or Tajima's D per gene) loadable with pandas
- gene-to-transcript ID mapping or gene/transcript identifiers present in the scRNA data set

## Outputs

- binning_map.tsv — gene-to-bin-group mapping table
- binned_index_per_stage.tsv — binned TEI-analogue index aggregated per developmental stage

## How to apply

Load the nematode scRNA count object (AnnData, handled via Scanpy) and the per-gene metric table (e.g. nucleotide diversity or Tajima's D per gene) with pandas. Partition genes into discrete bins by their metric values using the oggmap get_bins function, then write the gene-to-bin-group mapping as binning_map.tsv. Match gene and transcript IDs in the binned mapping against the identifiers in the scRNA data set, following the same matching step oggmap uses to link gene age class with scRNA data sets. Compute the TEI analogue per cell using the bin groups as weights via oggmap's TEI calculation on the Scanpy object, then aggregate the binned index per developmental stage into binned_index_per_stage.tsv. The rationale is that oggmap treats gene categories generically — the abstract explicitly names 'age class, nucleotide diversity bin' as interchangeable categories — so any gene-based metric can be binned and used as a weight column.

## Related tools

- **oggmap** (Provides get_bins to partition genes into discrete bins by metric value, and the TEI calculation to compute the bin-weighted expression index on Scanpy objects) — https://github.com/kullrich/oggmap
- **Scanpy** (Toolkit through which the AnnData scRNA sequencing object is handled and the gene category-weighted expression data are calculated)
- **Python** (Runtime for oggmap v0.0.2; oggmap was developed using Python 3.10)
- **pandas** (Loads the per-gene metric table and writes the binning_map.tsv gene-to-bin-group mapping)

## Examples

```
import pandas as pd; bins = oggmap.get_bins(pd.read_csv('gene_metrics.tsv', sep='\t')); bins.to_csv('binning_map.tsv', sep='\t')
```

## Evaluation signals

- binning_map.tsv assigns every gene with a metric value to exactly one discrete bin group, covering the metric range partitioned by get_bins
- Gene/transcript identifiers in binning_map.tsv match the identifiers in the scRNA AnnData object (same matching logic as the orthomap-to-scRNA matching step)
- The binned TEI analogue per cell is computed without error on the Scanpy object, mirroring the standard TEI calculation path
- binned_index_per_stage.tsv contains one aggregated index value per developmental stage, analogous to oggmap's TEI-per-stage output
- Substituting a different metric (e.g. Tajima's D vs nucleotide diversity) changes the bin groups and correspondingly shifts the index, confirming the weights are actually applied

## Limitations

- oggmap v0.0.2 was developed using Python 3.10; 'We do not support Python `2.7x` or Python `<=3.10`.'
- Genes lacking the per-gene metric value or whose identifiers do not match the scRNA data set cannot be assigned a bin and are excluded from the weighted index.
- The skill mirrors the gene-age TEI machinery; the bin index is an analogue, not the canonical transcriptome evolutionary index based on LCA gene ages.

## Evidence

- [abstract] oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles"
- [methods] match gene and transcript IDs and gene age class with scRNA data sets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
- [methods] calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [methods] calculate mean/relative expression per gene age class: "- calculate mean/relative expression per gene age class."
- [readme] Oggmap results can be further used to calculate and visualize weighted expression data from scRNA sequencing objects: [evidence span withheld — anchor status: external_doc]

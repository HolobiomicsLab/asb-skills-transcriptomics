---
name: coverage-based-bulge-removal
description: Use when assembling isoforms per read cloud (barcode) from barcoded RNA-seq
  linked reads and the extracted barcode subgraph contains parallel (bulge) paths
  between the same node pairs that must be resolved using barcode-specific coverage
  instead of aggregate graph coverage.
when_to_use_negative:
- The goal is to clean the single shared assembly graph itself — deletions there would
  remove edges that are correct isoform evidence for other barcodes
- Input is unbarcoded (standard) RNA-seq or genomic reads, where rnaSPAdes/genomic
  SPAdes pipelines apply graph simplification to the main graph instead
- The subgraph contains no parallel edges between the same node pairs (no bulge structure
  to resolve)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cloudrnaSPAdes
  role: Performs per-read-cloud subgraph extraction, barcode-specific coverage computation,
    and bulge removal by comparing barcode coverage of parallel edges connecting the
    same node pairs
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes / spades-gbuilder
  role: Builds the shared assembly graph from the barcoded read set that the per-barcode
    subgraphs are derived from
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Bandage
  role: Visual inspection of the assembly graph structure (nodes/edges, paths) when
    checking bulge topology and resolved subgraphs
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:4fa4829bbd1caae539e6f5d4191459ded335930ad16e424c022472279f74570d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - SPAdes
  - SPAdes / spades-gbuilder
  - cloudrnaSPAdes
  - spades-gbuilder
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# coverage-based-bulge-removal

## Summary

Resolves bulges in each barcode-specific subgraph built from a shared SPAdes assembly graph by comparing barcode-specific coverage of parallel edges connecting the same node pairs, rather than editing the shared graph, so that an edge erroneous for one barcode's isoform can remain available for another barcode's isoform.

## When to use

Use when assembling isoforms per read cloud (barcode) from barcoded RNA-seq linked reads and the extracted barcode subgraph contains parallel (bulge) paths between the same node pairs that must be resolved using barcode-specific coverage instead of aggregate graph coverage.

## When NOT to use

- The goal is to clean the single shared assembly graph itself — deletions there would remove edges that are correct isoform evidence for other barcodes
- Input is unbarcoded (standard) RNA-seq or genomic reads, where rnaSPAdes/genomic SPAdes pipelines apply graph simplification to the main graph instead
- The subgraph contains no parallel edges between the same node pairs (no bulge structure to resolve)

## Inputs

- Barcoded RNA-seq linked-read data (e.g. simulated GYPC gene barcoded reads)
- Shared assembly graph built with spades-gbuilder (SPAdes graph construction)
- Per-barcode read clouds grouping reads by barcode
- Barcode-specific edge coverage computed along sequence-of-edge paths in each subgraph

## Outputs

- Per-barcode simplified subgraphs with bulges resolved (parallel edges resolved by barcode coverage)
- Simple isoform paths extracted from each simplified per-barcode subgraph

## How to apply

First build the shared assembly graph from the barcoded RNA-seq read set (e.g. simulated GYPC barcoded reads) with the SPAdes assembly graph construction module (spades-gbuilder). For each read cloud, extract a barcode-specific subgraph while leaving the base (shared) graph unmodified, and compute barcode-specific coverage for every edge along sequence-of-edge paths within the subgraph. Detect bulges as sets of parallel edges connecting the same node pairs within a barcode subgraph, and resolve each bulge by comparing the barcode-specific coverage of the parallel edges — retaining the path supported by that barcode's coverage, since an edge erroneous for one barcode's isoform may be correct for another barcode's isoform. Perform this simplification in the same per-subgraph pass as tip clipping (removal of dead-end/dead-start edges and low-barcode-coverage tips) and gap closing (joining edges across short low-coverage edges), then extract simple isoform paths from each simplified subgraph. Keep all deletions local to the per-barcode subgraph so downstream barcodes still see the full shared graph.

## Related tools

- **cloudrnaSPAdes** (Performs per-read-cloud subgraph extraction, barcode-specific coverage computation, and bulge removal by comparing barcode coverage of parallel edges connecting the same node pairs) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes / spades-gbuilder** (Builds the shared assembly graph from the barcoded read set that the per-barcode subgraphs are derived from) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Bandage** (Visual inspection of the assembly graph structure (nodes/edges, paths) when checking bulge topology and resolved subgraphs) — https://github.com/rrwick/Bandage

## Evaluation signals

- Shared assembly graph must remain byte-identical before and after per-barcode simplification (all bulge resolution is confined to barcode subgraphs)
- After bulge removal, each pair of nodes previously joined by parallel edges retains a single path selected by barcode-specific coverage
- Each simplified per-barcode subgraph must yield extractable simple isoform paths
- Extracted isoform paths must be consistent with the known isoform structure of the GYPC gene in the validation data set
- Visual check of simplified subgraphs (e.g. in Bandage) shows no residual parallel-edge bulge structures

## Limitations

- An edge erroneous for one barcode's isoform may be correct for another barcode's isoform, so bulge resolution must never modify the shared assembly graph
- Bulge resolution depends on barcode-specific coverage; very low-coverage true isoform edges within a bulge can be mis-dropped
- No changelog found for the release repository and Discussion section content was absent, so exact parameter defaults for the bulge-removal step are not documented in the provided context

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [methods] per-barcode subgraph workflow: [evidence span withheld — anchor status: not_found]
- [methods] shared-graph preservation rationale: [evidence span withheld — anchor status: not_found]
- [readme] graph construction tool: [evidence span withheld — anchor status: not_found]
- [readme] graph visualization support: [evidence span withheld — anchor status: external_doc]

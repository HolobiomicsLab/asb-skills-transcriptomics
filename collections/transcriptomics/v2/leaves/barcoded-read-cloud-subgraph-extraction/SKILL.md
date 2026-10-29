---
name: barcoded-read-cloud-subgraph-extraction
description: 'Use when assembling reference-free isoforms from barcoded RNA-seq linked
  reads: you have a shared SPAdes assembly graph plus per-read barcode tags and need
  barcode-specific subgraphs for simplification.'
when_to_use_negative:
- Input is standard (non-barcoded) bulk RNA-seq — use rnaSPAdes rather than read-cloud
  subgraph extraction.
- Input is WGS isolate or metagenomic data — use the SPAdes/metaSPAdes pipelines;
  SPAdes is not intended for larger (e.g. mammalian-size) genomes.
- The goal is to clean the shared assembly graph itself — edges must not be deleted
  from the base graph because an edge erroneous for one barcode may be correct for
  another barcode's isoform.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Performs per-read-cloud barcode subgraph extraction and barcode-aware simplification
    (tip clipping, bulge removal, gap closing) on the subgraph rather than the shared
    graph
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Feature release backbone; its spades-gbuilder module constructs the shared
    assembly graph from which per-barcode subgraphs are extracted
  repo: http://cab.spbu.ru/software/spades/
- name: Bandage
  role: GUI visualization of the shared assembly graph and per-barcode subgraphs/paths
    to inspect simplification results
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:a371839f64be34ef2361c8c34a0f3c60f80a071760d9f4ce0df9fe1a31db50f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - SPAdes
  - cloudrnaSPAdes
  - spades-gbuilder
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# barcoded-read-cloud-subgraph-extraction

## Summary

Extracts per-read-cloud (barcode) subgraphs from a shared SPAdes assembly graph during cloudrnaSPAdes isoform assembly, computes barcode-specific edge coverage along sequence-of-edge paths, and simplifies each subgraph (tip clipping, bulge removal, gap closing) without modifying the shared graph.

## When to use

Use when assembling reference-free isoforms from barcoded RNA-seq linked reads: you have a shared SPAdes assembly graph plus per-read barcode tags and need barcode-specific subgraphs for simplification.

## When NOT to use

- Input is standard (non-barcoded) bulk RNA-seq — use rnaSPAdes rather than read-cloud subgraph extraction.
- Input is WGS isolate or metagenomic data — use the SPAdes/metaSPAdes pipelines; SPAdes is not intended for larger (e.g. mammalian-size) genomes.
- The goal is to clean the shared assembly graph itself — edges must not be deleted from the base graph because an edge erroneous for one barcode may be correct for another barcode's isoform.

## Inputs

- Barcoded RNA-seq linked reads (Illumina FASTQ with per-read barcode tags)
- Shared SPAdes assembly graph built by spades-gbuilder from the barcoded read set
- Per-read barcode (read cloud) assignments

## Outputs

- Per-barcode simplified subgraphs (shared assembly graph left unmodified)
- Barcode-specific edge coverage values along sequence-of-edge paths within each subgraph
- Simple isoform paths extracted from each simplified per-barcode subgraph

## How to apply

First build a shared assembly graph from the barcoded RNA-seq reads with spades-gbuilder (run BayesHammer/IonHammer error correction first, per SPAdes recommendations). For each read cloud (barcode), extract the barcode-specific subgraph of the shared assembly graph, leaving the base graph unmodified — this is essential because an edge erroneous for one barcode's isoform may be correct for another barcode's isoform. Compute barcode-specific coverage for every edge along sequence-of-edge paths within each subgraph, then clip tips that are dead-end/dead-start edges or have low barcode-specific coverage. Remove bulges by comparing barcode coverage of parallel edges connecting the same node pairs, and close gaps by joining edges across short low-coverage edges in the subgraph. Finally, extract simple isoform paths from each simplified subgraph and write out the per-barcode simplified subgraphs.

## Related tools

- **cloudrnaSPAdes** (Performs per-read-cloud barcode subgraph extraction and barcode-aware simplification (tip clipping, bulge removal, gap closing) on the subgraph rather than the shared graph) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Feature release backbone; its spades-gbuilder module constructs the shared assembly graph from which per-barcode subgraphs are extracted) — http://cab.spbu.ru/software/spades/
- **Bandage** (GUI visualization of the shared assembly graph and per-barcode subgraphs/paths to inspect simplification results) — https://github.com/rrwick/Bandage

## Examples

```
spades-gbuilder reads.yaml -k 55 -s shared_graph.gfa  # shared graph built from barcoded reads, then per-barcode subgraphs extracted by cloudrnaSPAdes
```

## Evaluation signals

- The shared assembly graph is unchanged after simplification — no edges are deleted from the base graph, only from per-barcode subgraphs.
- Each simplified per-barcode subgraph yields extractable simple isoform paths consistent with the known isoform structure of the target gene (e.g. GYPC).
- Removed tips are restricted to dead-end/dead-start edges and edges with low barcode-specific coverage, not high-coverage edges.
- Bulges are removed only between parallel edges connecting the same node pairs, using barcode-coverage comparison.
- Barcode-specific coverage is computed for every edge along sequence-of-edge paths within each subgraph before any clipping decision.

## Limitations

- SPAdes 3.13.1 is not intended for larger (e.g. mammalian-size) genomes — usable only at one's own risk — although cloudrnaSPAdes was evaluated on simulated and real human data.
- Simplification decisions are barcode-specific: an edge erroneous for one barcode's isoform may be correct for another's, so any simplification applied to the shared graph would be wrong.
- MismatchCorrector is turned off by default; read error correction (BayesHammer/IonHammer) is recommended before assembly graph construction.

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [results] it removes short tips (dead-end and dead-start edges) as well as tips with low barcode-specific coverage, rather than deleting edges from the shared assembly graph: [evidence span withheld — anchor status: not_found]
- [methods] assembly graph construction (spades-gbuilder): [evidence span withheld — anchor status: not_found]
- [readme] De novo assembly graphs contain not only assembled contigs but also the connections between those contigs, which were previously not easily accessible.: [evidence span withheld — anchor status: not_found]
- [readme] SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk.: "SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk."

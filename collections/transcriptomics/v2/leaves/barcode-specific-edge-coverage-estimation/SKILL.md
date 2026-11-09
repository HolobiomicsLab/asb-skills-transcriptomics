---
name: barcode-specific-edge-coverage-estimation
description: Use when assembling isoforms de novo from bulk barcoded RNA-seq linked
  reads and you need edge support quantified per barcode/read cloud, because an edge
  erroneous for one barcode's isoform may be correct for another barcode's isoform.
when_to_use_negative:
- Input reads carry no barcodes (standard isolate or single-cell WGS data), so no
  barcode-specific coverage can be defined
- The goal is whole-genome assembly where an erroneous edge is erroneous for all reads,
  and shared-graph-level simplification is appropriate
- The assembly graph is already resolved into isoform paths and no per-barcode subgraph
  extraction is needed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: cloudrnaSPAdes
  role: Performs read cloud subgraph extraction and barcode-specific coverage computation
    used for tip clipping, bulge removal, and gap closing
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: spades-gbuilder
  role: Builds the shared assembly graph from which barcode-specific subgraphs are
    extracted without modifying the base graph
  repo: http://cab.spbu.ru/software/spades/
- name: Bandage
  role: GUI inspection of assembly graph paths and connections to visually validate
    simplified per-barcode subgraphs
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:89952421be14f4bce007624688d7dbd8a89a008c2243cd307bb73a47697daa8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - SPAdes
  - cloudrnaSPAdes
  - spades-gbuilder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# barcode-specific-edge-coverage-estimation

## Summary

Computes per-barcode edge coverage along sequence-of-edge paths within the barcode-specific subgraph extracted for each read cloud, so that simplification (tip clipping, bulge removal, gap closing) is driven by barcode-resolved coverage instead of coverage aggregated on the shared assembly graph.

## When to use

Use when assembling isoforms de novo from bulk barcoded RNA-seq linked reads and you need edge support quantified per barcode/read cloud, because an edge erroneous for one barcode's isoform may be correct for another barcode's isoform.

## When NOT to use

- Input reads carry no barcodes (standard isolate or single-cell WGS data), so no barcode-specific coverage can be defined
- The goal is whole-genome assembly where an erroneous edge is erroneous for all reads, and shared-graph-level simplification is appropriate
- The assembly graph is already resolved into isoform paths and no per-barcode subgraph extraction is needed

## Inputs

- barcoded RNA-seq linked reads (e.g., simulated GYPC gene reads) in FASTQ
- shared SPAdes assembly graph built with spades-gbuilder (e.g., in GFA)
- read-to-barcode assignments defining read clouds

## Outputs

- per-edge barcode-specific coverage values within each read cloud subgraph
- per-barcode simplified subgraphs with tips clipped, bulges removed, and gaps closed
- extractable simple isoform paths per barcode

## How to apply

First build the shared assembly graph from the barcoded RNA-seq read set with the SPAdes assembly graph construction module (spades-gbuilder), leaving this base graph unmodified throughout. For each read cloud (barcode), extract the barcode-specific subgraph of the shared graph. Compute barcode-specific coverage for every edge along sequence-of-edge paths inside each subgraph. Use these coverage values to clip low-coverage tips (dead-end and dead-start edges), to remove bulges by comparing barcode coverage of parallel edges connecting the same node pairs, and to close gaps by joining edges across short low-coverage edges — all strictly on the subgraph, never on the shared graph, so barcode-specific isoform diversity is preserved.

## Related tools

- **cloudrnaSPAdes** (Performs read cloud subgraph extraction and barcode-specific coverage computation used for tip clipping, bulge removal, and gap closing) — https://github.com/1dayac/cloudrnaSPAdes-release
- **spades-gbuilder** (Builds the shared assembly graph from which barcode-specific subgraphs are extracted without modifying the base graph) — http://cab.spbu.ru/software/spades/
- **Bandage** (GUI inspection of assembly graph paths and connections to visually validate simplified per-barcode subgraphs) — https://github.com/rrwick/Bandage

## Evaluation signals

- Each simplified per-barcode subgraph must yield extractable simple isoform paths
- Extracted isoform paths are consistent with the known isoform structure of the source gene (e.g., GYPC)
- The shared assembly graph remains unmodified — no edges deleted from the base graph
- Bulge resolution decisions are traceable to comparisons of barcode coverage between parallel edges connecting the same node pairs
- Clipped tips are confined to the barcode-specific subgraph (dead-end/dead-start edges or low barcode-specific coverage tips)

## Limitations

- Thresholds and cutoffs for low-coverage tip clipping and short low-coverage gap edges are not stated in the provided context, so parameters must be set empirically
- Because an edge erroneous for one barcode's isoform may be correct for another's, no edge can be globally pruned from the shared graph — errors must be handled per barcode subgraph
- The article's discussion section content was absent and no changelog was found, limiting detail on parameter evolution

## Evidence

- [abstract] cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] Standalone graph construction binary context: [evidence span withheld — anchor status: not_found]
- [readme] SPAdes toolkit framing: "SPAdes &ndash; St. Petersburg genome assembler &ndash; is an assembly toolkit containing various assembly pipelines"

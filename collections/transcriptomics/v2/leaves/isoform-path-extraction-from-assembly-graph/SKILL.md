---
name: isoform-path-extraction-from-assembly-graph
description: 'Use when assembling full-length isoforms reference-free from barcoded
  RNA-seq linked reads: after per-barcode subgraph simplification (tip clipping, bulge
  removal, gap closing) you need to read out simple isoform paths per read cloud.'
when_to_use_negative:
- The input is standard short-read RNA-seq without barcodes — use rnaSPAdes or a reference-based
  transcriptome assembler instead, since there are no read clouds to derive per-barcode
  subgraphs from.
- You need to modify or prune the shared assembly graph globally — this skill deliberately
  avoids touching the shared graph, since an edge erroneous for one barcode's isoform
  may be correct for another barcode's isoform.
- The gene has low isoform diversity and a single dominant transcript — direct path
  extraction from one global subgraph may be simpler than per-read-cloud processing.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Performs per-read-cloud subgraph extraction, simplification, and simple isoform
    path extraction from barcoded RNA-seq data
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Feature release hosting cloudrnaSPAdes; supplies assembly graph construction
    (spades-gbuilder) used to build the shared graph before per-barcode simplification
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Bandage
  role: Used for visualising the assembly graph and graph paths to inspect isoform
    path structure
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:4047448f9f5861fe9e5abcab8ae21ea19148d2c1c45c58aa8eab12ccaf3d3340
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - SPAdes
  - cloudrnaSPAdes
  - spades-gbuilder
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isoform-path-extraction-from-assembly-graph

## Summary

Extract full-length isoform paths from simplified per-barcode subgraphs of a shared assembly graph, as done in cloudrnaSPAdes for bulk barcoded RNA-seq linked-read data. This step converts each barcode's cleaned subgraph into simple sequence-of-edges paths representing individual isoforms, without altering the shared assembly graph.

## When to use

Use when assembling full-length isoforms reference-free from barcoded RNA-seq linked reads: after per-barcode subgraph simplification (tip clipping, bulge removal, gap closing) you need to read out simple isoform paths per read cloud.

## When NOT to use

- The input is standard short-read RNA-seq without barcodes — use rnaSPAdes or a reference-based transcriptome assembler instead, since there are no read clouds to derive per-barcode subgraphs from.
- You need to modify or prune the shared assembly graph globally — this skill deliberately avoids touching the shared graph, since an edge erroneous for one barcode's isoform may be correct for another barcode's isoform.
- The gene has low isoform diversity and a single dominant transcript — direct path extraction from one global subgraph may be simpler than per-read-cloud processing.

## Inputs

- Barcoded RNA-seq linked-read data (e.g. simulated GYPC gene reads) in FASTQ
- Shared assembly graph built by SPAdes assembly graph construction (spades-gbuilder)
- Per-barcode read cloud assignments (barcode-to-edge/read mappings)
- Simplified per-barcode subgraphs after tip clipping, bulge removal, and gap closing

## Outputs

- Simple isoform paths (sequence-of-edge paths) extracted from each simplified barcode-specific subgraph
- Per-barcode simplified subgraphs with isoform paths written out
- Barcode-specific edge coverage values along sequence-of-edge paths

## How to apply

First construct a shared assembly graph from the barcoded RNA-seq reads (e.g. with spades-gbuilder), then for each read cloud (barcode) extract the barcode-specific subgraph and simplify it (clip low-barcode-coverage tips, remove bulges by comparing barcode coverage of parallel edges between the same node pairs, close gaps across short low-coverage edges). Crucially, all simplification is performed on the barcode-specific subgraph rather than the shared graph, because an edge erroneous for one barcode's isoform may be correct for another barcode's isoform. From each simplified subgraph, extract simple isoform paths as sequence-of-edge paths, using the barcode-specific edge coverage computed along those paths. Write out the per-barcode simplified subgraphs together with their isoform paths. Validate that each simplified per-barcode subgraph yields extractable simple isoform paths consistent with the known isoform structure of the target gene (e.g. the GYPC gene in the simulated test data).

## Related tools

- **cloudrnaSPAdes** (Performs per-read-cloud subgraph extraction, simplification, and simple isoform path extraction from barcoded RNA-seq data) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Feature release hosting cloudrnaSPAdes; supplies assembly graph construction (spades-gbuilder) used to build the shared graph before per-barcode simplification) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Bandage** (Used for visualising the assembly graph and graph paths to inspect isoform path structure) — https://github.com/rrwick/Bandage

## Examples

```
spades-gbuilder  # run without parameters to learn its options; use it to build the shared assembly graph from barcoded reads before per-barcode path extraction
```

## Evaluation signals

- Each simplified per-barcode subgraph must yield extractable simple isoform paths (no leftover complex/repeat structure blocking path extraction).
- Extracted isoform paths are consistent with the known isoform structure of the GYPC gene in the simulated validation data.
- The shared assembly graph is byte-identical before and after per-barcode simplification (simplification only touches barcode-specific subgraphs).
- Low-barcode-coverage tips and bulges (parallel edges between the same node pairs with differing barcode coverage) are absent from the final simplified subgraphs.
- Assembled isoforms match known transcripts for genes with high isoform diversity, as reported for simulated and real human data.

## Limitations

- cloudrnaSPAdes was designed and tested for bulk barcoded RNA-seq linked-read data; the manual's scope statements about SPAdes apply (designed for small genomes, not intended for mammalian-size genomes).
- Simplification operates per barcode on derived subgraphs, so path quality depends on accurate barcode-specific coverage estimates along sequence-of-edge paths.
- Gaps are closed only across short low-coverage edges; long unresolved repeats may prevent simple path extraction.

## Evidence

- [abstract] cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [methods] assembly graph construction via spades-gbuilder supplies the shared graph from which per-barcode subgraphs are derived: [evidence span withheld — anchor status: not_found]
- [intro] SPAdes as the assembly toolkit containing the pipeline in which cloudrnaSPAdes operates: "SPAdes &ndash; St. Petersburg genome assembler &ndash; is an assembly toolkit containing various assembly pipelines"
- [readme] Bandage can be used to visualise assembly graphs and their paths when inspecting extracted isoform paths: [evidence span withheld — anchor status: external_doc]
- [abstract] Accuracy of isoform assembly on high isoform diversity data supports the path-extraction validation: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."

---
name: assembly-graph-tip-clipping
description: Use when assembling full-length isoforms from barcoded RNA-seq linked-read
  data reference-free and per-barcode subgraphs contain short dead-end or dead-start
  edges, or tips with low barcode-specific coverage, that must be clipped before bulge
  removal and gap closing.
when_to_use_negative:
- The goal is to clean the shared assembly graph globally — erroneous edges must instead
  be removed only from each barcode-specific subgraph, since an edge erroneous for
  one barcode's isoform may be correct for another's.
- Input is standard (non-barcoded) RNA-seq — cloudrnaSPAdes targets barcoded linked-read
  data; for regular transcriptome assembly the SPAdes package provides rnaSPAdes instead.
- Input is already a simplified or path-resolved graph (e.g., extracted isoform paths
  or Bandage-rendered paths) with no dead-end/low-coverage tips left to clip.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: cloudrnaSPAdes
  role: Performs the per-read-cloud subgraph simplification, including tip clipping
    on barcode-specific subgraphs without modifying the shared assembly graph
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: spades-gbuilder
  role: Builds the shared assembly graph from the barcoded RNA-seq reads prior to
    per-barcode subgraph extraction and tip clipping
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Bandage
  role: GUI visualization of assembly graphs and graph paths for inspecting tips and
    simplified per-barcode subgraphs
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:822c575f08eca9aad3d54cf09b3b7317f446723176de1c5731c0cb14c195c082
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# assembly-graph-tip-clipping

## Summary

Per-read-cloud tip clipping of barcode-specific assembly subgraphs: cloudrnaSPAdes removes short dead-end/dead-start tips and tips with low barcode-specific edge coverage from each barcode's subgraph, deliberately leaving the shared SPAdes assembly graph unmodified.

## When to use

Use when assembling full-length isoforms from barcoded RNA-seq linked-read data reference-free and per-barcode subgraphs contain short dead-end or dead-start edges, or tips with low barcode-specific coverage, that must be clipped before bulge removal and gap closing.

## When NOT to use

- The goal is to clean the shared assembly graph globally — erroneous edges must instead be removed only from each barcode-specific subgraph, since an edge erroneous for one barcode's isoform may be correct for another's.
- Input is standard (non-barcoded) RNA-seq — cloudrnaSPAdes targets barcoded linked-read data; for regular transcriptome assembly the SPAdes package provides rnaSPAdes instead.
- Input is already a simplified or path-resolved graph (e.g., extracted isoform paths or Bandage-rendered paths) with no dead-end/low-coverage tips left to clip.

## Inputs

- Barcoded bulk RNA-seq linked-read data (e.g., simulated GYPC barcoded read set in FASTQ)
- Shared SPAdes assembly graph built with spades-gbuilder
- Read cloud (barcode) assignments grouping reads per barcode

## Outputs

- Per-barcode simplified subgraphs (tips clipped; bulges removed; gaps closed)
- Simple isoform paths extracted from each simplified subgraph
- Barcode-specific edge coverage values along sequence-of-edge paths

## How to apply

First build a shared assembly graph from the barcoded RNA-seq reads using the SPAdes graph construction module (spades-gbuilder), then for each read cloud (barcode) extract the barcode-specific subgraph so the base graph is left unmodified. Compute barcode-specific coverage for every edge along sequence-of-edge paths within each subgraph, since bulk graph coverage conflates isoforms from different barcodes. Clip tips from the barcode subgraph itself: remove short dead-end and dead-start edges, and remove tips whose barcode-specific coverage is low. The rationale is that an edge erroneous for one barcode's isoform may be correct for another barcode's isoform, so simplification must never delete edges from the shared assembly graph. After tip clipping, continue simplification on the same subgraph by removing bulges (parallel edges connecting the same node pairs, compared by barcode coverage) and closing gaps (joining edges across short low-coverage edges), then extract simple isoform paths.

## Related tools

- **cloudrnaSPAdes** (Performs the per-read-cloud subgraph simplification, including tip clipping on barcode-specific subgraphs without modifying the shared assembly graph) — https://github.com/1dayac/cloudrnaSPAdes-release
- **spades-gbuilder** (Builds the shared assembly graph from the barcoded RNA-seq reads prior to per-barcode subgraph extraction and tip clipping) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Bandage** (GUI visualization of assembly graphs and graph paths for inspecting tips and simplified per-barcode subgraphs) — https://github.com/rrwick/Bandage

## Examples

```
spades-gbuilder
```

## Evaluation signals

- The shared (base) assembly graph must be unchanged after simplification — compare the graph before and after to confirm no edges were deleted from it.
- Every tip removed from a barcode subgraph must be either a short dead-end or dead-start edge, or an edge with low barcode-specific coverage.
- Each simplified per-barcode subgraph must yield extractable simple isoform paths.
- Extracted isoform paths must be consistent with the known isoform structure of the target gene (validated on the simulated GYPC barcoded RNA-seq read set).
- Bulge removal should be justified by barcode-coverage comparison of parallel edges connecting the same node pairs, not by global coverage.

## Limitations

- Tip clipping decisions depend on barcode-specific coverage computed along sequence-of-edge paths; coverage estimation errors can misclassify a genuine isoform edge as a low-coverage tip.
- Simplification is only valid per barcode subgraph: globally removing edges from the shared graph would discard edges correct for other barcodes' isoforms.
- Validation described is on simulated GYPC barcoded RNA-seq data; performance on other genes depends on whether simplified subgraphs still yield simple isoform paths.

## Evidence

- [methods] it removes short tips (dead-end and dead-start edges) as well as tips with low barcode-specific coverage, rather than deleting edges from the shared assembly graph.: [evidence span withheld — anchor status: not_found]
- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [methods] assembly graph construction (spades-gbuilder): [evidence span withheld — anchor status: not_found]
- [methods] Build a shared assembly graph from the simulated GYPC barcoded RNA-seq read set using the SPAdes assembly graph construction module (spades-gbuilder).: [evidence span withheld — anchor status: not_found]
- [readme] De novo assembly graphs contain not only assembled contigs but also the connections between those contigs, which were previously not easily accessible.: [evidence span withheld — anchor status: not_found]

---
name: short-edge-gap-closing
description: Use when assembling isoforms from barcoded RNA-seq linked reads (read
  clouds per barcode) and a per-barcode subgraph contains short low-coverage edges
  interrupting otherwise continuous isoform paths.
when_to_use_negative:
- Input is standard unbarcoded RNA-seq without read clouds — use the rnaSPAdes transcriptome
  assembler pipeline instead of per-barcode subgraph simplification
- Gaps are too long to be bridged by short-read linked-read data — hybrid gap closure
  with PacBio CLR or Oxford Nanopore reads is the appropriate approach
- Goal is to correct the shared assembly graph itself — an edge erroneous for one
  barcode's isoform may be correct for another's, so edits must never be applied to
  the base graph
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Performs the per-read-cloud subgraph simplification, including gap closing
    across short low-coverage edges, after tip clipping and bulge removal
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes (spades-gbuilder)
  role: Builds the shared assembly graph from which barcode-specific subgraphs are
    extracted for gap closing
  repo: http://cab.spbu.ru/software/spades/
- name: Bandage
  role: Visual inspection of the per-barcode simplified subgraphs and their graph
    paths to confirm gap closure and isoform path continuity
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:ddaae0b0051c8d1f835033bdfea3cd640504d54f739dfa94b5e7ae738ff99a1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - SPAdes
  - SPAdes (spades-gbuilder)
  - cloudrnaSPAdes
  - spades-gbuilder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# short-edge-gap-closing

## Summary

cloudrnaSPAdes' gap-closing step: within each barcode-specific subgraph extracted from a shared assembly graph, edges are joined across short low-coverage edges to produce continuous isoform paths, without modifying the shared graph.

## When to use

Use when assembling isoforms from barcoded RNA-seq linked reads (read clouds per barcode) and a per-barcode subgraph contains short low-coverage edges interrupting otherwise continuous isoform paths.

## When NOT to use

- Input is standard unbarcoded RNA-seq without read clouds — use the rnaSPAdes transcriptome assembler pipeline instead of per-barcode subgraph simplification
- Gaps are too long to be bridged by short-read linked-read data — hybrid gap closure with PacBio CLR or Oxford Nanopore reads is the appropriate approach
- Goal is to correct the shared assembly graph itself — an edge erroneous for one barcode's isoform may be correct for another's, so edits must never be applied to the base graph

## Inputs

- Barcoded RNA-seq linked reads grouped into read clouds (per-barcode), after error correction (e.g., BayesHammer for Illumina data)
- Shared assembly graph built with spades-gbuilder (e.g., exportable in GFA format)
- Per-barcode subgraphs extracted from the shared assembly graph, with barcode-specific edge coverage computed along sequence-of-edge paths

## Outputs

- Per-barcode simplified subgraphs with gaps closed across short low-coverage edges (shared assembly graph left unmodified)
- Simple isoform paths extracted from each simplified per-barcode subgraph

## How to apply

Build the shared assembly graph from error-corrected barcoded RNA-seq reads with the SPAdes graph construction module (spades-gbuilder), then for each read cloud extract the barcode-specific subgraph, leaving the base graph unmodified. Compute barcode-specific edge coverage along sequence-of-edge paths in each subgraph. After clipping low-coverage tips (dead-end and dead-start edges) and removing bulges by comparing barcode coverage of parallel edges connecting the same node pairs, close gaps by joining edges across short low-coverage edges within the subgraph. Rationale: an edge erroneous for one barcode's isoform may be correct for another barcode's isoform, so all simplification is confined to the per-barcode subgraph rather than the shared assembly graph. Finally, extract simple isoform paths from each simplified subgraph.

## Related tools

- **cloudrnaSPAdes** (Performs the per-read-cloud subgraph simplification, including gap closing across short low-coverage edges, after tip clipping and bulge removal) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes (spades-gbuilder)** (Builds the shared assembly graph from which barcode-specific subgraphs are extracted for gap closing) — http://cab.spbu.ru/software/spades/
- **Bandage** (Visual inspection of the per-barcode simplified subgraphs and their graph paths to confirm gap closure and isoform path continuity) — https://github.com/rrwick/Bandage

## Evaluation signals

- Each simplified per-barcode subgraph must yield extractable simple isoform paths consistent with the known isoform structure of the gene (validated on the simulated GYPC barcoded data set)
- The shared assembly graph must be byte-identical before and after per-barcode simplification — no edges removed from the base graph
- Every gap-closed region corresponds to a short low-coverage edge that was bypassed, and resulting paths remain continuous (no remaining dead-end/dead-start tips after tip clipping)
- Parallel edges (bulges) between the same node pairs were resolved by comparing barcode-specific coverage, not global coverage
- Visual inspection of subgraphs in Bandage shows joined isoform paths across previously interrupted short edges

## Limitations

- Gap closing only bridges short low-coverage edges within barcode subgraphs; longer gaps require long-read data (PacBio CLR or Oxford Nanopore) for gap closure and repeat resolution
- Low barcode-specific coverage tips and edges may carry genuine low-abundance isoform signal, so aggressive clipping/closing risks losing rare isoforms in genes with high isoform diversity
- The method is confined to barcoded RNA-seq linked-read data processed per read cloud; it does not operate on the shared assembly graph or on unbarcoded data

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [methods] SPAdes will use PacBio CLR and Oxford Nanopore reads for gap closure and repeat resolution.: "SPAdes will use PacBio CLR and Oxford Nanopore reads for gap closure and repeat resolution."

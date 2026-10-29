---
name: sam-alignment-file-processing
description: Use when alignment reads are supplied as SAM files (collapsed or uncollapsed)
  and must be processed by small RNA counting tools, or when comparing counting runtimes
  across tools on the same alignment inputs.
when_to_use_negative:
- The input is already a feature count table rather than an alignment file — counting/benchmarking
  on SAM files is unnecessary
- Reads are supplied as unaligned FASTQ and the goal is upstream preprocessing/collapsing
  rather than counting from existing alignments
- The analysis goal is hierarchical feature classification (e.g. resolving piRNAs
  vs siRNAs from the same locus) rather than runtime comparison or plain SAM-file
  counting
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: tiny-count
  role: Precision counting tool run on collapsed and uncollapsed SAM files; quantifies
    multiple small RNA classes in parallel and is benchmarked for wall-clock runtime
  repo: github.com/MontgomeryLab/tinyRNA
- name: HTSeq-count
  role: Comparison counting tool run on the same two SAM files with equivalent counting
    parameters to record baseline runtime
- name: featureCounts
  role: Comparison counting tool run on the same two SAM files; fastest in the reported
    benchmark (~2.4 s on collapsed SAM)
- name: tinyRNA
  role: End-to-end workflow that consumes SAM/alignment-derived data and coordinates
    counting steps; installed via ./setup.sh into the tinyrna conda environment
  repo: github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:36ff6ddd013f7aa86bfb53d3396bf113d22938812fa2e7400b135f5dee457b18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTSeq-count
  - Python
  - featureCounts
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# sam-alignment-file-processing

## Summary

Handles collapsed and uncollapsed SAM alignment files of small RNA-seq reads as inputs to counting tools (tiny-count, HTSeq-count, featureCounts), including wall-clock runtime benchmarking of counting performance across tools.

## When to use

Use when alignment reads are supplied as SAM files (collapsed or uncollapsed) and must be processed by small RNA counting tools, or when comparing counting runtimes across tools on the same alignment inputs.

## When NOT to use

- The input is already a feature count table rather than an alignment file — counting/benchmarking on SAM files is unnecessary
- Reads are supplied as unaligned FASTQ and the goal is upstream preprocessing/collapsing rather than counting from existing alignments
- The analysis goal is hierarchical feature classification (e.g. resolving piRNAs vs siRNAs from the same locus) rather than runtime comparison or plain SAM-file counting

## Inputs

- Collapsed SAM alignment file of small RNA-seq reads
- Uncollapsed SAM alignment file of small RNA-seq reads
- Reference feature annotation file (GFF3/GFF2/GTF) with ID or gene_id and Parent column 9 attributes

## Outputs

- Per-run wall-clock times for tiny-count, HTSeq-count, and featureCounts on each SAM input
- Runtime benchmark table (e.g. runtime_benchmark.csv) with tool, input type (collapsed vs uncollapsed), and runtime in seconds, plus the runtime ordering across tools

## How to apply

Load the collapsed and uncollapsed SAM alignment files together with the reference feature annotation (GFF3/GFF2/GTF with ID/gene_id/Parent tags) supplied as inputs. Run tiny-count on the uncollapsed SAM file first, then on the collapsed SAM file, recording wall-clock time for each counting run; tiny-count quantifies multiple small RNA classes in parallel from a single pass. Run HTSeq-count and featureCounts on the same two SAM files with equivalent counting parameters, recording wall-clock time per run. Assemble recorded times into a table with columns for tool, input type (collapsed vs uncollapsed), and runtime in seconds, then compute the runtime ordering across the three tools. The paper's reference point is that on collapsed SAM files tiny-count's performance approaches featureCounts (45.1 s vs 2.4 s), with absolute times being hardware-dependent.

## Related tools

- **tiny-count** (Precision counting tool run on collapsed and uncollapsed SAM files; quantifies multiple small RNA classes in parallel and is benchmarked for wall-clock runtime) — github.com/MontgomeryLab/tinyRNA
- **HTSeq-count** (Comparison counting tool run on the same two SAM files with equivalent counting parameters to record baseline runtime)
- **featureCounts** (Comparison counting tool run on the same two SAM files; fastest in the reported benchmark (~2.4 s on collapsed SAM))
- **tinyRNA** (End-to-end workflow that consumes SAM/alignment-derived data and coordinates counting steps; installed via ./setup.sh into the tinyrna conda environment) — github.com/MontgomeryLab/tinyRNA

## Examples

```
conda install -c bioconda -c conda-forge tiny-count; conda activate tinyrna
```

## Evaluation signals

- The runtime table contains one row per tool × input-type combination (tiny-count, HTSeq-count, featureCounts × collapsed, uncollapsed) with wall-clock times in seconds
- On collapsed SAM files the reported ordering is reproduced: featureCounts fastest (~2.4 s), tiny-count approaching it (~45.1 s), with HTSeq-count slower — noting hardware dependence
- tiny-count on the collapsed SAM file produces counts for multiple small RNA classes (miRNAs/isomiRs, piRNAs, siRNAs, tRNA/rRNA fragments) in a single run
- The SAM inputs and the GFF3/GTF annotation use matching, case-sensitive chromosome identifiers so features are counted against the correct references

## Limitations

- Absolute runtimes are hardware-dependent; the 45.1 s (tiny-count, collapsed) vs 2.4 s (featureCounts, collapsed) figures only establish an ordering, not portable absolute times
- tiny-count is slower than featureCounts (~19× on collapsed SAM in the reported benchmark), so pure speed is not its advantage — parallel quantification of multiple small RNA classes is
- Reference annotation files must follow column 9 attribute conventions (ID or gene_id, Parent tags); Parent tags with multiple values are not yet supported
- Installation issues may arise on Apple Silicon-based systems with stable releases; the development release addresses this

## Evidence

- [results] On SAM files containing collapsed reads, tiny-count's performance approaches that of featureCounts while quantifying multiple small RNA classes in parallel, at 45.1 s versus 2.4 s.: [evidence span withheld — anchor status: not_found]
- [abstract] tiny-count quantifies reads from genome- or transcript-aligned SAM inputs: "tiny-count can quantify reads aligned to a genome or directly to small RNA or transcript sequences."
- [readme] Reference annotation requirements (ID/Parent tags) from README: [evidence span withheld — anchor status: external_doc]
- [readme] Standalone tiny-count installation via Bioconda: [evidence span withheld — anchor status: external_doc]

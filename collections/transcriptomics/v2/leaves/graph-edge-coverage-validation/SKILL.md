---
name: graph-edge-coverage-validation
description: Use when running Tiglon's label-based dynamic path extension on a labeled
  splice graph and you need to confirm no junction (edge labeled 1, 2+, or 2-) was
  left unrecovered by the assembled transcript paths.
when_to_use_negative:
- Input is raw BAM alignments from Hisat2/Star rather than a built labeled splice
  graph with predicted paths — graph construction must happen first.
- Assessing transcript accuracy or expression quantification (e.g., against NCBI RefSeq
  on GRCh38) — this skill checks edge coverage completeness, not correctness of assembled
  transcripts.
- Long-read RNA-seq data (PacBio/ONT) — Tiglon is not compatible with long-read datasets.
edam_operation: http://edamontology.org/operation_0251
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Tiglon
  role: Genome-guided transcriptome assembler whose label-based dynamic path extension
    produces the transcript paths whose edge coverage is validated; also builds the
    labeled splice graph
  repo: github.com/yutingsdu/Tiglon
- name: Hisat2
  role: Aligner whose mappings label the splice-graph edges (label 2 means captured
    by all employed aligners); one of the inputs to graph construction
  repo: https://github.com/DaehwanKimLab/hisat2
- name: Star
  role: Second aligner whose mappings are integrated with Hisat2's to label splice-graph
    edges before path extraction and coverage validation
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:de97ea1029165728b49742b1b10cbcae5e6232ce43eaafa7b742ccddd1b75464
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hisat2
  - Star
  - Tiglon
  edam_operation: http://edamontology.org/operation_0251
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# graph-edge-coverage-validation

## Summary

Validation skill for Tiglon's label-based path extraction: verifies that every edge (splice junction) of the labeled splice graph is traversed by at least one recovered transcript path, i.e., the edge-coverage fraction equals 1.

## When to use

Use when running Tiglon's label-based dynamic path extension on a labeled splice graph and you need to confirm no junction (edge labeled 1, 2+, or 2-) was left unrecovered by the assembled transcript paths.

## When NOT to use

- Input is raw BAM alignments from Hisat2/Star rather than a built labeled splice graph with predicted paths — graph construction must happen first.
- Assessing transcript accuracy or expression quantification (e.g., against NCBI RefSeq on GRCh38) — this skill checks edge coverage completeness, not correctness of assembled transcripts.
- Long-read RNA-seq data (PacBio/ONT) — Tiglon is not compatible with long-read datasets.

## Inputs

- Labeled splice graph of a gene locus (nodes = exons, edges = junctions with labels 1, 2+, 2-)
- Predicted transcript paths / recovered transcripts in GTF format (e.g., recovered_transcripts.gtf)

## Outputs

- Edge-coverage fraction (fraction of splice-graph edges traversed by at least one predicted transcript path; target value = 1)
- List of any uncovered edges (if coverage < 1)

## How to apply

After Tiglon extracts transcript-representing paths from the labeled splice graph (edges labeled 1, 2+, or 2- according to whether all employed aligners such as Hisat2 and Star captured the junction), map each predicted transcript path back onto the graph nodes (exons) and edges (junctions). Compute the fraction of graph edges covered by at least one predicted path; the success criterion is that this fraction equals 1. Coverage is achieved because Tiglon's path extraction iterates by descending label priority (2+ paired paths, then 2- paired paths, 2+ edges, 2- edges, label-1 elements), applies the fmin weight update and resets traversed edge labels to 1 after each extracted path, and repeats until all edges are covered. Only paths passing the default output filters (min_trans_cov > 0, min_trans_length 500) count toward coverage of the recovered_transcripts.gtf output.

## Related tools

- **Tiglon** (Genome-guided transcriptome assembler whose label-based dynamic path extension produces the transcript paths whose edge coverage is validated; also builds the labeled splice graph) — github.com/yutingsdu/Tiglon
- **Hisat2** (Aligner whose mappings label the splice-graph edges (label 2 means captured by all employed aligners); one of the inputs to graph construction) — https://github.com/DaehwanKimLab/hisat2
- **Star** (Second aligner whose mappings are integrated with Hisat2's to label splice-graph edges before path extraction and coverage validation) — https://github.com/alexdobin/STAR

## Examples

```
tiglon -i aligner_list.txt -o recovered_transcripts.gtf --min_trans_cov 0 --min_trans_length 500
```

## Evaluation signals

- Edge-coverage fraction equals 1: every edge (junction) of the labeled splice graph is traversed by at least one predicted transcript path.
- All emitted transcripts satisfy the default output thresholds: min_trans_cov > 0 and min_trans_length >= 500.
- Each recovered path is contiguous in the splice graph, using the Exp/Pre/Suc dynamic programming continuation choices (best predecessor/successor) at each node.
- Iterated label updates are consistent: after each extracted path, traversed edges receive the fmin weight update and their labels are reset to 1, so lower-priority elements become eligible in subsequent iterations.

## Limitations

- Coverage validation confirms completeness of edge traversal, not biological correctness — a fully covered graph can still contain incorrect transcript structures.
- Paths filtered out by min_trans_cov/min_trans_length thresholds may leave edges uncovered, so very short or lowly expressed junctions can evade recovery.
- The current Tiglon version lacks annotation-guided assembly and long-read (PacBio/ONT) support, so coverage is measured on de novo graph paths only.

## Evidence

- [intro] Tiglon employs a newly developed labeled-based dynamic path extension program to recover the expressed transcripts accurately.: "Tiglon employs a newly developed labeled-based dynamic path extension program to recover the expressed transcripts accurately."
- [intro] Tiglon further labels the edges (junctions) with 1 and 2, where label 2 indicates that they are captured by all the employed aligners and label 1 otherwise.: "Tiglon further labels the edges (junctions) with 1 and 2, where label 2 indicates that they are captured by all the employed aligners and label 1 otherwise."
- [abstract] followed by a label-based dynamic path-searching strategy to reconstruct the transcripts: "followed by a label-based dynamic path-searching strategy to reconstruct the transcripts"
- [methods] --min_trans_length/-L <int>   : Minimum assembled transcript length, default: 500.: "--min_trans_length/-L <int>    	: Minimum assembled transcript length, default: 500."
- [readme] Tiglon employs a dynamic programming algorithm to recover the transcripts by making strategic use of the label-information.: "Tiglon employs a dynamic programming algorithm to recover the transcripts by making strategic use of the label-information."

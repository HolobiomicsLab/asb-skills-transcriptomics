---
name: go-term-recovery-jaccard-scoring
description: Use when comparing unsupervised gene program/module discovery outputs
  (e.g., SSN gene modules, ICA components, NMF factors) for their ability to recover
  known functional annotations such as GO terms or PPI enrichments.
when_to_use_negative:
- 'When scoring cell-type marker sets against clusters for annotation (use the Miko
  scoring pipeline: nullScore, mikoScore, sigScore — a different, hypothesis-testing
  workflow)'
- When the input is already an annotated gene set with known GO labels rather than
  an unsupervised program output to be validated
- When the goal is per-gene differential expression testing (e.g., CDI-based bDEG
  identification) rather than gene-set-level recovery benchmarking
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: scMiko
  role: R package supplying getAnnotationPathways for GO retrieval and the SSN/ICA/NMF
    program-discovery functions whose outputs are scored
  repo: https://github.com/NMikolajewicz/scMiko
- name: STRING
  role: Protein-protein interaction reference database used for PPI enrichment testing
    of gene programs
- name: Seurat
  role: Framework on which scMiko is built; hosts the scRNA-seq objects from which
    program gene sets are derived
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:a7194bbfc1ff6878d98c9f550eef71c2d1a6c0f7e1f16baa74a4b57b3f52fab3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - STRING
  - Seurat
  - scMiko
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# go-term-recovery-jaccard-scoring

## Summary

Benchmarks gene program discovery methods (SSN vs ICA vs NMF) by scoring each program's GO-term recovery as the maximum column-wise Jaccard similarity between the program's gene set and curated GO terms, together with STRING PPI interaction enrichment. This operationalizes the scPipeline/scMiko claim that scale-free SSN programs recover known biology better than factorization-based methods.

## When to use

Use when comparing unsupervised gene program/module discovery outputs (e.g., SSN gene modules, ICA components, NMF factors) for their ability to recover known functional annotations such as GO terms or PPI enrichments.

## When NOT to use

- When scoring cell-type marker sets against clusters for annotation (use the Miko scoring pipeline: nullScore, mikoScore, sigScore — a different, hypothesis-testing workflow)
- When the input is already an annotated gene set with known GO labels rather than an unsupervised program output to be validated
- When the goal is per-gene differential expression testing (e.g., CDI-based bDEG identification) rather than gene-set-level recovery benchmarking

## Inputs

- Gene program gene sets per method (gene lists from SSN modules, ICA components, or consolidated NMF factors)
- GO annotation pathways retrieved via getAnnotationPathways
- STRING PPI interaction database for enrichment testing

## Outputs

- Per-program GO-term recovery score (maximum column-wise Jaccard similarity)
- Per-program STRING PPI enrichment statistics
- Per-method aggregated comparison table (ssn_benchmark_results.csv) supporting Fig. 5c-e

## How to apply

For every gene program produced by each method (SSN modules from runSSN/pruneSSN, ICA gene sets from getICAGenes, consolidated NMF factors from getNMFGenes/consolidateNMF), retrieve GO annotations with getAnnotationPathways. Compute GO-term recovery as the maximum column-wise Jaccard similarity between the program's gene set and the GO term gene sets, so each program is credited by its single best-matching GO term. In parallel, test enrichment of STRING PPI interactions within each program to assess physical/functional coherence beyond ontology overlap. Aggregate recovery and PPI-enrichment statistics per method into a comparison table (ssn_benchmark_results.csv) and evaluate whether scale-free SSN programs outperform the SN (scale_free=FALSE), ICA, and NMF comparators, as reported for Fig. 5c-e.

## Related tools

- **scMiko** (R package supplying getAnnotationPathways for GO retrieval and the SSN/ICA/NMF program-discovery functions whose outputs are scored) — https://github.com/NMikolajewicz/scMiko
- **STRING** (Protein-protein interaction reference database used for PPI enrichment testing of gene programs)
- **Seurat** (Framework on which scMiko is built; hosts the scRNA-seq objects from which program gene sets are derived)

## Examples

```
go.pathways <- getAnnotationPathways(gene.programs = ssn.programs); recovery <- max(jaccard(ssn.programs, go.pathways))
```

## Evaluation signals

- Each program's recovery score equals the maximum (not mean) column-wise Jaccard similarity against the GO term matrix
- All programs from every method (SSN, SN, ICA, NMF) are scored on identical network features so the benchmark is comparable across methods
- Aggregated statistics reproduce the expected result: SSN programs show significantly superior GO term recovery and STRING PPI enrichment relative to ICA and NMF (Fig. 5c-e)
- The comparison table ssn_benchmark_results.csv is produced with one row set per method containing recovery and PPI-enrichment statistics

## Limitations

- Marker/annotation quality is bounded by the reference databases used: 'marker-based annotations are only as good as the cell-type prescribed to the original dataset'
- Maximum-Jaccard recovery is a best-match overlap metric; it does not by itself establish statistical significance of ontology enrichment for a program
- The benchmark's conclusion (SSN superiority) is specific to the comparator set tested (SN without scale-free transform, ICA, NMF) and the STRING/GO references employed

## Evidence

- [discussion] gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods: "gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods"
- [full_text] SSN gene programs showed significantly superior GO term recovery and STRING PPI enrichment compared to ICA and NMF: [evidence span withheld — anchor status: not_found]
- [full_text] compute GO-term recovery as the maximum column-wise Jaccard similarity: [evidence span withheld — anchor status: not_found]
- [discussion] SSN analysis to identify and functionally annotate gene sets: "we describe SSN analysis as an approach to identify and functionally annotate gene sets in an unsupervised manner"

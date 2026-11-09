---
name: genomic-coordinate-format-unification
description: Use when merging already annotated miRNAs from heterogeneous databases
  (miRBase, PmiREN, sRNAanno, PsRNA) with novel sRNA-Seq-derived precursor predictions
  that use inconsistent coordinate representations.
when_to_use_negative:
- sRNA-Seq data from species without reference genome sequences — reads that do not
  map to the reference genome are ignored, so coordinate unification against a genome
  is impossible
- Downstream candidate selection — use the miRNASelection / manualCuration modules
  once the uniform candidate list exists
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Provides the miRNATranslate module that converts annotated and predicted miRNAs
    into a uniform genomic coordinate system before merging
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Framework in which miRNATranslate and the surrounding Compilation workflow
    are organized with user-friendly web interfaces
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:146f5e541b0d1c68893721c35dfda1d8397a96e5a802bd2e40378ee98bccfcac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Genomic coordinate format unification

## Summary

Harmonizes miRNA annotations retrieved from multiple public databases and newly predicted precursors into a single uniform genomic coordinate system (GFF3), enabling their merger into one candidate precursor list in iwa-miRNA.

## When to use

Use when merging already annotated miRNAs from heterogeneous databases (miRBase, PmiREN, sRNAanno, PsRNA) with novel sRNA-Seq-derived precursor predictions that use inconsistent coordinate representations.

## When NOT to use

- sRNA-Seq data from species without reference genome sequences — reads that do not map to the reference genome are ignored, so coordinate unification against a genome is impossible
- Downstream candidate selection — use the miRNASelection / manualCuration modules once the uniform candidate list exists

## Inputs

- Output from miRNARetrival (already annotated miRNAs from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0)
- Output from miRNAPredict (novel miRNA precursors predicted from sRNA-Seq data)
- Formatted reference genome sequences and annotation from genomePrepare

## Outputs

- miRNA and miRNA precursors with a uniform genomic coordinate format
- Combined GFF3-format list of candidate precursor precursors (e.g., 667 precursors / 1190 mature candidates for A. thaliana)

## How to apply

After aggregating annotated precursors with miRNARetrival and predicting novel precursors with miRNAPredict (which maps sRNA-Seq reads to the reference genome with GMAP), run miRNATranslate on the outputs of both steps. miRNATranslate converts annotated and predicted miRNAs and their precursors into a uniform genomic coordinate format so that redundant entries across the four databases and new predictions can be reconciled. The translated, uniformly formatted candidates are then merged into a single candidate precursor list (GFF3 for the A. thaliana benchmark: 667 precursors / 1190 mature candidates), and counts overlapping each source database are tallied. The rationale is that database-specific naming and coordinate conventions would otherwise prevent a combined, non-redundant candidate set from being assembled.

## Related tools

- **iwa-miRNA** (Provides the miRNATranslate module that converts annotated and predicted miRNAs into a uniform genomic coordinate system before merging) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Framework in which miRNATranslate and the surrounding Compilation workflow are organized with user-friendly web interfaces) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- The combined output file exists and is in valid GFF3 format
- For the A. thaliana benchmark, merging yields 667 precursor candidates corresponding to 1190 mature candidates (365 annotated precursors/625 mature forms from the four databases plus 435 predicted precursors, 302 of them novel)
- Every annotated and predicted precursor is represented in the unified coordinate system with no database-specific formatting remaining
- Overlap tallies per source database can be computed against the merged candidate list

## Limitations

- The aggregated candidate list certainly contains false positive hits, which downstream MiRNA Selection (high-throughput criteria, one-class SVM) and Manual Curation must address
- Unification depends on a reference genome; iwa-miRNA ignores sequencing reads that do not map to the reference genome
- Further wet experiments are still needed to validate candidates of interest

## Evidence

- [methods] miRNATranslate Output from miRNARetrival and miRNAPredict miRNA and miRNA precursors with a uniform format: [evidence span withheld — anchor status: not_found]
- [results] This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2): "This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2)"
- [intro] iwa-miRNA not only provides functions for automatically incorporating miRNA annotations from four representative databases (i.e., miRBase, PmiREN, sRNAanno, and PsRNA): "iwa-miRNA not only provides functions for automatically incorporating miRNA annotations from four representative databases (i.e., miRBase, PmiREN, sRNAanno, and PsRNA)"
- [discussion] The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits: "The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits"
- [discussion] In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome.: "In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome."

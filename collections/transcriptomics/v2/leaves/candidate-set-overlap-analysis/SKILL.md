---
name: candidate-set-overlap-analysis
description: Use when you have a uniform-format list of already annotated plus newly
  predicted miRNA/precursor candidates and need to label promising candidates via
  parallel HT criteria and one-class SVM sets and compute their overlap.
when_to_use_negative:
- sRNA-Seq data comes from a species without a reference genome — iwa-miRNA cannot
  handle such data in the current version
- Input is raw sRNA-Seq FASTQ/SRA data rather than the uniform-format miRNATranslate
  output — miRNA compilation/translation must run first
- Many sequencing reads do not map to the reference genome — the current version ignores
  unmapped reads, so candidates from unmapped reads will be missed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: iwa-miRNA
  role: Provides the miRNASelection step that runs the HT criteria filter and one-class
    SVM with default parameters to label promising candidates
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Framework organizing the iwa-miRNA modules and their dependencies behind user-friendly
    web interfaces
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:2aa87cf7e15800fd3c20d36e1aed1e02e319a2ec85c35eb59c636ef8c93f55ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - iwa-miRNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# candidate-set-overlap-analysis

## Summary

Reconstructs the iwa-miRNA MiRNA Selection step: two candidate-selection mechanisms — a high-throughput (HT) criteria filter and a one-class SVM classifier — are run in parallel with default parameters on the aggregated miRNA precursor candidate list, and the overlap between the two labelled sets identifies candidates supported by both mechanisms.

## When to use

Use when you have a uniform-format list of already annotated plus newly predicted miRNA/precursor candidates and need to label promising candidates via parallel HT criteria and one-class SVM sets and compute their overlap.

## When NOT to use

- sRNA-Seq data comes from a species without a reference genome — iwa-miRNA cannot handle such data in the current version
- Input is raw sRNA-Seq FASTQ/SRA data rather than the uniform-format miRNATranslate output — miRNA compilation/translation must run first
- Many sequencing reads do not map to the reference genome — the current version ignores unmapped reads, so candidates from unmapped reads will be missed

## Inputs

- Uniform-format miRNA and precursor candidate list (output of miRNATranslate), combining already annotated miRNAs (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0) and newly predicted miRNAs from sRNA-Seq data
- Sequence-, structure-, and expression-based features of candidates (used by the one-class SVM classifier)

## Outputs

- First labelled candidate set passing high-throughput criteria (e.g., Ara-Set1, maize-Set1)
- Second labelled candidate set predicted as true positives by the one-class SVM (e.g., Ara-Set2, maize-Set2)
- Overlap records between the HT-criteria set and the SVM-predicted set
- Tab-delimited table of selected miRNA candidates (e.g., selected_mirna_candidates.tsv)

## How to apply

Load the uniform-format miRNA and precursor candidate list produced by miRNATranslate within the Galaxy-based iwa-miRNA framework. Run the miRNASelection step with default parameters so that two mechanisms label candidates in parallel: (1) the high-throughput (HT) criteria filter, which yields the first set (e.g., Ara-Set1 with 330 A. thaliana precursors), and (2) a one-class SVM classifier built on sequence-, structure-, and expression-based features, which predicts true positives as the second set (e.g., Ara-Set2 with 308 A. thaliana precursors). Compute the overlap between the HT-criteria set and the SVM-predicted set to identify candidates supported by both mechanisms. Export the two labelled sets and their overlap records as a tab-delimited table (e.g., selected_mirna_candidates.tsv), then validate by reproducing the reported benchmark precursor counts (A. thaliana: 330/308; maize: 588/704) and confirming every candidate belongs to at least one set with the overlap identifiable.

## Related tools

- **iwa-miRNA** (Provides the miRNASelection step that runs the HT criteria filter and one-class SVM with default parameters to label promising candidates) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Framework organizing the iwa-miRNA modules and their dependencies behind user-friendly web interfaces) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- For the A. thaliana benchmark, the HT-criteria set contains 330 precursors (Ara-Set1) and the one-class SVM set contains 308 precursors (Ara-Set2)
- For the maize benchmark, the HT-criteria set contains 588 precursors (maize-Set1) and the one-class SVM set contains 704 precursors (maize-Set2)
- The output table is tab-delimited and contains the two labelled sets whose overlap records are identifiable
- Every candidate in the input list is assigned to at least one of the two sets
- Both selection mechanisms were run with default parameters, as in the article's workflow

## Limitations

- The aggregated candidate list certainly contains false positive hits, and further wet experiments are required to validate miRNA candidates before functional investigation
- iwa-miRNA ignores sequencing reads that do not map to the reference genome in the current version
- It cannot handle sRNA-Seq data from species without genome sequences

## Evidence

- [results] Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates: "Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates"
- [results] Ara-Set1 counts: "There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1)"
- [results] Ara-Set2 counts: "iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives"
- [methods] module input/output definition: [evidence span withheld — anchor status: not_found]
- [readme] three-module structure: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."

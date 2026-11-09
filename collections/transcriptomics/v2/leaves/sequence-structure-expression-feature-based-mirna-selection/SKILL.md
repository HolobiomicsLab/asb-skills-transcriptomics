---
name: sequence-structure-expression-feature-based-mirna-selection
description: Use when you have a uniform-format list of miRNA/precursor candidates
  (output from miRNATranslate in iwa-miRNA's Galaxy framework) and need to prioritize
  true positives before manual curation.
when_to_use_negative:
- sRNA-Seq data are from a species without a reference genome — iwa-miRNA cannot handle
  such data in the current version
- Candidate miRNAs must be discovered from reads that do not map to the reference
  genome — these reads are ignored by the current version
- The goal is downstream enrichment analysis (GO annotation, miRNA set enrichment)
  or cross-species comparative analysis of already selected miRNAs — these are out
  of scope for the MiRNA Selection module
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Provides the miRNASelection module implementing the HT-criteria filter and
    one-class SVM classifier on the miRNATranslate output
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Framework hosting the miRNASelection tool with web interfaces so the selection
    can be run without computational skills
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:4659e4498fa7483e7c56b15b496eba2e945ca8404f84c8bb084baff1805b1820
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sequence-structure-expression-feature-based-mirna-selection

## Summary

The MiRNA Selection module of iwa-miRNA screens an aggregated list of annotated and newly predicted miRNA precursors using two parallel mechanisms — a high-throughput (HT) criteria filter and a one-class SVM classifier built on sequence, structure (MFE/AMFE), and expression features from sRNA-Seq data — each run with default parameters to label promising candidates (e.g., Ara-Set1 and Ara-Set2).

## When to use

Use when you have a uniform-format list of miRNA/precursor candidates (output from miRNATranslate in iwa-miRNA's Galaxy framework) and need to prioritize true positives before manual curation.

## When NOT to use

- sRNA-Seq data are from a species without a reference genome — iwa-miRNA cannot handle such data in the current version
- Candidate miRNAs must be discovered from reads that do not map to the reference genome — these reads are ignored by the current version
- The goal is downstream enrichment analysis (GO annotation, miRNA set enrichment) or cross-species comparative analysis of already selected miRNAs — these are out of scope for the MiRNA Selection module

## Inputs

- Uniform-format miRNA and miRNA precursor candidate list (output from miRNATranslate, which combines miRNARetrival of annotated miRNAs and miRNAPredict results from sRNA-Seq data)
- Sequence-, structure-, and expression-based features of candidates (sRNA-Seq read support, MFE/AMFE descriptors)
- Default parameters of the HT criteria and the one-class SVM classifier

## Outputs

- Set of candidates passing high-throughput criteria (e.g., Ara-Set1: 330 A. thaliana precursors)
- Set of candidates predicted true by the one-class SVM (e.g., Ara-Set2: 308 A. thaliana precursors)
- Tab-delimited table of the two labelled candidate sets and their overlap records (e.g., selected_mirna_candidates.tsv)
- Input for downstream manualCuration and exploratory analysis of selected miRNAs

## How to apply

Load the uniform-format miRNA and precursor candidate list produced by miRNATranslate. Apply the high-throughput criteria filter with default parameters to label a first set of promising candidates (for A. thaliana this yielded 330 precursors, Ara-Set1). In parallel, build and run a one-class SVM classifier with default parameters on sequence-, structure-, and expression-based features to predict a second set of candidates as true positives (308 precursors for A. thaliana, Ara-Set2). Compute the overlap between the two labelled sets so users can prioritize candidates supported by both mechanisms, and export the results as a tab-delimited table before proceeding to manualCuration. The rationale is that the aggregated candidate list contains false positives, so dual computational filtering narrows the candidates requiring interactive manual curation.

## Related tools

- **iwa-miRNA** (Provides the miRNASelection module implementing the HT-criteria filter and one-class SVM classifier on the miRNATranslate output) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Framework hosting the miRNASelection tool with web interfaces so the selection can be run without computational skills) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- For the A. thaliana benchmark, the HT-criteria set contains 330 precursor candidates (Ara-Set1) and the one-class SVM set contains 308 precursor candidates (Ara-Set2)
- The output tab-delimited table contains exactly two labelled sets, and every candidate is assigned to at least one set with the overlap identifiable
- Both selection mechanisms were run with default parameters (HT criteria and one-class SVM), as in the article's workflow (Figure 1B)
- The selected-miRNA table feeds the manualCuration module and summary/report pages without format errors
- Analogous benchmarks exist for other species: maize-Set1 = 588 and maize-Set2 = 704 precursors

## Limitations

- The aggregated candidate list certainly contains false positive hits; wet-lab validation (e.g., qRT-PCR) is still required before functional investigation
- iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- Downstream analysis of selected miRNAs is limited — no GO annotation, miRNA set enrichment, or cross-species comparative analysis; a new miRNA discovery module for unmapped reads is planned but not yet implemented

## Evidence

- [methods] MiRNA Selection miRNASelection Output from miRNATranslate Selected miRNAs: [evidence span withheld — anchor status: not_found]
- [results] Both high-throughput criteria and one-class SVM with default parameters were used to identify promising miRNA candidates: "Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates"
- [results] There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1): "There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1)"
- [results] iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives: "iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives"
- [readme] The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation.: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."
- [discussion] The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits: "The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits"

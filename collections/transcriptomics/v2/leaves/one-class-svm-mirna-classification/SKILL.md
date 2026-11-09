---
name: one-class-svm-mirna-classification
description: Use when a uniform-format list of annotated plus newly predicted miRNA
  precursors (output of miRNATranslate) must be filtered into promising candidates
  using the one-class SVM classifier of iwa-miRNA's miRNASelection step with default
  parameters.
when_to_use_negative:
- Input sRNA-Seq data come from a species without a reference genome — iwa-miRNA cannot
  handle sRNA-Seq data from species without genome sequences.
- You only need candidate prediction, not filtering — use miRNAPredict; the one-class
  SVM belongs to the downstream miRNASelection step.
- Candidates require functional validation for investigation — the SVM label is computational;
  wet experiments are still required before functional investigation.
tools:
- name: iwa-miRNA
  role: Framework whose MiRNA Selection module builds and runs the one-class SVM classifier
    on candidate miRNA precursors
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNASelection
  role: Tool that takes the miRNATranslate output and applies the one-class SVM (and
    HT criteria) to output selected miRNAs
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Platform within which the miRNASelection/one-class SVM step is organized and
    executed
  repo: https://github.com/cma2015/iwa-miRNA
- name: Docker
  role: Container image for reproducible deployment of the iwa-miRNA stack including
    the SVM-based selection step
  repo: https://hub.docker.com/r/malab/iwa-mirna
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:58578ffc5585215f87f2b88e184d37ed504abfd8f6b160812d2c4eb33e15dc04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Docker
  - Galaxy
  - PmiREN
  - PsRNA
  - RefSeq
  - SRA
  - genomePrepare
  - iwa-miRNA
  - miRBase
  - miRNAPredict
  - miRNARetrival
  - miRNASelection
  - miRNATranslate
  - psRNAtarget
  - sRNAanno
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# one-class-svm-mirna-classification

## Summary

Within the iwa-miRNA MiRNA Selection module, a one-class SVM classifier trained on sequence-, structure-, and expression-based features labels miRNA precursor candidates (e.g., Ara-Set2, maize-Set2, wheat-Set2) as true positives, run in parallel with high-throughput (HT) criteria filtering to reduce false positives in the aggregated candidate list.

## When to use

Use when a uniform-format list of annotated plus newly predicted miRNA precursors (output of miRNATranslate) must be filtered into promising candidates using the one-class SVM classifier of iwa-miRNA's miRNASelection step with default parameters.

## When NOT to use

- Input sRNA-Seq data come from a species without a reference genome — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences.
- You only need candidate prediction, not filtering — use miRNAPredict; the one-class SVM belongs to the downstream miRNASelection step.
- Candidates require functional validation for investigation — the SVM label is computational; wet experiments are still required before functional investigation.

## Inputs

- Uniform-format miRNA and precursor candidate list from miRNATranslate (annotated + newly predicted precursors)
- Default parameters for the miRNASelection module (one-class SVM and HT criteria)

## Outputs

- Set2: miRNA precursor candidates predicted as true positives by the one-class SVM (e.g., Ara-Set2 = 308, maize-Set2 = 704, wheat-Set2 = 1410)
- Set1/Set2 overlap records
- selected_mirna_candidates.tsv (tab-delimited table of labelled candidate sets)

## How to apply

Take the uniform-format miRNA and precursor candidate list produced by miRNATranslate (which merges miRNARetrival database annotations with miRNAPredict sRNA-Seq-derived precursors) inside the Galaxy-based iwa-miRNA framework. Run miRNASelection so that the one-class SVM classifier is built and applied with default parameters to sequence-, structure-, and expression-based features, labelling a second candidate set (Set2) of true positives alongside the high-throughput criteria set (Set1). Because aggregation of already annotated and newly predicted miRNAs certainly contains false positive hits, this classifier serves as a principled filter that captures candidates the simple HT criteria may miss. Record the Set2 counts and compute the overlap with Set1 (e.g., A. thaliana: 330 in Ara-Set1, 308 in Ara-Set2, 208 shared), then export both labelled sets and their overlap as a tab-delimited table (selected_mirna_candidates.tsv) that feeds manualCuration for report generation.

## Related tools

- **iwa-miRNA** (Framework whose MiRNA Selection module builds and runs the one-class SVM classifier on candidate miRNA precursors) — https://github.com/cma2015/iwa-miRNA
- **miRNASelection** (Tool that takes the miRNATranslate output and applies the one-class SVM (and HT criteria) to output selected miRNAs) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Platform within which the miRNASelection/one-class SVM step is organized and executed) — https://github.com/cma2015/iwa-miRNA
- **Docker** (Container image for reproducible deployment of the iwa-miRNA stack including the SVM-based selection step) — https://hub.docker.com/r/malab/iwa-mirna

## Evaluation signals

- For A. thaliana Columbia ecotype candidates, the SVM-labelled set (Ara-Set2) contains 308 precursors and the HT set (Ara-Set1) 330, with an identifiable 208-member overlap.
- For maize (B73), maize-Set2 contains 704 precursors and maize-Set1 588, giving 886 total selected candidates.
- For wheat, wheat-Set2 contains 1410 precursors and wheat-Set1 1617, within the 2030 selected precursors (2163 loci).
- selected_mirna_candidates.tsv contains two labelled sets plus overlap records, and every candidate is assigned to at least one set with the overlap identifiable.
- The one-class SVM was run with default parameters on sequence-, structure-, and expression-based features, as in the reported benchmark behavior.

## Limitations

- The aggregated candidate list certainly contains false positive hits, so SVM labels are not definitive; further wet experiments should validate candidates of particular interest before any functional investigation.
- iwa-miRNA ignores sequencing reads that do not map to the reference genome, so candidates derived from unmapped reads are never fed to the classifier.
- The framework cannot handle sRNA-Seq data from species without genome sequences, limiting SVM-based selection to species with a reference genome.

## Evidence

- [results] Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates: "Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates"
- [results] SVM predicts 308 A. thaliana precursors as true positives: "iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives"
- [methods] miRNASelection takes miRNATranslate output and produces selected miRNAs: [evidence span withheld — anchor status: not_found]
- [discussion] Aggregated candidate list contains false positives, motivating SVM filtering: "The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits"

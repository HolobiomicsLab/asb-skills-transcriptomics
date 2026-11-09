---
name: high-throughput-criteria-filtering-mirna
description: Use when a uniform-format list of annotated + newly predicted plant miRNA
  precursor candidates (output of miRNATranslate) must be filtered with the HT criteria
  to label promising candidates (Set1) inside the iwa-miRNA/Galaxy MiRNA Selection
  module.
when_to_use_negative:
- The input is raw sRNA-Seq data (SRA accessions or FASTQ) — this must first go through
  genomePrepare and miRNAPredict; the HT filter operates only on the translated candidate
  list
- The sRNA-Seq data come from a species without a reference genome or contain reads
  that do not map to the reference — iwa-miRNA cannot handle these cases, so the HT
  filter cannot be applied meaningfully
- The goal is de novo miRNA discovery from unmapped reads — this is planned but not
  implemented in the current version
edam_operation: http://edamontology.org/operation_0226
tools:
- name: iwa-miRNA
  role: Framework providing the miRNASelection module in which the HT criteria filter
    is applied to the miRNATranslate output with default parameters
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Web interface in which the miRNASelection step (HT criteria + one-class SVM)
    is organized and executed
- name: Rmarkdown
  role: Used to summarize selection results into HTML pages for visualization, interpretation,
    and sharing
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:e785582054af4238dbd59538c1014ff0b0a227dfbd77ced87e2d7878d1981a13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - genomePrepare
  - iwa-miRNA
  - miRNAPredict
  - miRNARetrival
  - miRNASelection
  - miRNATranslate
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# high-throughput-criteria-filtering-mirna

## Summary

The high-throughput (HT) criteria filter of iwa-miRNA's miRNASelection module, which labels promising miRNA precursor candidates (Set1-type outputs such as Ara-Set1 and maize-Set1) from the uniform-format candidate list, run in parallel with a one-class SVM classifier.

## When to use

Use when a uniform-format list of annotated + newly predicted plant miRNA precursor candidates (output of miRNATranslate) must be filtered with the HT criteria to label promising candidates (Set1) inside the iwa-miRNA/Galaxy MiRNA Selection module.

## When NOT to use

- The input is raw sRNA-Seq data (SRA accessions or FASTQ) — this must first go through genomePrepare and miRNAPredict; the HT filter operates only on the translated candidate list
- The sRNA-Seq data come from a species without a reference genome or contain reads that do not map to the reference — iwa-miRNA cannot handle these cases, so the HT filter cannot be applied meaningfully
- The goal is de novo miRNA discovery from unmapped reads — this is planned but not implemented in the current version

## Inputs

- Uniform-format miRNA and miRNA precursor candidate list produced by miRNATranslate (aggregated annotated + newly predicted candidates, e.g. 667 A. thaliana precursor / 1190 mature candidates)
- Default parameters of the miRNASelection HT criteria filter

## Outputs

- Labelled Set1 candidate list (precursors passing the high-throughput criteria)
- Tab-delimited table selected_mirna_candidates.tsv containing Set1, Set2 and their overlap records
- Stage counts (e.g. Set1/Set2/overlap) for downstream benchmark CSVs such as arabidopsis_benchmark_counts.csv or maize_miRNA_candidate_counts.csv

## How to apply

Run the miRNASelection tool on the uniform-format miRNA and precursor candidate list produced by miRNATranslate (which aggregates miRNARetrival and miRNAPredict outputs into a shared genomic coordinate system). Apply the HT criteria filter with default parameters — it is one of two parallel candidate-selection mechanisms in miRNASelection, the other being the one-class SVM classifier built on sequence-, structure- and expression-based features (e.g., MFE/AMFE-related attributes noted in the README abbreviations). Record the labelled HT-criteria set (Set1) alongside the SVM set (Set2), then compute the overlap to identify candidates supported by both mechanisms. Export the two labelled sets and their overlap as a tab-delimited table (selected_mirna_candidates.tsv). Benchmark expectations: 330 of 667 A. thaliana precursor candidates passed the HT criteria (Ara-Set1) and 588 maize precursors passed (maize-Set1), both with default parameters.

## Related tools

- **iwa-miRNA** (Framework providing the miRNASelection module in which the HT criteria filter is applied to the miRNATranslate output with default parameters) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Web interface in which the miRNASelection step (HT criteria + one-class SVM) is organized and executed)
- **Rmarkdown** (Used to summarize selection results into HTML pages for visualization, interpretation, and sharing)

## Evaluation signals

- For A. thaliana Columbia ecotype, the HT-criteria set must contain 330 precursor candidates (Ara-Set1) out of the 667 aggregated precursor candidates
- For maize B73, the HT-criteria set must contain 588 precursor candidates (maize-Set1), and the SVM set 704 (maize-Set2), with 886 combined candidates selected for downstream analysis
- Every candidate in selected_mirna_candidates.tsv must be assigned to at least one set (Set1 and/or Set2), with the overlap between the two sets identifiable (e.g. 208 A. thaliana precursors in both)
- Set counts reproduced from the workflow must equal the paper's reported values when written to the benchmark CSVs (output_matches_reference validation)

## Limitations

- The aggregated candidate list fed to the filter certainly contains false positive hits; further wet-lab validation (e.g. qRT-PCR) is still required before functional investigation
- HT filtering alone is insufficient — it is designed to run in parallel with the one-class SVM classifier, and consensus (overlap) is more informative than either set alone
- iwa-miRNA ignores sequencing reads that do not map to the reference genome, so candidates derivable only from unmapped reads are absent from the filtered set

## Evidence

- [methods] MiRNA Selection miRNASelection Output from miRNATranslate Selected miRNAs: [evidence span withheld — anchor status: not_found]
- [results] Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates: "Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates"
- [results] There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1): "There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1)"
- [readme] SVM: support vector machine; HT criteria: high-throughput criteria; MFE: minimal free energy; AMFE: adjusted minimal free energy: "SVM: support vector machine; HT criteria: high-throughput criteria;"
- [other] Build and run the one-class SVM classifier with default parameters on sequence-, structure-, and expression-based features: [evidence span withheld — anchor status: not_found]

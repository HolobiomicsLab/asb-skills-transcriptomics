---
name: pipeline-count-validation-against-reference
description: Use when reproducing a published pipeline (e.g., iwa-miRNA on maize B73)
  and you must confirm that intermediate counts (619 aggregated, 1241 predicted, 588/704
  selected, 886 total) equal reported values.
when_to_use_negative:
- The analysis is a novel exploration with no published reference counts to compare
  against — use exploratory QC instead
- Inputs differ from the publication (e.g., different genome build than B73, different
  number of sRNA-Seq datasets), making exact count equality an invalid expectation
- The pipeline is not count-deterministic and only distributions or ranges can be
  compared
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0797
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose maize run produces the counts being validated
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNARetrival
  role: Aggregates annotated miRNA precursors from four databases; its count (619)
    is a validation field
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNAPredict
  role: Predicts candidate precursors from sRNA-Seq data; its count (1241) is a validation
    field
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNASelection
  role: Applies HT criteria and one-class SVM to select candidates; produces maize-Set1
    (588) and maize-Set2 (704) counts
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNATranslate
  role: Converts miRNARetrival and miRNAPredict outputs to a uniform format upstream
    of selection
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Summarizes workflow results into HTML pages for visualization and reproducible
    reporting
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:175fe91034047be3857f2a42e3de3b29f3d6affae089edd00b33a2b0c6958689
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Rmarkdown
  - genomePrepare
  - iwa-miRNA
  - miRNAPredict
  - miRNARetrival
  - miRNASelection
  - miRNATranslate
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pipeline-count-validation-against-reference

## Summary

Validate that counts produced at each stage of a bioinformatics pipeline (e.g., iwa-miRNA miRNA precursor aggregation, prediction, and selection) match reference values reported in the source publication, using an output_matches_reference check on a summary CSV.

## When to use

Use when reproducing a published pipeline (e.g., iwa-miRNA on maize B73) and you must confirm that intermediate counts (619 aggregated, 1241 predicted, 588/704 selected, 886 total) equal reported values.

## When NOT to use

- The analysis is a novel exploration with no published reference counts to compare against — use exploratory QC instead
- Inputs differ from the publication (e.g., different genome build than B73, different number of sRNA-Seq datasets), making exact count equality an invalid expectation
- The pipeline is not count-deterministic and only distributions or ranges can be compared

## Inputs

- Counts written to maize_miRNA_candidate_counts.csv from each iwa-miRNA stage (miRNARetrival, miRNAPredict, miRNASelection)
- Reference count values reported in the publication: 619, 1241, 588, 704, 886
- Intermediate tool outputs (already annotated miRNAs from four databases, predicted miRNAs from 195 maize sRNA-Seq datasets, maize-Set1/maize-Set2 selected precursors)

## Outputs

- Validated maize_miRNA_candidate_counts.csv with per-stage counts matching the reference (619, 1241, 588, 704, 886)
- A pass/fail output_matches_reference verdict per count field

## How to apply

Run the iwa-miRNA workflow end to end: miRNARetrival with species 'Zea mays' aggregates precursors from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0; genomePrepare formats the B73 genome; miRNAPredict processes 195 maize SRA sRNA-Seq datasets; miRNATranslate converts outputs to a uniform genomic coordinate system on B73; miRNASelection applies high-throughput criteria (HT criteria) and a one-class SVM classifier with default parameters. Record counts at each stage into a summary file (maize_miRNA_candidate_counts.csv). Then perform an output_matches_reference validation: assert each field equals the reference values — 619 aggregated, 1241 predicted, 588 maize-Set1 (HT criteria), 704 maize-Set2 (SVM), and 886 total selected. Rationale: exact count reproduction is the primary check that the software version, parameters, and input datasets match those used in the publication (doi:10.1016/j.gpb.2021.02.010); discrepancies indicate version drift, changed inputs, or parameter deviations.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose maize run produces the counts being validated) — https://github.com/cma2015/iwa-miRNA
- **miRNARetrival** (Aggregates annotated miRNA precursors from four databases; its count (619) is a validation field) — https://github.com/cma2015/iwa-miRNA
- **miRNAPredict** (Predicts candidate precursors from sRNA-Seq data; its count (1241) is a validation field) — https://github.com/cma2015/iwa-miRNA
- **miRNASelection** (Applies HT criteria and one-class SVM to select candidates; produces maize-Set1 (588) and maize-Set2 (704) counts) — https://github.com/cma2015/iwa-miRNA
- **miRNATranslate** (Converts miRNARetrival and miRNAPredict outputs to a uniform format upstream of selection) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Summarizes workflow results into HTML pages for visualization and reproducible reporting) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- output_matches_reference: every count field in maize_miRNA_candidate_counts.csv equals the paper's value — 619 (aggregated), 1241 (predicted), 588 (maize-Set1), 704 (maize-Set2), 886 (total selected)
- maize-Set1 count (588) is strictly less than the aggregated+predicted union count, and total selected (886) equals the union of Set1 and Set2
- The four source databases (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0) each contributed (326, 221, 163, 142 precursors per the analogous A. thaliana reporting pattern), confirming miRNARetrival pulled from all four
- Predicted set derives from exactly 195 maize SRA sRNA-Seq datasets as specified in the workflow
- Results are reproducible when re-run from the released Docker image or Galaxy web server

## Limitations

- Aggregated candidate lists contain false positive hits; count equality alone does not guarantee biological correctness of individual precursors
- iwa-miRNA ignores sequencing reads that do not map to the reference genome, so counts are reference-build dependent
- Exact count matching requires the same software version (source released 2020-06-20; demo server 2020-06-24), same databases (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0), and same input datasets as the publication

## Evidence

- [results] 588 passed with the high-throughput criteria (maize-Set1): "588 passed with the high-throughput criteria (maize-Set1)"
- [results] 704 predicted by the one-class SVM classifier (maize-Set2): "704 predicted by the one-class SVM classifier (maize-Set2)"
- [results] Both high-throughput criteria and one-class SVM with default parameters: "Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates"
- [methods] miRNASelection output: [evidence span withheld — anchor status: not_found]
- [discussion] In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome.: "In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome."
- [readme] Source codes and Docker image of iwa-miRNA were released for the first time.: "2020-06-20: Source codes and Docker image of iwa-miRNA were released for the first time."

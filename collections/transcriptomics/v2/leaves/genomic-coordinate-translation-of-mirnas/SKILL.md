---
name: genomic-coordinate-translation-of-mirnas
description: Use when annotated miRNA records from public plant databases and newly
  predicted precursors from sRNA-Seq must be merged into one non-redundant candidate
  set on a common reference genome.
when_to_use_negative:
- Input sRNA-Seq data come from a species without a reference genome — iwa-miRNA cannot
  handle sRNA-Seq data from species without genome sequences
- Sequencing reads of interest do not map to the reference genome — the current version
  ignores unmapped reads
- Candidate selection is already done and only downstream analyses (e.g., GO enrichment,
  target prediction with psRNAtarget) are needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose MiRNA Compilation module hosts the coordinate-translation
    step
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNATranslate
  role: Tool that converts miRNARetrival and miRNAPredict outputs into a uniform genomic-coordinate
    format
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNARetrival
  role: Produces the annotated-miRNA side of the translation input by aggregating
    four public plant miRNA databases
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNAPredict
  role: Produces the predicted-precursor side of the translation input from SRA sRNA-Seq
    datasets
  repo: https://github.com/cma2015/iwa-miRNA
- name: genomePrepare
  role: Formats the reference genome sequences and annotation onto which miRNA records
    are translated
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Web platform in which miRNATranslate is exposed as a user-friendly workflow
    step
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:ea01740bd628728fdf770d99106acd47e17496c85fd5bd816f3b0bc47d445bec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - genomePrepare
  - iwa-miRNA
  - miRNAPredict
  - miRNARetrival
  - miRNASelection
  - miRNATranslate
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Genomic coordinate translation of miRNAs

## Summary

Convert the outputs of miRNARetrival (annotated miRNAs from miRBase, PmiREN, sRNAanno, PsRNA) and miRNAPredict (sRNA-Seq-derived candidates) into a uniform genomic-coordinate format so they can be merged and deduplicated into a single precursor candidate list (e.g., 667 A. thaliana candidates / 1190 mature candidates, 1241 maize predictions aggregated with 619 annotated precursors).

## When to use

Use when annotated miRNA records from public plant databases and newly predicted precursors from sRNA-Seq must be merged into one non-redundant candidate set on a common reference genome.

## When NOT to use

- Input sRNA-Seq data come from a species without a reference genome — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads of interest do not map to the reference genome — the current version ignores unmapped reads
- Candidate selection is already done and only downstream analyses (e.g., GO enrichment, target prediction with psRNAtarget) are needed

## Inputs

- Output from miRNARetrival (already annotated miRNAs from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0)
- Output from miRNAPredict (predicted miRNA precursors from sRNA-Seq data)
- Formatted genome sequences and annotation from genomePrepare (e.g., TAIR10 for A. thaliana, B73 for maize)

## Outputs

- miRNA and miRNA precursors with a uniform genomic-coordinate format
- Aggregated precursor/mature candidate table for downstream miRNASelection (e.g., A. thaliana: 667 precursor candidates / 1190 mature candidates, Table S2)

## How to apply

Run miRNATranslate after miRNARetrival (which aggregates already-annotated precursors from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0) and after miRNAPredict (which predicts precursors from SRA sRNA-Seq datasets, e.g., 435 from 1063 A. thaliana Columbia datasets or 1241 from 195 maize datasets). The tool consumes both outputs plus the formatted genome reference produced by genomePrepare and re-expresses every miRNA precursor and mature form in a uniform genomic-coordinate system on the species reference (e.g., TAIR10 for A. thaliana, B73 for maize). This step is essential because database records and de novo predictions use heterogeneous identifier and coordinate conventions, and MiRNA Selection (miRNASelection with high-throughput criteria and one-class SVM) requires a single translated candidate table. Verify by counting: for A. thaliana the translated aggregate should contain 667 precursor candidates corresponding to 1190 mature candidates (Table S2); for maize, 619 annotated precursors merged with 1241 predicted ones feed into selection.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose MiRNA Compilation module hosts the coordinate-translation step) — https://github.com/cma2015/iwa-miRNA
- **miRNATranslate** (Tool that converts miRNARetrival and miRNAPredict outputs into a uniform genomic-coordinate format) — https://github.com/cma2015/iwa-miRNA
- **miRNARetrival** (Produces the annotated-miRNA side of the translation input by aggregating four public plant miRNA databases) — https://github.com/cma2015/iwa-miRNA
- **miRNAPredict** (Produces the predicted-precursor side of the translation input from SRA sRNA-Seq datasets) — https://github.com/cma2015/iwa-miRNA
- **genomePrepare** (Formats the reference genome sequences and annotation onto which miRNA records are translated) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Web platform in which miRNATranslate is exposed as a user-friendly workflow step) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- A. thaliana benchmark: after translating miRNARetrival (365 precursors / 625 mature forms) and miRNAPredict (435 precursors) outputs, the uniform-format aggregate must equal 667 precursor candidates and 1190 mature candidates (Table S2)
- Maize benchmark: translation merges 619 aggregated annotated precursors with 1241 predicted precursors before miRNASelection (588 maize-Set1, 704 maize-Set2, 886 selected)
- Every translated precursor and mature form must be expressed in the coordinate system of the genome reference produced by genomePrepare (e.g., TAIR10 or B73)
- Precursors not previously annotated in any of the four databases (302 of 435 for A. thaliana) must remain distinguishable after translation, since selection reports the new-annotation fraction
- Downstream miRNASelection must accept the translated output without coordinate or identifier parsing errors

## Limitations

- The aggregated candidate list resulting from merging annotated and newly predicted miRNAs certainly contains false positive hits, so translated output still requires selection and manual curation
- Translation depends on a reference genome; sRNA-Seq data from species without genome sequences cannot be handled
- Reads that do not map to the reference genome are ignored in the current version, so unmapped-read miRNAs never enter the translated candidate set

## Evidence

- [results] miRNATranslate converts the outputs of miRNARetrival and miRNAPredict into a uniform genomic coordinate system on the B73 reference.: [evidence span withheld — anchor status: not_found]
- [methods] miRNATranslate takes outputs from miRNARetrival and miRNAPredict as input.: [evidence span withheld — anchor status: not_found]
- [results] A. thaliana translated aggregate of 667 precursor candidates corresponding to 1190 mature candidates.: "This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2)"
- [discussion] iwa-miRNA ignores sequencing reads that do not map to the reference genome.: "In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome."
- [discussion] The aggregation of annotated and newly predicted miRNAs contains false positive hits.: "The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits"

---
name: small-rna-seq-mirna-prediction
description: Use when you have SRA accession numbers or FASTQ files of small RNA sequencing
  datasets and a reference genome plus annotation for a plant species, and you need
  to predict miRNA precursors and select credible candidates genome-wide.
when_to_use_negative:
- The species has no reference genome — iwa-miRNA cannot handle sRNA-Seq data from
  species without genome sequences and ignores reads that do not map to the reference
- The goal is downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment)
  or cross-species comparative analysis of the selected miRNAs — the framework has
  limited downstream analysis capability
- Only a single small dataset with prior confident annotation exists and no genome-wide
  candidate selection is needed
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: iwa-miRNA
  role: Galaxy-based framework providing the miRNAPredict, miRNARetrival, genomePrepare,
    miRNATranslate, miRNASelection, and manualCuration tools used for sRNA-Seq-based
    miRNA prediction and candidate selection
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNAPredict
  role: Predicts miRNA precursors from SRA accession numbers or uploaded FASTQ files
    (sRNA-Seq data)
  repo: https://github.com/cma2015/iwa-miRNA
- name: genomePrepare
  role: Formats reference genome sequences and annotation for the species before prediction
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNATranslate
  role: Converts miRNARetrival and miRNAPredict outputs into a uniform genomic coordinate
    format
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRNASelection
  role: Selects promising candidates using high-throughput criteria and a one-class
    SVM classifier at default parameters
  repo: https://github.com/cma2015/iwa-miRNA
- name: manualCuration
  role: Generates Rmarkdown HTML summary/report pages for interactive manual curation
    of selected miRNAs
  repo: https://github.com/cma2015/iwa-miRNA
- name: psRNAtarget
  role: Predicts miRNA-target interactions depicted as bipartite networks in the report
    pages
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:1d360572d28f68b0925eac3a50b9ffaa943ba5d53fc8c857e275a6cc7a3e1b36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Docker
  - PmiREN
  - PsRNA
  - RefSeq
  - SRA
  - genomePrepare
  - iwa-miRNA
  - manualCuration
  - miRBase
  - miRNAPredict
  - miRNARetrival
  - miRNASelection
  - miRNATranslate
  - psRNAtarget
  - sRNAanno
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# small-rna-seq-mirna-prediction

## Summary

Predict plant miRNA precursors from sRNA-Seq datasets using the miRNAPredict tool of the Galaxy-based iwa-miRNA framework, then merge predictions with database-annotated precursors (miRBase, PmiREN, sRNAanno, PsRNA) via miRNATranslate and select promising candidates with high-throughput criteria and a one-class SVM in miRNASelection.

## When to use

Use when you have SRA accession numbers or FASTQ files of small RNA sequencing datasets and a reference genome plus annotation for a plant species, and you need to predict miRNA precursors and select credible candidates genome-wide.

## When NOT to use

- The species has no reference genome — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences and ignores reads that do not map to the reference
- The goal is downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment) or cross-species comparative analysis of the selected miRNAs — the framework has limited downstream analysis capability
- Only a single small dataset with prior confident annotation exists and no genome-wide candidate selection is needed

## Inputs

- SRA accession numbers or uploaded FASTQ files of small RNA sequencing (sRNA-Seq) datasets
- Reference genome sequences and genome annotation files (e.g., TAIR10 for A. thaliana, B73 for maize, IWGSC RefSeq v1.0 for wheat)
- Species name for aggregation of annotated miRNAs from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0

## Outputs

- Predicted miRNA precursors from miRNAPredict
- Already annotated miRNA precursors aggregated from the four public databases via miRNARetrival
- miRNA and miRNA precursors in a uniform genomic coordinate format from miRNATranslate (e.g., GFF3/GTF and FASTA exports)
- Selected miRNA candidate sets from miRNASelection (HT-criteria Set1 and one-class SVM Set2, plus union)
- Summary and report HTML pages from manualCuration (Rmarkdown) with secondary structure plots, read stacks, expression boxplots, and miRNA-target bipartite networks

## How to apply

First run miRNARetrival with the species name to aggregate already annotated precursors from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0, and run genomePrepare on the genome sequences and annotation to produce formatted reference files. Then run miRNAPredict on SRA accession numbers or uploaded FASTQ files to predict candidate miRNA precursors from the sRNA-Seq data (e.g., 1063 A. thaliana datasets yielded 435 precursors; 195 maize datasets yielded 1241; 95 wheat datasets contributed to 2857 aggregated candidates). Run miRNATranslate to convert the miRNARetrival and miRNAPredict outputs into a uniform genomic coordinate system on the reference. Finally run miRNASelection with default parameters, applying both the high-throughput (HT) criteria and a one-class SVM classifier to identify promising candidates (e.g., Ara-Set1 330 / Ara-Set2 308 for A. thaliana; maize-Set1 588 / maize-Set2 704). Optionally run manualCuration to generate Rmarkdown HTML summary and report pages with secondary structure plots, read stacks, expression boxplots, and psRNAtarget miRNA-target bipartite networks for interactive inspection.

## Related tools

- **iwa-miRNA** (Galaxy-based framework providing the miRNAPredict, miRNARetrival, genomePrepare, miRNATranslate, miRNASelection, and manualCuration tools used for sRNA-Seq-based miRNA prediction and candidate selection) — https://github.com/cma2015/iwa-miRNA
- **miRNAPredict** (Predicts miRNA precursors from SRA accession numbers or uploaded FASTQ files (sRNA-Seq data)) — https://github.com/cma2015/iwa-miRNA
- **genomePrepare** (Formats reference genome sequences and annotation for the species before prediction) — https://github.com/cma2015/iwa-miRNA
- **miRNATranslate** (Converts miRNARetrival and miRNAPredict outputs into a uniform genomic coordinate format) — https://github.com/cma2015/iwa-miRNA
- **miRNASelection** (Selects promising candidates using high-throughput criteria and a one-class SVM classifier at default parameters) — https://github.com/cma2015/iwa-miRNA
- **manualCuration** (Generates Rmarkdown HTML summary/report pages for interactive manual curation of selected miRNAs) — https://github.com/cma2015/iwa-miRNA
- **psRNAtarget** (Predicts miRNA-target interactions depicted as bipartite networks in the report pages)

## Examples

```
docker run malab/iwa-mirna iwa-miRNA --tool miRNAPredict --sra_accessions File_S1.txt --genome TAIR10.fa --annotation TAIR10.gff3 --output predicted_mirnas/
```

## Evaluation signals

- Benchmark counts match published values: A. thaliana aggregated 365 / predicted 435 / total 667 / Ara-Set1 330 / Ara-Set2 308 / overlap 208
- Maize counts match: 619 aggregated, 1241 predicted, 588 maize-Set1, 704 maize-Set2, 886 selected
- Wheat counts match: 2857 aggregated, 1617 wheat-Set1, 1410 wheat-Set2, 2030 selected precursors (2163 loci)
- Count of predicted precursors not previously annotated in the four databases is computed (302 of 435 for A. thaliana)
- Output tables (e.g., arabidopsis_benchmark_counts.csv, maize_miRNA_candidate_counts.csv, wheat_mirna_counts.csv) contain every required count field and each equals the reported reference value exactly

## Limitations

- Cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- The aggregated candidate list certainly contains false positive hits; further wet experiments are required to validate miRNA candidates before functional investigation
- Limited downstream analysis of selected miRNAs: no enrichment analyses such as GO annotation or miRNA set enrichment, and no cross-species comparative analysis

## Evidence

- [methods] miRNAPredict SRA accession numbers or uploaded fastq files Predicted miRNAs Predict miRNAs from sRNA-Seq data: [evidence span withheld — anchor status: not_found]
- [methods] miRNASelection: [evidence span withheld — anchor status: not_found]
- [results] Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors, 302 of which were not previously annotated: "Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors, 302 of which were not previously annotated"
- [results] iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives: "iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives"
- [readme] Source codes and user manual are available at https://github.com/cma2015/iwa-miRNA.: "Source codes and user manual are available at https://github.com/cma2015/iwa-miRNA."
- [discussion] It cannot handle sRNA-Seq data from species without genome sequences.: "It cannot handle sRNA-Seq data from species without genome sequences."

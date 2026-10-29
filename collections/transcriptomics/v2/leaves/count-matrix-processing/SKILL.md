---
name: count-matrix-processing
description: Use when a raw or simulated RNA-seq count table (genes as rows, samples
  as columns, with control/treatment conditions) must be validated and prepared as
  input for DEG detection in the ExpHunter Suite.
when_to_use_negative:
- Input is still raw FASTQ — the README explicitly requires a clean&map program to
  transform fastq files into count tables before DEgenes Hunter can be used
- Input is already DEG results or a ranked gene list rather than a table of counts
  — this skill covers the upstream count layer, not downstream consensus or enrichment
  steps
- The goal is co-expression module detection or functional enrichment without a count
  matrix, which are separate ExpHunter Suite modules
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose DEgenes Hunter module consumes the count table
    to run DESeq2, edgeR, and limma differential expression
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: pipeline executed on the count matrix and experimental design for the Lafora
    and PMM2-CDG experiments
  repo: https://github.com/fmjabato/ExpHunterSuiteExamples
- name: DESeq2
  role: one of the DEG detection methods run on the count matrix; in the PMM2-CDG
    experiment DESeq2 was by far the least conservative
- name: limma
  role: one of the DEG detection methods run on the count matrix; showed higher precision
- name: edgeR
  role: one of the three DEG detection methods executed on the count table by DEgenes
    Hunter
- name: Bioconductor
  role: ecosystem providing ExpHunterSuite and the TCC A. thaliana reference count
    dataset used by the simulation helper
  repo: https://github.com/seoanezonjic/ExpHunterSuite
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:0ca02f5a3875f2ae9e8592acc2805c57661b213c2115b422d99cb92fa0f63026
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - R
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# count-matrix-processing

## Summary

Preparation and handling of gene-by-sample RNA-seq count tables as the input layer of the ExpHunterSuite/DEgenes Hunter workflow, which requires a table of counts (genes as rows) before DESeq2, edgeR, and limma differential expression detection can be run.

## When to use

Use when a raw or simulated RNA-seq count table (genes as rows, samples as columns, with control/treatment conditions) must be validated and prepared as input for DEG detection in the ExpHunter Suite.

## When NOT to use

- Input is still raw FASTQ — the README explicitly requires a clean&map program to transform fastq files into count tables before DEgenes Hunter can be used
- Input is already DEG results or a ranked gene list rather than a table of counts — this skill covers the upstream count layer, not downstream consensus or enrichment steps
- The goal is co-expression module detection or functional enrichment without a count matrix, which are separate ExpHunter Suite modules

## Inputs

- gene-by-sample RNA-seq count matrix (e.g. simulated_counts.tsv, genes as rows)
- sample metadata / experimental design mapping samples to control and treatment conditions
- ground-truth DEG labels file (deg_labels.tsv) when benchmarking on simulated data
- FASTQ files plus NCBI SRA download configuration (nform ncbi_sra_file_downloader.sh JSON credentials) when starting from raw reads

## Outputs

- validated count table ready for DEgenes Hunter input
- per-method DEG call table with vote count and estimated logFC (e.g. deg_results.csv)
- results HTML report generated after running get_results.sh
- paired simulated_counts.tsv and deg_labels.tsv when generating synthetic benchmark data

## How to apply

Obtain a gene-by-sample table of counts either from an initial RNA-seq data analysis producing a table of counts, or from the ExpHunterSuite TCC-based simulation helper which writes simulated_counts.tsv with genes as rows and ground-truth deg_labels.tsv. For raw FASTQ data, the ExpHunterSuiteExamples README requires first running a clean&map program to transform fastq files into count tables, with NCBI SRA downloads and conversion_loop.sh configured beforehand. Verify the count matrix structure (row count of simulated_counts.tsv equals the simulated gene count, e.g. 10,000 genes) and attach the experimental design/metadata associating samples with control and treatment conditions. Then execute the DEgenes Hunter pipeline (launch_DEgenesHunter.sh for the Lafora or PMM2-CDG experiments) so the count table feeds DESeq2, edgeR, and limma, with per-method DEG calls combined by a minimum vote threshold cut-off (optimum threshold 2 for spike-in data by F-measure).

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose DEgenes Hunter module consumes the count table to run DESeq2, edgeR, and limma differential expression) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (pipeline executed on the count matrix and experimental design for the Lafora and PMM2-CDG experiments) — https://github.com/fmjabato/ExpHunterSuiteExamples
- **DESeq2** (one of the DEG detection methods run on the count matrix; in the PMM2-CDG experiment DESeq2 was by far the least conservative)
- **limma** (one of the DEG detection methods run on the count matrix; showed higher precision)
- **edgeR** (one of the three DEG detection methods executed on the count table by DEgenes Hunter)
- **Bioconductor** (ecosystem providing ExpHunterSuite and the TCC A. thaliana reference count dataset used by the simulation helper) — https://github.com/seoanezonjic/ExpHunterSuite

## Examples

```
bash launch_DEgenesHunter.sh
```

## Evaluation signals

- Row count of simulated_counts.tsv equals the simulated gene count (e.g. 10,000 genes) and both simulated_counts.tsv and deg_labels.tsv exist
- Count matrix has genes as rows and samples annotated with control vs treatment conditions matching the experimental design
- Downstream consensus reproduces the PMM2-CDG expectation: 1081 DEGs detected by at least one method and 345 by all three
- Outlier genes reproduce logFC values of approximately -23.5, -8.8, and -2.6
- After get_results.sh, a results HTML report has been generated

## Limitations

- Starting from FASTQ requires external NCBI credentials (JSON in ncbi_sra_file_downloader.sh) and a separate clean&map program; the count matrix is not produced by ExpHunterSuite itself in that path
- reactome.db installation can fail due to a 60-second R download timeout since the package is >400 Mb; set options(timeout=1000) before installing ExpHunterSuite
- The optimal vote threshold does not necessarily generalize: 'this may differ for other organisms, tissues and experimental designs', so threshold choices validated on spike-in data (optimum 2 by F-measure) should be re-examined per dataset

## Evidence

- [intro] Various workflows exist for initial RNA-seq data analysis to produce a table of counts, which serves as input for downstream processes such as differential expression (DE) analysis: "Various workflows exist for initial RNA-seq data analysis to produce a table of counts, which serves as input for downstream processes such as differential expression (DE) analysis"
- [readme] At this point, you must use a clean&map program to transform fastq files into count tables.: [evidence span withheld — anchor status: external_doc]
- [discussion] we would suggest the use of a minimum vote threshold cut-off where possible: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [other] verify both output files exist and that the row count of simulated_counts.tsv equals the simulated gene count (e.g. 10,000 genes).: [evidence span withheld — anchor status: not_found]

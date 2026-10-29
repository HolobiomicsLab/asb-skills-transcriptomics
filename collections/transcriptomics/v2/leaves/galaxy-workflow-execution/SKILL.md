---
name: galaxy-workflow-execution
description: Use when annotating plant miRNAs from a reference genome plus sRNA-Seq
  datasets (SRA accessions or FASTQ) and you need to chain database aggregation, prediction,
  selection, and curation steps in a reproducible web-based environment.
when_to_use_negative:
- sRNA-Seq data comes from a species without a reference genome — iwa-miRNA cannot
  handle data from species without genome sequences
- The analysis targets reads that do not map to the reference genome — the current
  version ignores unmapped sequencing reads
- The goal is downstream enrichment or cross-species comparative analysis of already-selected
  miRNAs (e.g. GO annotation, miRNA set enrichment) — iwa-miRNA has limited downstream
  analysis capability
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0091
tools:
- name: iwa-miRNA
  role: Three-module miRNA annotation workflow (MiRNA Compilation, MiRNA Selection,
    Manual Curation) whose source codes are packaged as Galaxy tools
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Framework hosting the iwa-miRNA modules with user-friendly web interfaces
    for step-by-step execution and reproducibility
  repo: https://github.com/cma2015/iwa-miRNA
- name: Docker (malab/iwa-mirna)
  role: Container image enabling local execution of the full Galaxy-based iwa-miRNA
    workflow
  repo: https://hub.docker.com/r/malab/iwa-mirna
- name: Rmarkdown
  role: Renders the manualCuration module's summary and report pages into HTML for
    visualization and sharing
- name: psRNAtarget
  role: Predicts miRNA-target interactions displayed as bipartite networks in the
    manualCuration report pages
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:b57325a3e2e44bf406b535f96758f4fd754909893e0e9340a2ff0c10fee586dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Docker
  - Docker (malab/iwa-mirna)
  - Galaxy
  - PmiREN
  - PsRNA
  - RefSeq
  - Rmarkdown
  - SRA
  - iwa-miRNA
  - miRBase
  - psRNAtarget
  - sRNAanno
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# galaxy-workflow-execution

## Summary

Runs the iwa-miRNA pipeline (MiRNA Compilation, MiRNA Selection, Manual Curation) as Galaxy modules, either via the public Galaxy-based web server or a local Docker deployment, so plant miRNA annotation is executed without command-line skills and with reproducible tool histories.

## When to use

Use when annotating plant miRNAs from a reference genome plus sRNA-Seq datasets (SRA accessions or FASTQ) and you need to chain database aggregation, prediction, selection, and curation steps in a reproducible web-based environment.

## When NOT to use

- sRNA-Seq data comes from a species without a reference genome — iwa-miRNA cannot handle data from species without genome sequences
- The analysis targets reads that do not map to the reference genome — the current version ignores unmapped sequencing reads
- The goal is downstream enrichment or cross-species comparative analysis of already-selected miRNAs (e.g. GO annotation, miRNA set enrichment) — iwa-miRNA has limited downstream analysis capability

## Inputs

- Species name and choice of miRNA databases (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0) for miRNARetrival
- Reference genome sequences and annotation files (e.g. IWGSC RefSeq v1.0 for wheat) for genomePrepare
- sRNA-Seq data as SRA accession numbers or uploaded FASTQ files for miRNAPredict
- Test datasets from github.com/cma2015/iwa-miRNA/tree/master/Test_data for pipeline validation

## Outputs

- Uniform-format miRNA and precursor candidate list from miRNATranslate (e.g. 2857 aggregated wheat precursors)
- Selected miRNA candidate sets from miRNASelection (Set1 high-throughput-criteria set, Set2 one-class-SVM set, and their overlap) as tab-delimited tables
- Rmarkdown HTML summary and report pages with secondary structure plots, read stacks, expression boxplots, and psRNAtarget miRNA-target bipartite networks
- Exported table, GFF3/GTF, and FASTA files of curated miRNAs and precursors

## How to apply

Assemble the three iwa-miRNA modules — MiRNA Compilation, MiRNA Selection, and Manual Curation — whose source codes and dependencies are fully organized within the Galaxy framework. Chain the fine-grained steps in order: miRNARetrival (species name + miRNA databases) → genomePrepare (genome sequences and annotation) → miRNAPredict (SRA accession numbers or uploaded FASTQ) → miRNATranslate (merge retrieval and prediction outputs into a uniform genomic coordinate format) → miRNASelection (high-throughput criteria and one-class SVM with default parameters) → manualCuration (Rmarkdown HTML summary/report pages). Execute on the public Galaxy server (http://iwa-miRNA.omicstudio.cloud), a local Docker deployment (image malab/iwa-mirna), or the released source code, using the Test_data bundle from the GitHub repository to validate the setup before real runs. The rationale is that Galaxy captures each tool invocation and its parameters in a reproducible history, so the whole annotation — e.g. 2857 aggregated wheat precursors narrowed to wheat-Set1 (1617 HT-criteria) and wheat-Set2 (1410 one-class SVM) and finally 2030 selected precursors — can be audited step by step.

## Related tools

- **iwa-miRNA** (Three-module miRNA annotation workflow (MiRNA Compilation, MiRNA Selection, Manual Curation) whose source codes are packaged as Galaxy tools) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Framework hosting the iwa-miRNA modules with user-friendly web interfaces for step-by-step execution and reproducibility) — https://github.com/cma2015/iwa-miRNA
- **Docker (malab/iwa-mirna)** (Container image enabling local execution of the full Galaxy-based iwa-miRNA workflow) — https://hub.docker.com/r/malab/iwa-mirna
- **Rmarkdown** (Renders the manualCuration module's summary and report pages into HTML for visualization and sharing)
- **psRNAtarget** (Predicts miRNA-target interactions displayed as bipartite networks in the manualCuration report pages)

## Examples

```
docker pull malab/iwa-mirna  # then run the Galaxy-based iwa-miRNA modules at http://iwa-miRNA.omicstudio.cloud or locally
```

## Evaluation signals

- Each module in the Galaxy history completes in the documented order (miRNARetrival → genomePrepare → miRNAPredict → miRNATranslate → miRNASelection → manualCuration) with the expected input/output wiring
- miRNATranslate output is a uniform-format candidate list whose count matches the aggregated expectation (e.g. 2857 wheat precursors, or 667 A. thaliana precursor candidates / 1190 mature candidates)
- miRNASelection output contains two labelled sets whose counts match reported benchmarks (e.g. wheat-Set1 = 1617 and wheat-Set2 = 1410; Ara-Set1 = 330 and Ara-Set2 = 308)
- The manualCuration HTML report renders with secondary structure plots, read stacks, expression boxplots, and psRNAtarget miRNA-target bipartite networks
- A dry run with the bundled Test_data reproduces the tutorial outputs before real SRA/FASTQ data are processed

## Limitations

- It cannot handle sRNA-Seq data from species without genome sequences
- In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome
- Limited downstream analysis of selected miRNAs: no enrichment analysis (e.g. microRNA gene ontology annotation, miRNA set enrichment) and no cross-species comparative analysis
- Planned but not yet implemented: sRNA-Seq analysis for species without a reference genome and miRNA discovery from unmapped reads

## Evidence

- [methods] The source codes of the modules and their dependencies are fully organized within the Galaxy framework.: "The source codes of the modules and their dependencies are fully organized within the Galaxy framework."
- [methods] The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation (Figure 1; Table 1).: "The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation (Figure 1; Table 1)."
- [intro] The source codes of iwa-miRNA have been combined into a Galaxy platform, organized with user-friendly web interfaces: "The source codes of iwa-miRNA have been combined into a Galaxy platform, organized with user-friendly web interfaces"
- [readme] Source codes and Docker image of iwa-miRNA were released for the first time.: "2020-06-20: Source codes and Docker image of iwa-miRNA were released for the first time."
- [readme] Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"

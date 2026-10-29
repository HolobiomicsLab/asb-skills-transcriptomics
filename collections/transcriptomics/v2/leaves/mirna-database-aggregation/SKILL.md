---
name: mirna-database-aggregation
description: Use when annotating miRNAs in a plant species with a reference genome
  and you need a consolidated candidate precursor list combining public database annotations
  with novel sRNA-Seq-based predictions.
when_to_use_negative:
- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq
  data from species without genome sequences and ignores reads that do not map to
  the reference
- You need a curated, false-positive-free miRNA annotation set — the aggregated candidate
  list certainly contains false positive hits and requires downstream selection and
  wet-lab validation
- You only need annotations from a single database rather than a cross-database consolidated
  candidate list
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0659
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose MiRNA Compilation module (miRNARetrival, genomePrepare,
    miRNAPredict, miRNATranslate) performs the database aggregation and prediction
    merge
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Platform hosting the iwa-miRNA modules with user-friendly web interfaces
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRBase
  role: Source database of already annotated miRNA precursors (v22.1)
- name: PmiREN
  role: Source database of already annotated plant miRNA precursors (v1.0)
- name: sRNAanno
  role: Source database of already annotated plant miRNA precursors (v1.0)
- name: PsRNA
  role: Source database of already annotated plant small RNA genes/miRNA precursors
    (v1.0)
- name: Docker
  role: Deployment mechanism; iwa-miRNA Docker image at malab/iwa-mirna
  repo: https://hub.docker.com/r/malab/iwa-mirna
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:903580b745cff12608c36ab6fbc065e03ac845e1777f442fd5eb2700ea1a165d
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
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# mirna-database-aggregation

## Summary

Aggregate already annotated plant miRNA precursors from four representative databases (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0) using the iwa-miRNA miRNARetrival tool, then merge them with newly predicted precursors into a uniform genomic coordinate system to produce a comprehensive candidate list.

## When to use

Use when annotating miRNAs in a plant species with a reference genome and you need a consolidated candidate precursor list combining public database annotations with novel sRNA-Seq-based predictions.

## When NOT to use

- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences and ignores reads that do not map to the reference
- You need a curated, false-positive-free miRNA annotation set — the aggregated candidate list certainly contains false positive hits and requires downstream selection and wet-lab validation
- You only need annotations from a single database rather than a cross-database consolidated candidate list

## Inputs

- Species name (e.g., 'Arabidopsis thaliana', 'Zea mays', 'Triticum aestivum')
- miRNA database selection: miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0
- Reference genome sequences and annotation files (e.g., TAIR10, B73, IWGSC RefSeq v1.0) for genomePrepare
- sRNA-Seq datasets as SRA accession numbers or uploaded fastq files

## Outputs

- Already annotated miRNA precursors from miRNARetrival (e.g., 365 precursors / 625 mature forms for A. thaliana)
- Formatted genome sequences and annotation from genomePrepare
- Predicted miRNA precursors from miRNAPredict (e.g., 435 for A. thaliana from 1063 sRNA-Seq datasets)
- Merged candidate precursor list in uniform GFF3 format from miRNATranslate (e.g., 667 precursor candidates / 1190 mature candidates for A. thaliana)

## How to apply

Run the iwa-miRNA miRNARetrival tool with the species name and target miRNA databases to retrieve already annotated precursors from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0. Run genomePrepare with the species name or genome sequences and annotation (e.g., TAIR10 for A. thaliana, IWGSC RefSeq v1.0 for wheat) to obtain formatted reference files. Run miRNAPredict on sRNA-Seq inputs supplied as SRA accession numbers or uploaded fastq files, mapping reads to the reference genome with GMAP to predict novel precursors. Run miRNATranslate to convert the outputs of miRNARetrival and miRNAPredict into a uniform genomic coordinate system, then merge both sources into a single candidate precursor list and tally overlaps with each source database. Because the aggregation of annotated and newly predicted miRNAs can contain false positives, the merged list is intended as input to downstream selection (miRNASelection with high-throughput criteria and one-class SVM) rather than as a final annotation.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose MiRNA Compilation module (miRNARetrival, genomePrepare, miRNAPredict, miRNATranslate) performs the database aggregation and prediction merge) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Platform hosting the iwa-miRNA modules with user-friendly web interfaces) — https://github.com/cma2015/iwa-miRNA
- **miRBase** (Source database of already annotated miRNA precursors (v22.1))
- **PmiREN** (Source database of already annotated plant miRNA precursors (v1.0))
- **sRNAanno** (Source database of already annotated plant miRNA precursors (v1.0))
- **PsRNA** (Source database of already annotated plant small RNA genes/miRNA precursors (v1.0))
- **Docker** (Deployment mechanism; iwa-miRNA Docker image at malab/iwa-mirna) — https://hub.docker.com/r/malab/iwa-mirna

## Evaluation signals

- Combined output file exists in GFF3 format with precursors in a uniform genomic coordinate system
- For the A. thaliana benchmark, counts match the paper-reported values: 365 annotated precursors / 625 mature forms, 435 predicted precursors (302 previously unannotated), 667 combined precursor candidates / 1190 mature candidates
- Per-database overlap tallies are computed for each of the four source databases (miRBase, PmiREN, sRNAanno, PsRNA)
- Wheat and maize benchmark counts reproduce: wheat aggregated = 2857, maize aggregated = 619 / predicted = 1241
- Downstream miRNASelection counts are consistent with the aggregated candidate list (e.g., Ara-Set1 330 / Ara-Set2 308, wheat-Set1 1617 / wheat-Set2 1410)

## Limitations

- Cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- The aggregated candidate list certainly contains false positive hits; wet experiments are still required to validate candidates before functional investigation
- MiRNA discovery from unmapped reads and sRNA-Seq analysis for species without a reference genome are planned but not yet implemented

## Evidence

- [intro] iwa-miRNA not only provides functions for automatically incorporating miRNA annotations from four representative databases (i.e., miRBase, PmiREN, sRNAanno, and PsRNA) but also builds a: "iwa-miRNA not only provides functions for automatically incorporating miRNA annotations from four representative databases (i.e., miRBase, PmiREN, sRNAanno, and PsRNA)"
- [results] Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors, 302 of which were not previously annotated: "Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors"
- [methods] miRNATranslate Output from miRNARetrival and miRNAPredict miRNA and miRNA precursors with a uniform format: [evidence span withheld — anchor status: not_found]
- [abstract] iwa-miRNA is specifically designed to generate a comprehensive list of miRNA candidates, bridging the gap between already annotated miRNAs provided by public miRNA databases and new predictions from: "iwa-miRNA is specifically designed to generate a comprehensive list of miRNA candidates, bridging the gap between already annotated miRNAs provided by public miRNA databases and new predictions"
- [discussion] The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits: "The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits"

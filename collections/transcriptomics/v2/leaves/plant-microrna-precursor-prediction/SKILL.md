---
name: plant-microrna-precursor-prediction
description: Use when you need a genome-wide list of miRNA precursor candidates for
  a plant species with a reference genome, combining public annotations with novel
  predictions from small RNA sequencing (sRNA-Seq) data.
when_to_use_negative:
- 'The species has no reference genome sequence: iwa-miRNA ''cannot handle sRNA-Seq
  data from species without genome sequences'''
- 'The analysis goal is miRNA discovery from reads that do not map to the reference
  genome: ''In the current version, iwa-miRNA ignores sequencing reads that do not
  map to the reference genome'''
- 'You need only a curated, high-confidence miRNA set without downstream filtering:
  the aggregated list ''certainly contain[s] false positive hits'' and requires MiRNA
  Selection and Manual Curation before functional use'
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Galaxy-based framework providing the miRNARetrival, genomePrepare, miRNAPredict,
    and miRNATranslate tools used to compile annotated and predicted miRNA precursor
    candidates
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Platform within which the iwa-miRNA modules and their dependencies are organized
    with user-friendly web interfaces
  repo: https://github.com/cma2015/iwa-miRNA
- name: Docker
  role: Container image for local deployment of iwa-miRNA
  repo: https://hub.docker.com/r/malab/iwa-mirna
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:9dbf353af31793995e37718a105baf62c6511bb0065794d6e96d52d6fa059909
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Docker
  - Galaxy
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# plant-microrna-precursor-prediction

## Summary

Predict novel miRNA precursors in plant species with reference genomes using iwa-miRNA, by aggregating already annotated precursors from four plant miRNA databases (miRBase, PmiREN, sRNAanno, PsRNA) with new predictions from sRNA-Seq data mapped to the reference genome.

## When to use

Use when you need a genome-wide list of miRNA precursor candidates for a plant species with a reference genome, combining public annotations with novel predictions from small RNA sequencing (sRNA-Seq) data.

## When NOT to use

- The species has no reference genome sequence: iwa-miRNA 'cannot handle sRNA-Seq data from species without genome sequences'
- The analysis goal is miRNA discovery from reads that do not map to the reference genome: 'In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome'
- You need only a curated, high-confidence miRNA set without downstream filtering: the aggregated list 'certainly contain[s] false positive hits' and requires MiRNA Selection and Manual Curation before functional use

## Inputs

- Species name (e.g., Arabidopsis thaliana, maize, hexaploid bread wheat)
- Reference genome sequences and annotation (for genomePrepare)
- sRNA-Seq data as SRA accession numbers or uploaded fastq files (e.g., 1063 sRNA-Seq datasets for A. thaliana Columbia ecotype)
- miRNA database selections (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0)

## Outputs

- Already annotated miRNAs (output of miRNARetrival)
- Predicted miRNAs from sRNA-Seq data (output of miRNAPredict)
- miRNA and miRNA precursors with a uniform format (output of miRNATranslate, GFF3-format combined precursor list)
- Combined list of miRNA precursor candidates (e.g., 667 precursors / 1190 mature candidates for A. thaliana) with tallies of overlap with each source database

## How to apply

Run miRNARetrival with the species name to aggregate already annotated miRNA precursors from the four representative plant miRNA databases (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0). Run genomePrepare to format reference genome sequences and annotation, then run miRNAPredict on sRNA-Seq inputs (SRA accession numbers or uploaded fastq files), mapping reads to the reference genome with GMAP to predict novel precursors. Run miRNATranslate to place annotated and predicted miRNAs into a uniform genomic coordinate system, and merge them into a single candidate precursor list. Rationale: existing plant miRNA databases contain false positives and false negatives, so aggregation plus de novo prediction yields a more comprehensive candidate set; in the A. thaliana benchmark this yielded 667 combined precursor candidates corresponding to 1190 mature candidates.

## Related tools

- **iwa-miRNA** (Galaxy-based framework providing the miRNARetrival, genomePrepare, miRNAPredict, and miRNATranslate tools used to compile annotated and predicted miRNA precursor candidates) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Platform within which the iwa-miRNA modules and their dependencies are organized with user-friendly web interfaces) — https://github.com/cma2015/iwa-miRNA
- **Docker** (Container image for local deployment of iwa-miRNA) — https://hub.docker.com/r/malab/iwa-mirna

## Evaluation signals

- The combined output file exists and is in GFF3 format
- For the A. thaliana benchmark, the annotated set recovers 365 miRNA precursors corresponding to 625 mature forms from the four databases
- For the A. thaliana benchmark, prediction from 1063 sRNA-Seq datasets yields 435 miRNA precursors, 302 of which were not previously annotated in any of the four databases
- The combined run recovers 667 miRNA precursor candidates corresponding to 1190 mature miRNA candidates
- Tallies of how many candidates overlap each of the four source databases are produced

## Limitations

- Cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- The aggregated candidate list certainly contains false positive hits, so downstream selection (high-throughput criteria, one-class SVM) and manual curation are needed
- Further wet experiments should be performed to validate miRNA candidates of particular interest before any functional investigation
- Planned but not yet implemented: sRNA-Seq analysis for species without a reference genome and miRNA discovery from unmapped reads

## Evidence

- [methods] MiRNA Compilation miRNARetrival Name of species and miRNA databases Already annotated miRNAs: [evidence span withheld — anchor status: not_found]
- [methods] miRNAPredict SRA accession numbers or uploaded fastq files Predicted miRNAs Predict miRNAs from sRNA-Seq data: [evidence span withheld — anchor status: not_found]
- [results] This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2): "This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2)"
- [results] Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors: "Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors"
- [discussion] It cannot handle sRNA-Seq data from species without genome sequences.: "It cannot handle sRNA-Seq data from species without genome sequences."
- [readme] Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"

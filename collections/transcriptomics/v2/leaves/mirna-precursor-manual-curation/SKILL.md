---
name: mirna-precursor-manual-curation
description: Use when annotating miRNAs in a plant species WITH a reference genome,
  by combining already annotated precursors from miRBase/PmiREN/sRNAanno/PsRNA with
  new predictions from public sRNA-Seq datasets (SRA accessions or FASTQ), and interactively
  selecting promising candidates.
when_to_use_negative:
- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq
  data from species without genome sequences
- Most reads are expected to be unaligned — sequencing reads that do not map to the
  reference genome are ignored in the current version
- The goal is downstream enrichment analysis (GO annotation, miRNA set enrichment)
  or cross-species comparative analysis of already-selected miRNAs — iwa-miRNA has
  limited ability for such downstream analyses
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0092
tools:
- name: iwa-miRNA
  role: Galaxy-based framework providing the MiRNA Compilation, MiRNA Selection, and
    Manual Curation modules that implement this skill end-to-end
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Hosts the iwa-miRNA modules with user-friendly web interfaces so curation
    proceeds without computational skills
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Renders the manualCuration summary and report pages into HTML for easy visualization,
    interpretation, and sharing
- name: psRNAtarget
  role: Provides the miRNA-target interaction predictions displayed as bipartite networks
    during candidate inspection
- name: Docker
  role: Container image for local deployment of the iwa-miRNA platform
  repo: https://hub.docker.com/r/malab/iwa-mirna
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:020355b2ee1840fbf91540561724ee8f8db0584644eeaa61c30a324571742483
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
  - Rmarkdown
  - SRA
  - iwa-miRNA
  - miRBase
  - psRNAtarget
  - sRNAanno
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# mirna-precursor-manual-curation

## Summary

Interactive, Galaxy-based manual curation of plant miRNA precursor candidates using iwa-miRNA: aggregated database annotations and sRNA-Seq-derived predictions are filtered with high-throughput criteria and a one-class SVM, then inspected via Rmarkdown HTML reports before export as table, GFF3/GTF, and FASTA.

## When to use

Use when annotating miRNAs in a plant species WITH a reference genome, by combining already annotated precursors from miRBase/PmiREN/sRNAanno/PsRNA with new predictions from public sRNA-Seq datasets (SRA accessions or FASTQ), and interactively selecting promising candidates.

## When NOT to use

- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Most reads are expected to be unaligned — sequencing reads that do not map to the reference genome are ignored in the current version
- The goal is downstream enrichment analysis (GO annotation, miRNA set enrichment) or cross-species comparative analysis of already-selected miRNAs — iwa-miRNA has limited ability for such downstream analyses

## Inputs

- Species name (e.g., Triticum aestivum) and selected miRNA databases (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0)
- Reference genome sequences and annotation files (e.g., IWGSC RefSeq v1.0) via genomePrepare
- SRA accession numbers or uploaded FASTQ files of small RNA-Seq datasets (e.g., 95 bread wheat sRNA-Seq datasets)
- miRNATranslate output (uniform-format miRNA and miRNA precursor candidates)
- miRNASelection output (high-throughput criteria set and one-class SVM set)

## Outputs

- Two filtered candidate sets: HT-criteria set (e.g., wheat-Set1, 1617 precursors) and one-class SVM set (e.g., wheat-Set2, 1410 precursors)
- Manual curation summary and report pages (Rmarkdown HTML with secondary structure plots, read stacks, expression boxplots, psRNAtarget miRNA-target bipartite networks)
- Final selected precursor set (e.g., 2030 precursors / 2163 loci) exported as table, GFF3/GTF, and FASTA files

## How to apply

Run the MiRNA Compilation module: miRNARetrival aggregates already annotated miRNAs from miRBase (v22.1), PmiREN (v1.0), sRNAanno (v1.0), and PsRNA (v1.0); genomePrepare formats the reference genome sequences and annotation; miRNAPredict predicts precursor candidates from sRNA-Seq data given SRA accession numbers or uploaded FASTQ; miRNATranslate merges both sources into a uniform genomic coordinate format. In MiRNA Selection, apply miRNASelection on the miRNATranslate output using BOTH high-throughput criteria and the one-class SVM classifier at default parameters to produce two candidate sets (e.g., wheat-Set1 = 1617 HT-passing precursors; wheat-Set2 = 1410 SVM-predicted precursors from 2857 aggregated candidates). In Manual Curation, run manualCuration on the miRNASelection output to generate summary and report pages as Rmarkdown HTML, dynamically adjusting thresholds while inspecting secondary structure plots, read stacks, expression boxplots, and psRNAtarget miRNA-target bipartite networks. Rationale: aggregation of annotated and newly predicted miRNAs certainly contains false positive hits, so interactive human inspection plus qRT-PCR validation of selected candidates is required before functional investigation.

## Related tools

- **iwa-miRNA** (Galaxy-based framework providing the MiRNA Compilation, MiRNA Selection, and Manual Curation modules that implement this skill end-to-end) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Hosts the iwa-miRNA modules with user-friendly web interfaces so curation proceeds without computational skills) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Renders the manualCuration summary and report pages into HTML for easy visualization, interpretation, and sharing)
- **psRNAtarget** (Provides the miRNA-target interaction predictions displayed as bipartite networks during candidate inspection)
- **Docker** (Container image for local deployment of the iwa-miRNA platform) — https://hub.docker.com/r/malab/iwa-mirna

## Examples

```
docker pull malab/iwa-mirna
```

## Evaluation signals

- Reproduce the wheat benchmark counts: aggregated = 2857 precursor candidates; wheat-Set1 (HT criteria) = 1617; wheat-Set2 (one-class SVM) = 1410; manually selected = 2030 precursors / 2163 loci
- miRNASelection yields exactly two candidate sets (HT-criteria set and SVM set) computed at default parameters on the miRNATranslate output
- manualCuration HTML report contains secondary structure plots, read stacks, expression boxplots, and psRNAtarget miRNA-target bipartite networks
- Final export includes all three formats: table, GFF3/GTF, and FASTA for the selected precursor loci
- Candidates selected during curation should be validated (e.g., by qRT-PCR with primers specific to mature sequences) before functional claims

## Limitations

- The aggregated candidate list certainly contains false positive hits, so 'wet' experiments should validate candidates of interest before functional investigation
- iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences, and ignores sequencing reads that do not map to the reference genome
- Limited downstream analysis of selected miRNAs: no enrichment analyses (microRNA GO annotation, miRNA set enrichment) and no cross-species comparative analysis; plans exist for analyzing sRNA-Seq data without a reference genome and discovering miRNAs from unmapped reads, but are not yet implemented

## Evidence

- [readme] The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation.: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."
- [methods] manualCuration consumes miRNASelection output to produce summary and report pages: [evidence span withheld — anchor status: not_found]
- [methods] Rmarkdown HTML reporting in iwa-miRNA: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [methods] psRNAtarget bipartite network in reports: "A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget [36]."
- [results] Both filters at default parameters identify promising candidates: "Both high-throughput criteria and one-class SVM with default parameters (Figure 1B) were used to identify promising miRNA candidates"
- [abstract] Interactive selection is the stated purpose: "It can also assist users in selecting promising miRNA candidates in an interactive mode, contributing to the accessibility and reproducibility of genome-wide miRNA annotation."

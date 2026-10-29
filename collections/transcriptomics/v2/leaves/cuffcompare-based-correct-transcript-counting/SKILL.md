---
name: cuffcompare-based-correct-transcript-counting
description: Use when assembled transcript GTFs from one or more assemblers (e.g.,
  Tiglon, StringTie2, Scallop) must be scored for structural correctness against a
  reference annotation such as RefSeq on human GRCh38 or mouse.
when_to_use_negative:
- 'Annotation-guided assembly is desired: the current Tiglon version has no option
  where known transcript annotations guide assembly, so this counting skill cannot
  substitute for annotation-informed pipelines'
- 'Data are long-read RNA-seq (PacBio or Oxford Nanopore): the current Tiglon version
  used in this workflow is not compatible with long-read datasets'
- The goal is transcript abundance quantification rather than structural correctness
  counting against a reference annotation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tiglon
  role: Produces the assembled transcript GTF (via labeled splice graphs and label-based
    dynamic path extension) whose correct transcripts are counted
  repo: https://github.com/yutingsdu/Tiglon
- name: StringTie2
  role: Single-aligner baseline assembler whose GTF output is scored by Cuffcompare
    for comparison against Tiglon
  repo: https://github.com/gpertea/stringtie
- name: Scallop
  role: Single-aligner baseline assembler whose GTF output is scored by Cuffcompare
    for comparison against Tiglon
  repo: https://github.com/Kingsford-Group/scallop
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:c15498a623307cd003d7cd34f6ad0d44b846d00bf6a034e8159a1499db27172d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yutingsdu/Tiglon
  tools:
  - Hisat2
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cuffcompare-based-correct-transcript-counting

## Summary

Evaluates genome-guided transcriptome assemblies (Tiglon, StringTie2, Scallop) by running Cuffcompare of each assembler's GTF against the species-specific NCBI RefSeq annotation and counting correct transcripts per dataset, enabling the paper's cross-assembler performance comparisons.

## When to use

Use when assembled transcript GTFs from one or more assemblers (e.g., Tiglon, StringTie2, Scallop) must be scored for structural correctness against a reference annotation such as RefSeq on human GRCh38 or mouse.

## When NOT to use

- Annotation-guided assembly is desired: the current Tiglon version has no option where known transcript annotations guide assembly, so this counting skill cannot substitute for annotation-informed pipelines
- Data are long-read RNA-seq (PacBio or Oxford Nanopore): the current Tiglon version used in this workflow is not compatible with long-read datasets
- The goal is transcript abundance quantification rather than structural correctness counting against a reference annotation

## Inputs

- Assembled transcript GTF files (Tiglon.gtf, StringTie2 output GTF, Scallop output GTF) per dataset
- NCBI RefSeq annotation files for the respective species (human GRCh38, Mus musculus)

## Outputs

- Per-dataset correct-transcript counts per assembler
- Aggregated comparison CSV (e.g., hisat2_default_consistency.csv) used to verify performance trends

## How to apply

Run Cuffcompare on every assembled transcript set (Tiglon.gtf, StringTie2 GTFs, Scallop GTFs) produced from the 12 real datasets (8 Homo sapiens H1–H8, 4 Mus musculus M1–M4), using the respective RefSeq annotations for human and mouse as the reference. Count, per assembler and per dataset, how many assembled transcripts are judged correct (matching reference transcript structures). Aggregate these per-dataset correct-transcript counts into a CSV (e.g., hisat2_default_consistency.csv) so that comparative performance trends between assemblers can be verified. As a validation invariant from the source workflow, check that Tiglon's correct-transcript count exceeds both StringTie2's and Scallop's counts (difference > 0) for each of the 12 datasets.

## Related tools

- **Tiglon** (Produces the assembled transcript GTF (via labeled splice graphs and label-based dynamic path extension) whose correct transcripts are counted) — https://github.com/yutingsdu/Tiglon
- **StringTie2** (Single-aligner baseline assembler whose GTF output is scored by Cuffcompare for comparison against Tiglon) — https://github.com/gpertea/stringtie
- **Scallop** (Single-aligner baseline assembler whose GTF output is scored by Cuffcompare for comparison against Tiglon) — https://github.com/Kingsford-Group/scallop

## Evaluation signals

- For each of the 12 datasets (H1–H8, M1–M4), Tiglon's correct-transcript count exceeds both StringTie2's and Scallop's counts (difference > 0), reproducing the reported trend
- Cuffcompare is run against the species-matched NCBI RefSeq annotation (human GRCh38 for H. sapiens datasets, mouse RefSeq for M. musculus datasets)
- Correct-transcript counts are recorded per assembler per dataset and aggregated into a single CSV (hisat2_default_consistency.csv)
- Expected magnitude of Tiglon's advantage on real human datasets: on average 9.51% more correct transcripts than StringTie2-Merge and 11.25%–13.68% more than StringTie2 and Scallop
- The same trend holds under Hisat2 --dta alignments, Hisat2 default (no --dta) alignments, and Star alignments

## Limitations

- The current Tiglon version does not support annotation-guided assemblies, i.e., known transcript annotations cannot guide the assembly before Cuffcompare scoring
- The current Tiglon version is not compatible with long-read RNA-seq datasets (e.g., Pacific Biosciences [PacBio] or Oxford Nanopore Technologies [ONT])
- Counts of correct transcripts depend on the chosen reference annotation; only NCBI RefSeq for human and mouse was used in this workflow

## Evidence

- [other] with Cuffcompare against the respective RefSeq annotations for human and mouse, counting correct transcripts per assembler per dataset: [evidence span withheld — anchor status: not_found]
- [intro] Tiglon averagely reconstructed 9.51% more correct transcripts than StringTie2-Merge, 11.25%–13.68% more than StringTie2 and Scallop: "Tiglon averagely reconstructed 9.51% more correct transcripts than StringTie2-Merge, 11.25%–13.68% more than StringTie2 and Scallop"
- [readme] It takes as input spliced alignments in coordinate-sorted SAM/BAM/CRAM format and produces a GTF output which consists of assembled: [evidence span withheld — anchor status: external_doc]
- [discussion] The current version of Tiglon does not have an option for annotation-guided assemblies: "The current version of Tiglon does not have an option for annotation-guided assemblies"

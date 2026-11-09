---
name: cross-tool-cell-call-overlap-analysis
description: Use when validating a new pathogen-detection tool for 10x scRNAseq against
  established tools by measuring concordance of infected-cell calls and correlating
  per-cell pathogen read/UMI counts.
when_to_use_negative:
- Input scRNAseq data has not yet been processed by cellranger count, so no unaligned
  reads or 10x output folders exist for pathogen mapping.
- The goal is detection of a pathogen with a complicated fasta header and no gtf,
  without simplifying the header — total read quantification relies on the fasta header
  name.
- Comparing against tools not benchmarked here or data types outside 10x single-cell
  output, where the four-tool overlap design does not apply.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0781
tools:
- name: scPathoQuant
  role: Primary tool under evaluation; aligns unaligned host reads to the HIV genome
    with default bbmap and outputs per-cell pathogen counts
  repo: https://github.com/galelab/scPathoQuant
- name: ViralTrack
  role: Comparator tool run independently on the same donor 2 sample to obtain per-cell
    HIV+ calls
  repo: https://github.com/PierreBSC/Viral-Track
- name: PathogenTrack
  role: Comparator tool run independently on the same donor 2 sample to obtain per-cell
    HIV+ calls
  repo: https://github.com/ncrna/PathogenTrack
- name: Venus
  role: Comparator tool run independently on the same donor 2 sample; expected to
    yield higher counts than UMI-based SPQ
  repo: https://github.com/aicb-ZhangLabs/Venus
- name: cellranger
  role: Generates the 10x output folder and host-aligned data whose unaligned reads
    SPQ maps to the pathogen genome
- name: bbmap
  role: Default aligner used by SPQ to map extracted unaligned reads to the HIV reference
    genome
- name: samtools
  role: External software used by SPQ to process pathogen alignments
- name: htseq
  role: External software (htseq-count with --intersection-nonempty) used by SPQ to
    quantify pathogen reads and gene counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:f84c5b4cf76c2bb0284b5c5c508f15a20651d8a9450bd629c074881e9d8590d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - PathogenTrack
  - Venus
  - ViralTrack
  - bbmap
  - cellranger
  - htseq
  - samtools
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-tool-cell-call-overlap-analysis

## Summary

Benchmark scPathoQuant (SPQ) HIV-infected cell calls against ViralTrack, PathogenTrack, and Venus by building a four-tool overlap table of per-cell HIV+ calls and computing HIV read-abundance correlations between SPQ and each comparator tool.

## When to use

Use when validating a new pathogen-detection tool for 10x scRNAseq against established tools by measuring concordance of infected-cell calls and correlating per-cell pathogen read/UMI counts.

## When NOT to use

- Input scRNAseq data has not yet been processed by cellranger count, so no unaligned reads or 10x output folders exist for pathogen mapping.
- The goal is detection of a pathogen with a complicated fasta header and no gtf, without simplifying the header — total read quantification relies on the fasta header name.
- Comparing against tools not benchmarked here or data types outside 10x single-cell output, where the four-tool overlap design does not apply.

## Inputs

- cellranger count output folder for the HIV-infected donor 2 sample (10x scRNAseq)
- Simplified HIV reference genome fasta (header simplified, e.g. '>HIV_virus') from galelab/Whitmore_scPathoQuant_testSets
- Per-cell HIV+ call lists from ViralTrack, PathogenTrack, and Venus run on the same donor 2 sample

## Outputs

- Four-tool overlap table of HIV-infected cell calls (expected 672 cells, 81.3% overlap, Fig. 5A)
- SPQ per-cell HIV read counts (pathogen_al_counts.csv)
- Correlation plots of SPQ counts vs. each comparator tool's counts (Fig. 5B-D)
- Pathogen-integrated filtered_feature_bc_matrix for downstream Seurat analysis

## How to apply

Run scPathoQuant on the cellranger count output folder for the HIV-infected donor 2 sample with the default bbmap aligner (scpathoquant -10x <10x folder> -op <results> -p 8 -p2genome <HIV fasta folder>), producing pathogen_al_counts.csv (per-cell HIV read counts) and pathogen-integrated filtered_feature_bc_matrix. Independently run ViralTrack, PathogenTrack, and Venus on the same sample to obtain each tool's per-cell HIV+ calls. Build the four-tool overlap table counting cells called HIV+ by all four methods; the expected result is 672 cells, i.e. 81.3% four-tool overlap, as in Fig. 5A. Then compute HIV read-abundance correlations between SPQ counts and each comparator tool (Fig. 5B-D), checking that Venus counts run higher than the UMI-based SPQ counts as an expected systematic offset. Accept the result when the overlap table reproduces the 672-cell / 81.3% overlap and correlations match the Fig. 5A-D panel behavior.

## Related tools

- **scPathoQuant** (Primary tool under evaluation; aligns unaligned host reads to the HIV genome with default bbmap and outputs per-cell pathogen counts) — https://github.com/galelab/scPathoQuant
- **ViralTrack** (Comparator tool run independently on the same donor 2 sample to obtain per-cell HIV+ calls) — https://github.com/PierreBSC/Viral-Track
- **PathogenTrack** (Comparator tool run independently on the same donor 2 sample to obtain per-cell HIV+ calls) — https://github.com/ncrna/PathogenTrack
- **Venus** (Comparator tool run independently on the same donor 2 sample; expected to yield higher counts than UMI-based SPQ) — https://github.com/aicb-ZhangLabs/Venus
- **cellranger** (Generates the 10x output folder and host-aligned data whose unaligned reads SPQ maps to the pathogen genome)
- **bbmap** (Default aligner used by SPQ to map extracted unaligned reads to the HIV reference genome)
- **samtools** (External software used by SPQ to process pathogen alignments)
- **htseq** (External software (htseq-count with --intersection-nonempty) used by SPQ to quantify pathogen reads and gene counts)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- The four-tool overlap table reproduces the reported 672 cells called HIV+ by all four methods, corresponding to 81.3% four-tool overlap (Fig. 5A).
- Correlation values between SPQ counts and each comparator tool are consistent with the Fig. 5B-D panel results.
- SPQ-vs-Venus counts show the expected upward offset, with Venus counts running higher than UMI-based SPQ counts.
- pathogen_al_counts.csv and the integrated filtered_feature_bc_matrix are produced and loadable into Seurat via Read10X.
- ViralTrack, PathogenTrack, and Venus results are consistent with SPQ on the same donor 2 sample.

## Limitations

- Venus counts systematically run higher than UMI-based SPQ counts, so raw count magnitudes should not be compared directly across tools.
- scPathoQuant overwrites the original 10x data provided to include pathogen counts; use a copy if the original output must be preserved.
- The temporary _tmp/ directory of files used by scpathoquant can be large and must be removed with --tmp_removal if space is an issue.
- The pathogen fasta header is used to quantify total pathogen reads; complicated headers should be simplified (e.g. >HIV_virus).
- No gtf file is mandatory; pathogen gene counts (pathogen_genes_al_sort_counts.sam) are only produced when a gtf is provided.

## Evidence

- [results] SPQ, ViralTrack, PathogenTrack, and Venus show consistent results on the HIV-infected donor 2 sample, with an 81.3% overlap between all 4 tools for identifying HIV-infected cells (672 cells HIV+ in all 4 methods).: [evidence span withheld — anchor status: not_found]
- [abstract] scPathoQuant extracts unaligned reads and maps them to a pathogen genome: "scPathoQuant extracts sequences that were not aligned to the primary host genome"
- [readme] README on the -10x parameter and bbmap default aligner: "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running [cellranger count]"
- [readme] README on fasta header simplification: "it is recommended that if the fasta header is a complicated name it be simplified (i.e. > HIV_virus)"
- [readme] README on overwrite behavior of original 10x data: "Overwrites original 10x data provided to include pathogen counts and pathogen gene counts (if gtf file is provided)"
- [intro] README on external software used for quantification: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads"

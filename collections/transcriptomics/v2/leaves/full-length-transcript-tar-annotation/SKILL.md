---
name: full-length-transcript-tar-annotation
description: Use when analyzing full-length Smart-seq2 (or similar plate-based paired-end)
  scRNA-seq from an organism with an incomplete genome annotation, and you need to
  detect cell-type-specific unannotated transcripts (uTARs) beyond the reference annotation.
when_to_use_negative:
- Input is 10x-style 3'-end droplet data already processed by cellranger count — use
  the from_cellranger droplet workflow instead of the Smart-seq2 paired-end fragment
  workflow
- Input is raw FASTQ rather than STAR-aligned BAMs — run the from_fastqs workflow
  (STAR alignment) first
- The genome annotation is high quality and complete for the target cell types, so
  uTAR discovery adds little; standard gene-level scRNA-seq analysis suffices
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: STAR
  role: Aligns paired-end Smart-seq2 sequencing reads for each cell prior to fragment
    extraction
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: bedtools
  role: Extracts read fragments from paired-end alignments via bamtobed with the '-bedpe'
    option
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model that partitions the genome into transcribed/untranscribed
    states to call strand-specific TARs per cell
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: featureCounts
  role: Counts TAR features per cell with strandedness 1 (-s 1) and paired-end option
    (-p) to build the TAR expression matrix
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: Seurat
  role: Analyzes the uTAR expression matrix and performs the Wilcoxon rank-sum test
    to identify differentially expressed uTARs per cell cluster
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: rtracklayer
  role: R package used to load/write labeled TAR annotation files (refFlat, gtf);
    must be manually installed for the groHMM scripts to run
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: Wraps the pipeline steps into reproducible workflows (from_fastqs or from_cellranger
    variants)
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:7c26ff9a20114dbe64aded9b9cdbf1a34218a1a7d3c7f4042e3c61c0a310742d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - STAR
  - Seurat
  - Snakemake
  - bedtools
  - featureCounts
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# full-length-transcript-tar-annotation

## Summary

Extends the gene-annotation-independent TAR-scRNA-seq pipeline to full-length plate-based scRNA-seq (Smart-seq2) data: STAR-aligned paired-end BAMs are converted to bedpe fragments, groHMM calls strand-specific TARs that are labeled aTAR/uTAR against existing annotations, and differential uTAR analysis recovers cell-type-specific unannotated transcripts.

## When to use

Use when analyzing full-length Smart-seq2 (or similar plate-based paired-end) scRNA-seq from an organism with an incomplete genome annotation, and you need to detect cell-type-specific unannotated transcripts (uTARs) beyond the reference annotation.

## When NOT to use

- Input is 10x-style 3'-end droplet data already processed by cellranger count — use the from_cellranger droplet workflow instead of the Smart-seq2 paired-end fragment workflow
- Input is raw FASTQ rather than STAR-aligned BAMs — run the from_fastqs workflow (STAR alignment) first
- The genome annotation is high quality and complete for the target cell types, so uTAR discovery adds little; standard gene-level scRNA-seq analysis suffices

## Inputs

- STAR-aligned paired-end Smart-seq2 BAM files (one per cell)
- Genome gene annotation file (for aTAR/uTAR labeling, refFlat/gtf)
- Cell cluster/metadata assignments (e.g. spermatocytes, spermatogonium, spermatids)

## Outputs

- Strand-specific TAR calls per cell in bed/refFlat/gtf formats with aTAR/uTAR labels
- TAR-by-cell expression count matrix
- Differential uTAR table per cluster annotated by sequence homology (ss2_differential_utars.csv)

## How to apply

Load STAR-aligned paired-end Smart-seq2 BAMs per cell and extract read fragments with 'bedtools bamtobed -bedpe'. Run groHMM on the fragments in a strand-specific manner (e.g. via scripts/SingleCellHMM_MW_SS2.bash) to call TARs per cell, labeling each TAR as aTAR or uTAR by overlap with existing gene annotations and writing the labeled annotations in refFlat and gtf formats. Count TAR features per cell with featureCounts using strandedness 1 (-s 1) and the paired-end option (-p), then aggregate count vectors into a TAR expression matrix. Analyze uTAR expression in Seurat with a Wilcoxon rank-sum test per cluster, filtering to uTARs expressed in at least 25% of cluster cells with log2 fold change > 0.25 versus the rest of the cells. Finally, annotate the top differentially expressed uTARs per cluster by nucleotide sequence homology (e.g. CCER1 in spermatocytes, TBXAS1 in spermatogonium, ncRNA LOC105867359 in spermatids in gray mouse lemur testes).

## Related tools

- **STAR** (Aligns paired-end Smart-seq2 sequencing reads for each cell prior to fragment extraction) — https://github.com/fw262/TAR-scRNA-seq
- **bedtools** (Extracts read fragments from paired-end alignments via bamtobed with the '-bedpe' option) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model that partitions the genome into transcribed/untranscribed states to call strand-specific TARs per cell) — https://github.com/fw262/TAR-scRNA-seq
- **featureCounts** (Counts TAR features per cell with strandedness 1 (-s 1) and paired-end option (-p) to build the TAR expression matrix) — https://github.com/fw262/TAR-scRNA-seq
- **Seurat** (Analyzes the uTAR expression matrix and performs the Wilcoxon rank-sum test to identify differentially expressed uTARs per cell cluster) — https://github.com/fw262/TAR-scRNA-seq
- **rtracklayer** (R package used to load/write labeled TAR annotation files (refFlat, gtf); must be manually installed for the groHMM scripts to run) — https://github.com/fw262/TAR-scRNA-seq
- **Snakemake** (Wraps the pipeline steps into reproducible workflows (from_fastqs or from_cellranger variants)) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
bash scripts/SingleCellHMM_MW_SS2.bash  # strand-specific groHMM TAR calling on SS2 bedpe fragments; featureCounts -s 1 -p then Seurat Wilcoxon yields ss2_differential_utars.csv
```

## Evaluation signals

- The differential uTAR table (ss2_differential_utars.csv) reproduces the reported assignments: CCER1 in spermatocytes, TBXAS1 in spermatogonium, and ncRNA LOC105867359 in spermatids, matching Supplementary Fig. 2b
- groHMM bed files (chr*_HMM.bed) exist for all chromosomes — absence triggers the known 'no lines available in input' error in calcHMMrefFlat
- Every TAR in the annotation output carries an aTAR or uTAR label consistent with overlap against the reference gene annotation
- Differential uTARs pass the filter: expressed in at least 25% of cluster cells and > 0.25 log fold change versus the rest of the cells
- featureCounts was run with strandedness 1 (-s 1) and paired-end (-p); counts aggregate into a TAR-by-cell expression matrix with one column per Smart-seq2 cell

## Limitations

- The groHMM R script can silently fail to produce chr*_HMM.bed files, causing downstream calcHMMrefFlat errors; groHMM and rtracklayer must be manually installed and loadable in R
- The degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so uTAR yield varies across datasets
- uTAR annotation relies on nucleotide sequence homology, so uTARs without recognizable homologs may remain unannotated

## Evidence

- [other] Extract read fragments from the paired-end data using bedtools bamtobed with the '-bedpe' option set.: [evidence span withheld — anchor status: not_found]
- [other] Count TAR features per cell using featureCounts with strandedness set to 1 (-s 1) and the paired-end option (-p): [evidence span withheld — anchor status: not_found]
- [other] filtering to uTARs expressed in at least 25% of cluster cells with more than 0.25 log-e fold change: [evidence span withheld — anchor status: not_found]
- [abstract] recovers biologically relevant transcriptional activity beyond the scope of the best available genome annotation: "recovers biologically relevant transcriptional activity beyond the scope of the best available genome annotation by performing scRNA-seq analysis on any region in the genome"
- [intro] groHMM uses a hidden Markov model to divide the genome into transcribed and untranscribed states: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Please manually install groHMM and rtracklayer and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"

---
name: hidden-markov-model-transcribed-region-calling
description: Use when analyzing scRNA-seq alignments (e.g. STAR-aligned 10x or Smart-seq2
  BAMs) to discover transcriptional activity outside a genome annotation, especially
  for organisms with incomplete gene annotation.
when_to_use_negative:
- You already have a feature-count expression table — this skill operates upstream
  on alignments, not on feature matrices
- You only want quantification within known gene annotations; standard featureCounts/gene-level
  counting suffices and groHMM segmentation is unnecessary
- Snakemake workflow selection does not match your starting point (e.g. using from_fastqs
  when data are already aligned with cellranger count)
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0749
tools:
- name: groHMM
  role: Runs the hidden Markov model that divides the genome into transcribed/untranscribed
    states per cell to call TARs
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: Loads and writes genomic interval tracks during groHMM bed-file generation
    and refFlat/gtf annotation export
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: bedtools
  role: Extracts read fragments from paired-end BAMs via bamtobed -bedpe before HMM
    segmentation
- name: Snakemake
  role: Orchestrates the per-cell HMM TAR-calling rules (e.g. calcHMMrefFlat) in the
    from_fastqs/from_cellranger workflows
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:50a0eae6c3a8f17b815059fd6186243ba9cb2e9599db6d124f528d5abe5cc6c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - STAR
  - Seurat
  - Snakemake
  - bedtools
  - featureCounts
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# Hidden Markov Model Transcribed Region Calling

## Summary

Applies groHMM's hidden Markov model to per-cell scRNA-seq alignments to segment the genome into transcriptionally active regions (TARs), classifying each as annotated (aTAR) or unannotated (uTAR) by overlap with existing gene annotations — the core segmentation step of the gene-annotation-independent TAR-scRNA-seq workflow.

## When to use

Use when analyzing scRNA-seq alignments (e.g. STAR-aligned 10x or Smart-seq2 BAMs) to discover transcriptional activity outside a genome annotation, especially for organisms with incomplete gene annotation.

## When NOT to use

- You already have a feature-count expression table — this skill operates upstream on alignments, not on feature matrices
- You only want quantification within known gene annotations; standard featureCounts/gene-level counting suffices and groHMM segmentation is unnecessary
- Snakemake workflow selection does not match your starting point (e.g. using from_fastqs when data are already aligned with cellranger count)

## Inputs

- STAR-aligned per-cell scRNA-seq BAM files (paired-end Smart-seq2 or 10x-compatible)
- Reference genome annotations in refFlat/gtf format for aTAR/uTAR labeling
- Raw FASTQ files (via the from_fastqs workflow) or cellranger count output (via from_cellranger workflow)

## Outputs

- Per-cell strand-specific TAR calls (groHMM bed files, chr*_HMM.bed)
- Labeled TAR annotation files in refFlat and gtf formats with aTAR/uTAR labels

## How to apply

Load the STAR-aligned sequencing files per cell and extract read fragments — for paired-end Smart-seq2 data use bedtools bamtobed with '-bedpe'. Run groHMM per cell on the read fragments in a strand-specific manner: the HMM divides the genome into transcribed and untranscribed states based on read coverage in each bin, emitting TAR intervals per cell. Label each TAR as aTAR or uTAR by overlap with existing gene annotations, and write the labeled annotations in both refFlat and gtf formats using the repository script (scripts/SingleCellHMM_MW_SS2.bash for SS2). The rationale is that per-cell HMM segmentation captures transcriptional signal regardless of annotation quality, so downstream featureCounts counting (-s 1, -p for paired-end) yields a TAR-by-cell expression matrix containing uTARs conventional workflows miss. If the rule calcHMMrefFlat fails with 'no lines available in input' or missing chr*_HMM.bed files, manually install groHMM and rtracklayer and verify they load in R.

## Related tools

- **groHMM** (Runs the hidden Markov model that divides the genome into transcribed/untranscribed states per cell to call TARs) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (Loads and writes genomic interval tracks during groHMM bed-file generation and refFlat/gtf annotation export) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **bedtools** (Extracts read fragments from paired-end BAMs via bamtobed -bedpe before HMM segmentation)
- **Snakemake** (Orchestrates the per-cell HMM TAR-calling rules (e.g. calcHMMrefFlat) in the from_fastqs/from_cellranger workflows) — github.com/fw262/TAR-scRNA-seq

## Examples

```
bedtools bamtobed -bedpe -i cell.bam > cell_fragments.bed && bash scripts/SingleCellHMM_MW_SS2.bash
```

## Evaluation signals

- groHMM runs complete and chr*_HMM.bed files exist for every chromosome/cell — absence with 'no lines available in input' from calcHMMrefFlat signals a failed groHMM R script
- Labeled annotation files are emitted in both refFlat and gtf formats with each TAR labeled aTAR or uTAR
- Reproducing the gray mouse lemur SS2 testes run recovers the expected uTAR assignments: CCER1 in spermatocytes, TBXAS1 in spermatogonium, ncRNA LOC105867359 in spermatids (Supplementary Fig. 2b)
- featureCounts on the TAR annotations with -s 1 -p yields a populated TAR-by-cell expression matrix that aggregates across cells without empty count vectors

## Limitations

- groHMM and rtracklayer must be manually installed; a common failure mode is the calcHMMrefFlat rule erroring with 'no lines available in input' because 'chr*_HMM.bed: No such file or directory' when the groHMM R script did not finish
- Degree of biologically relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so uTAR yield varies across runs
- Workflows assume specific starting points (raw fastqs or cellranger count alignment); other alignment states require adaptation

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [other] Run groHMM on the read fragments to generate TARs in a strand-specific manner, labeling TARs as aTAR or uTAR by overlap with existing gene annotations: [evidence span withheld — anchor status: not_found]
- [readme] This error indicates that the groHMM R script did not finish running to generate groHMM bed files.: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
- [readme] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs): "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs)"

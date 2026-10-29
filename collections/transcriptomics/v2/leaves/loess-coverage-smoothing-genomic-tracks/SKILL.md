---
name: loess-coverage-smoothing-genomic-tracks
description: Use when differentially expressed uTAR coverage tracks need smoothing
  and peak-coordinate extraction (FWHM) to define the nucleotide sequence that will
  be BLASTn-annotated.
when_to_use_negative:
- Coverage tracks that are already peak-called or the uTAR coordinates themselves
  are the intended BLAST query and peak refinement is unnecessary
- Annotating aTARs that already overlap a known genome annotation — homology-based
  labeling is aimed at unannotated TARs (uTARs)
- Read-level variant or splice analysis; LOESS smoothing destroys base-level resolution
  by design
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0102
tools:
- name: SAMtools
  role: Computes per-base depth over each uTAR (samtools depth) and extracts FASTA
    over FWHM coordinates (samtools faidx)
- name: BLAST
  role: BLASTn search of each FWHM peak sequence against the NCBI nt database with
    e-value <= 0.01 and bit score >= 50 cutoffs
- name: Snakemake
  role: Orchestrates the uTAR identification-to-annotation workflow in the TAR-scRNA-seq
    repository
  repo: github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: 'Upstream: hidden Markov model on read coverage that produces the TAR/uTAR
    coordinates whose peaks are smoothed'
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:50202e4705c56faaa261baacae72a451bed6a102ce4b48db054b83e5965c3a8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - SAMtools
  - Snakemake
  - groHMM
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# loess-coverage-smoothing-genomic-tracks

## Summary

Smooth per-uTAR read coverage from SAMtools depth with locally weighted (LOESS) regression (span = 0.25) and locate the full-width at half-maximum (FWHM) coordinates of the largest coverage peak, so a compact nucleotide sequence can be extracted for BLASTn homology annotation of differentially expressed uTARs.

## When to use

Use when differentially expressed uTAR coverage tracks need smoothing and peak-coordinate extraction (FWHM) to define the nucleotide sequence that will be BLASTn-annotated.

## When NOT to use

- Coverage tracks that are already peak-called or the uTAR coordinates themselves are the intended BLAST query and peak refinement is unnecessary
- Annotating aTARs that already overlap a known genome annotation — homology-based labeling is aimed at unannotated TARs (uTARs)
- Read-level variant or splice analysis; LOESS smoothing destroys base-level resolution by design

## Inputs

- Merged alignment files (BAM/CRAM) covering the differentially expressed uTARs
- List of differentially expressed uTARs with genomic coordinates (from Wilcoxon rank-sum screening)
- Reference genome FASTA indexed with samtools faidx for sequence extraction

## Outputs

- Smoothed per-uTAR LOESS coverage tracks (span = 0.25)
- FWHM peak coordinates for the largest coverage peak per uTAR
- FASTA sequences of the FWHM peak regions (via samtools faidx)
- utar_blast_annotations.csv: one labeled row per differentially expressed uTAR with top BLASTn hit or unknown

## How to apply

After differentially expressed uTARs are identified (Wilcoxon rank-sum test with dataset-specific thresholds such as >=25-50% of cells expressing the uTAR and >=0.25-1 log-e fold change), compute total per-base coverage of each uTAR from the merged alignment files using SAMtools depth. Smooth the coverage along each uTAR with LOESS using span = 0.25 to suppress local read-count noise while preserving the dominant transcribed peak. Locate the FWHM coordinates of the largest peak along the smoothed track, which delimits the most robustly transcribed portion of the uTAR rather than its full span. Extract the FASTA sequence over these FWHM coordinates with samtools faidx and feed it to BLASTn against the NCBI nt database (e-value <= 0.01, bit score >= 50), labeling each uTAR by its highest bit-score hit or 'unknown' if no hit qualifies. The rationale is that peak-centered sequences give cleaner homology signals than noisy full-uTAR sequences when annotating unannotated transcriptionally active regions.

## Related tools

- **SAMtools** (Computes per-base depth over each uTAR (samtools depth) and extracts FASTA over FWHM coordinates (samtools faidx))
- **BLAST** (BLASTn search of each FWHM peak sequence against the NCBI nt database with e-value <= 0.01 and bit score >= 50 cutoffs)
- **Snakemake** (Orchestrates the uTAR identification-to-annotation workflow in the TAR-scRNA-seq repository) — github.com/fw262/TAR-scRNA-seq
- **groHMM** (Upstream: hidden Markov model on read coverage that produces the TAR/uTAR coordinates whose peaks are smoothed) — https://bioconductor.org/packages/release/bioc/html/groHMM.html

## Examples

```
Rscript -e 'cov <- read.delim("utar_depth.txt"); fit <- loess(depth ~ pos, data = cov, span = 0.25); cat(which(fitted(fit) >= max(fitted(fit))/2), sep="\n")'
```

## Evaluation signals

- Output table utar_blast_annotations.csv contains exactly one labeled row per differentially expressed uTAR
- All labels derive from BLASTn hits passing e-value <= 0.01 and bit score >= 50; uTARs without qualifying hits are labeled unknown/undetermined
- Top-hit labels are consistent with homologs reported in the paper (e.g., GTH1, PRPF8, SNX29, ATF7IP2, CLEC14A, SOX5, RUNX1T1, H3F3C, BST2, DUSP1, NATD1, LOC115926763, LOC115920005)
- FWHM coordinates bracket the largest peak of the LOESS-smoothed (span = 0.25) coverage track and fall within the uTAR bounds

## Limitations

- uTARs whose peak sequences have no BLASTn hit passing e-value <= 0.01 / bit score >= 50 remain unknown, so annotation coverage depends on nt database content
- LOESS span = 0.25 and single-largest-peak selection may miss secondary coverage peaks within multi-peak uTARs
- The groHMM step can fail silently (e.g., 'cannot read: chr*_HMM.bed: No such file or directory'), leaving downstream smoothing steps without uTAR coordinates; groHMM and rtracklayer may need manual installation in R

## Evidence

- [methods] Smooth the coverage along each uTAR with locally weighted smoothing (LOESS) using span = 0.25, and locate the full-width at half-maximum (FWHM) coordinates of the largest peak along the uTAR.: [evidence span withheld — anchor status: not_found]
- [intro] Retrieve uTARs that are differentially expressed across cell clusters and annotate them based on nucleotide sequence homology: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [abstract] performs single-cell TAR expression analysis to identify biologically significant TARs: "performs single-cell TAR expression analysis to identify biologically significant TARs, and then annotates TARs using gene homology analysis"
- [readme] Please manually install groHMM and rtracklayer and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"

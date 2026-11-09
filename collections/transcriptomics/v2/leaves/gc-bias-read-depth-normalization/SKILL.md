---
name: gc-bias-read-depth-normalization
description: Use when preparing raw single-cell (or paired bulk) WGS BAMs for read-depth-based
  CNA calling and GC-biased depth must be corrected before segmentation, e.g. as the
  normalize step of a HiScanner pipeline.
when_to_use_negative:
- Data are low-coverage single-cell profiles from DLP+/10x-style assays — use CHISEL
  or Alleloscope, which are tailored to clonal CNAs in low-coverage cancer cells
- The caller used downstream performs its own GC normalization (e.g. SCOPE's own BAM
  processing with a MAPQ 40 threshold) and a second correction would double-correct
- Analysis requires sex chromosomes — the HiScanner pipeline restricts to autosomes
  1-22 and SCAN2 supports only diploid autosomes
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: HiScanner
  role: Implements the normalize step ('hiscanner run --step normalize') that bins
    reads and applies BICseq2-norm-style GC correction
  repo: https://github.com/parklab/HiScanner
- name: BICseq2-norm
  role: Reference method for GC correction and binning with default parameters on
    single-cell and bulk data; also supplies the hg19CRG.100bp mappability track
  repo: https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/BICseq2.html
- name: samtools
  role: Used for BAM-level read processing/filtering (MAPQ >= 30, uniquely mappable
    reads) feeding the normalization
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:2cee9160d637fabc622d9d03b685592d56d1c147063895b12c3cd1ac4ed2ff7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2
  - BICseq2-norm
  - HiScanner
  - Mutect2
  - R
  - SCAN2
  - SECEDO
  - SHAPEIT
  - bcftools
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# gc-bias-read-depth-normalization

## Summary

Binning of single-cell and bulk scWGS/WGS read depth at 100 kb over uniquely mappable loci (MAPQ >= 30, 100mer mappability track) followed by GC-bias correction in the BICseq2-norm style, as implemented in the HiScanner normalize step. This step is prerequisite to joint segmentation and allelic copy-number inference.

## When to use

Use when preparing raw single-cell (or paired bulk) WGS BAMs for read-depth-based CNA calling and GC-biased depth must be corrected before segmentation, e.g. as the normalize step of a HiScanner pipeline.

## When NOT to use

- Data are low-coverage single-cell profiles from DLP+/10x-style assays — use CHISEL or Alleloscope, which are tailored to clonal CNAs in low-coverage cancer cells
- The caller used downstream performs its own GC normalization (e.g. SCOPE's own BAM processing with a MAPQ 40 threshold) and a second correction would double-correct
- Analysis requires sex chromosomes — the HiScanner pipeline restricts to autosomes 1-22 and SCAN2 supports only diploid autosomes

## Inputs

- Single-cell scWGS BAM files (e.g. PTA-amplified, hg19/GRCh37+decoy aligned)
- Paired bulk WGS BAM file per brain/sample
- Split reference genome (per-chromosome FASTA)
- 100mer mappability track (hg19CRG.100bp from BICseq2, or hg38 150mer track)
- HiScanner config.yaml specifying bin size (100 kb), autosomes only, and BAM paths

## Outputs

- GC-corrected, mappability-filtered 100 kb read-depth (RDR) profiles per single cell and per bulk sample
- Normalized per-bin read counts ready for joint BIC-based segmentation (--step segment)

## How to apply

First restrict counting to autosomes and retain only uniquely mappable reads with mapping quality >= 30, using a mappability track appropriate to the reference (hg19CRG.100bp from BICseq2 for hg19; a 150mer track for hg38/GRCh38). Bin read depth at 100 kb (the HiScanner default; SCAN2's ADO step may select a different optimal bin size, e.g. 500 kb for ~0.5x data). Then apply GC correction in the style of BICseq2-norm with default parameters, jointly on single-cell and bulk data so that read depth ratios are comparable. Run this as 'hiscanner run --step normalize' after the ado step; because this is the most time-consuming stage, use the --use-cluster option to parallelize. Rationale: uncorrected GC bias distorts RDR profiles and confounds small-CNA detection, and downstream CHISEL-style comparison showed strong GC bias at 100 kb bins when GC normalization is absent.

## Related tools

- **HiScanner** (Implements the normalize step ('hiscanner run --step normalize') that bins reads and applies BICseq2-norm-style GC correction) — https://github.com/parklab/HiScanner
- **BICseq2-norm** (Reference method for GC correction and binning with default parameters on single-cell and bulk data; also supplies the hg19CRG.100bp mappability track) — https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/BICseq2.html
- **samtools** (Used for BAM-level read processing/filtering (MAPQ >= 30, uniquely mappable reads) feeding the normalization)

## Examples

```
hiscanner run --step normalize --use-cluster
```

## Evaluation signals

- Per-bin read counts come only from autosomes and only from reads with MAPQ >= 30 on uniquely mappable loci (100mer mappability track for hg19)
- RDR profiles no longer correlate with GC content after correction (compare post-normalization RDR-vs-GC against the raw profile)
- 'hiscanner validate' passes on the initialized project with the config.yaml paths, 100 kb bins, autosomes-only settings
- Normalized profiles feed joint BIC-based segmentation that produces consistent segment boundaries across cells (multi-sample extension of BICseq2-seg)
- Downstream '--step cnv' integer copy-number calls are stable; on the neurotypical brain cohort, ~77% of neurons and 53% of oligodendrocytes show at least one detected CNA

## Limitations

- Amplification artifacts in single-cell whole-genome amplification confound CNA detection even at high sequencing depth, causing large read-depth-ratio fluctuations that GC correction alone cannot remove
- GC correction quality depends on the mappability track matching the reference and read length (100mer for hg19, 150mer for hg38); mismatched tracks skew normalization
- The CHISEL comparison showed that lack of GC normalization at 100 kb bins produces strong GC bias — any pipeline skipping this step will overfit or misestimate ploidy

## Evidence

- [discussion] BICseq2-norm25 with default parameters was used to perform GC correction and binning on single-cell and bulk sequencing data: "BICseq2-norm25 with default parameters was used to perform GC correction and binning on single-cell and bulk sequencing data from each individual"
- [discussion] Only uniquely mappable reads with a minimum mapping quality of 30 were considered: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [discussion] GC bias in CHISEL-preprocessed RDR: "there was a strong GC bias in CHISEL-preprocessed RDR for 100 kb bins, likely due to the lack of GC-normalization in the CHISEL pipeline"
- [methods] mappability track download: "wget https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/Mappability/hg19CRG.100bp.tar.gz"
- [readme] HiScanner normalize is the most time-consuming step: "The typical run time for the demo without (`--use-cluster` option) is less than 30 minutes."

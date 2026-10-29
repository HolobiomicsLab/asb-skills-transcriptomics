---
name: read-depth-binning-at-100kb-resolution
description: Use when preparing high-coverage (>20x) single-cell WGS BAMs for CNA
  detection or tool benchmarking at small variant sizes (200 kb–5 Mb), where a fixed
  100 kb bin resolution harmonizes read depth across callers.
when_to_use_negative:
- Data are low-coverage (<0.05x per cell) single-cell libraries (DLP+, 10x CNV Solution,
  DOP-PCR) where CHISEL-style large-bin (e.g. 5 Mb) analysis is the appropriate regime
- Analysis targets sex chromosomes — SCAN2/HiScanner preprocessing supports only diploid
  autosomes 1–22
- The goal is bulk-tumor CNA calling, where mature bulk methods such as ASCAT or Battenberg
  are the standard
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HiScanner
  role: Single-cell CNA caller whose pipeline includes ADO-based optimal bin size
    selection, GC-corrected binning, and joint segmentation of the binned profiles
  repo: github.com/parklab/HiScanner
- name: BICseq2
  role: Performs GC correction and binning (BICseq2-norm) and provides the single-sample
    segmentation algorithm extended to joint multi-sample segmentation (BICseq2-seg)
    plus the 100mer mappability track
- name: SCOPE
  role: BAF-agnostic comparator caller run on the same 100 kb-resolution binned BAM
    inputs with MAPQ threshold 40
- name: CHISEL
  role: BAF-aware comparator caller whose preprocessed RDR at 100 kb bins was evaluated
    for GC bias in the benchmark
  repo: github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic comparator single-cell copy-number analysis tool run on the same
    simulated inputs
  repo: github.com/robertaboukhalil/ginkgo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:1397742bae81beb6a764e7d01a46d5517095626258a247b18ba34fd6317ecb9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BICseq2
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCOPE
  - SCYN
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# Read-depth binning at 100 kb resolution

## Summary

Binning single-cell WGS read depth into 100 kb genomic windows (with GC correction via BICseq2-norm) to enable benchmarkable, allele-specific CNA calling across HiScanner and comparator callers such as SCOPE and CHISEL on high-coverage SDX simulation data.

## When to use

Use when preparing high-coverage (>20x) single-cell WGS BAMs for CNA detection or tool benchmarking at small variant sizes (200 kb–5 Mb), where a fixed 100 kb bin resolution harmonizes read depth across callers.

## When NOT to use

- Data are low-coverage (<0.05x per cell) single-cell libraries (DLP+, 10x CNV Solution, DOP-PCR) where CHISEL-style large-bin (e.g. 5 Mb) analysis is the appropriate regime
- Analysis targets sex chromosomes — SCAN2/HiScanner preprocessing supports only diploid autosomes 1–22
- The goal is bulk-tumor CNA calling, where mature bulk methods such as ASCAT or Battenberg are the standard

## Inputs

- Aligned single-cell WGS BAM files (PTA-amplified, high-coverage ~30x)
- Matched bulk whole-genome sequencing BAM files
- 100mer mappability track (e.g. hg19CRG.100bp from BICseq2, or hg38/GRCh38 150mer from figshare DOI 10.6084/m9.figshare.28370357.v1)
- Reference genome (hg19/GRCh37+decoy or hg38/GRCh38)

## Outputs

- GC-corrected, binned read-depth profiles (BICseq2-norm output) per cell at 100 kb resolution
- Optimal bin size determination from ADO HMM analysis
- Binned RDR inputs for downstream joint segmentation and CNA calling

## How to apply

Convert aligned single-cell and matched bulk BAMs into read counts in fixed genomic bins using BICseq2-norm with default parameters, which performs GC correction and binning simultaneously; restrict to uniquely mappable reads with mapping quality ≥ 30 and to autosomal chromosomes. Use 100 kb bins as the working resolution — SCOPE was run at 100 kb resolution with a MAPQ threshold of 40, and CHISEL's preprocessed RDR was evaluated at 100 kb bins. Because bin size interacts with allelic dropout, run HiScanner's ADO analysis (two-state HMM trained with Baum-Welch on 'ADO'/'non-ADO' states) to select the optimal bin size before segmentation. The rationale is that smaller bins increase effective resolution for sub-megabase CNAs but amplify amplification-artifact noise, so bin size must balance the two; downstream joint segmentation (BICseq2-seg extended to multi-sample with BIC) then merges adjacent bins to call segment boundaries.

## Related tools

- **HiScanner** (Single-cell CNA caller whose pipeline includes ADO-based optimal bin size selection, GC-corrected binning, and joint segmentation of the binned profiles) — github.com/parklab/HiScanner
- **BICseq2** (Performs GC correction and binning (BICseq2-norm) and provides the single-sample segmentation algorithm extended to joint multi-sample segmentation (BICseq2-seg) plus the 100mer mappability track)
- **SCOPE** (BAF-agnostic comparator caller run on the same 100 kb-resolution binned BAM inputs with MAPQ threshold 40)
- **CHISEL** (BAF-aware comparator caller whose preprocessed RDR at 100 kb bins was evaluated for GC bias in the benchmark) — github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic comparator single-cell copy-number analysis tool run on the same simulated inputs) — github.com/robertaboukhalil/ginkgo

## Examples

```
hiscanner run --step ado      # ADO analysis to identify optimal bin size
```

## Evaluation signals

- All callers' CNA calls harmonized to a common bed-like format (start, end, allele-specific copy-number state) and matched to ground-truth simulated events by type and size bin yield plausible TP/FP/FN counts
- Per-size precision/sensitivity reproduce the reported Fig. 2b–i trends: sensitivity ~0.90 and precision ~0.96 for ≥1 Mb single-copy losses, sensitivity ~0.65 at 500 kb single-copy gains
- All callers show reduced performance for CNAs below 500 kb, consistent with the reported finding
- GC bias check: binned RDR should show no strong GC dependence (CHISEL-preprocessed 100 kb RDR showed strong GC bias due to lack of GC normalization)
- Coverage restricted to uniquely mappable reads with MAPQ ≥ 30 and autosomes only, with gHETs retained at bulk VAF 0.1–0.9

## Limitations

- All callers, including HiScanner, struggled with CNAs below 500 kb in the SDX simulations
- Bin-size choice interacts with assay-specific allelic dropout; a fixed bin size without ADO modeling can miss the optimal resolution
- Preprocessed RDR from some comparators (CHISEL at 100 kb bins) carries strong GC bias due to lack of GC normalization, confounding cross-tool comparison

## Evidence

- [discussion] BICseq2-norm performed GC correction and binning: "BICseq2-norm25 with default parameters was used to perform GC correction and binning on single-cell and bulk sequencing data from each individual"
- [discussion] only uniquely mappable reads with MAPQ ≥ 30 were considered: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [discussion] 100 kb resolution with MAPQ threshold 40 for SCOPE: "we used the R package to process BAM files with a 100 kb resolution, MAPQ threshold of 40"
- [discussion] ADO HMM selects optimal bin size: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [discussion] CHISEL 100 kb RDR lacked GC normalization: "there was a strong GC bias in CHISEL-preprocessed RDR for 100 kb bins, likely due to the lack of GC-normalization in the CHISEL pipeline"
- [methods] HiScanner ADO step invocation from README: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"

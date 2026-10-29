---
name: cross-caller-benchmark-comparison
description: Use when comparing a new or existing single-cell CNA caller against HiScanner,
  Ginkgo, CHISEL, SCYN, and SCOPE on high-coverage (≥20x) PTA-amplified scWGS with
  simulated ground-truth CNAs or a known aneuploid single cell.
when_to_use_negative:
- Input is low-coverage (<0.05x per cell) scWGS from DLP+ or 10x Genomics CNV assays
  — CHISEL/Alleloscope are designed for that regime, and HiScanner's ADO-based bin-size
  selection assumes high-coverage PTA data
- Goal is germline or clonal variant genotyping rather than somatic CNA benchmarking
  — SCAN2 excludes candidates with any read support in matched bulk
- Only a single caller's output exists with no matched bulk, ground-truth simulations,
  or comparator calls — precision/sensitivity and cross-caller concordance cannot
  be computed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HiScanner
  role: Primary caller under evaluation; full RDR+BAF pipeline with joint BIC segmentation
    at 100 kb bins
  repo: github.com/parklab/HiScanner
- name: SCAN2
  role: Preprocessing; somatic SNV/indel genotyping of PTA-amplified single cells
    whose output HiScanner checks in the 'snp' step
  repo: github.com/parklab/SCAN2
- name: Ginkgo
  role: BAF-agnostic comparator producing RDR-based copy-number profiles from the
    same single-cell BAMs
  repo: github.com/robertaboukhalil/ginkgo
- name: CHISEL
  role: BAF-aware comparator run via its bioconda package at 100 kb and 5 Mb bin settings
  repo: github.com/raphael-group/chisel
- name: SCOPE
  role: BAF-agnostic R-package comparator run at 100 kb resolution with MAPQ threshold
    40
- name: SCYN
  role: BAF-agnostic comparator run on the same single-cell inputs
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:6bac37124c958fddda3c99bad9484539a91b49c1b5ac85552b467c98d302da42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCAN2
  - SCOPE
  - SCYN
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-caller-benchmark-comparison

## Summary

Benchmarks HiScanner against four single-cell CNA callers (Ginkgo, CHISEL, SCYN, SCOPE) on simulated SDX data and a single 29x PTA-amplified aneuploid neuron (5823PFC-B), quantifying precision, sensitivity, and breakpoint density per caller across CNA types and sizes (200 kb–5 Mb).

## When to use

Use when comparing a new or existing single-cell CNA caller against HiScanner, Ginkgo, CHISEL, SCYN, and SCOPE on high-coverage (≥20x) PTA-amplified scWGS with simulated ground-truth CNAs or a known aneuploid single cell.

## When NOT to use

- Input is low-coverage (<0.05x per cell) scWGS from DLP+ or 10x Genomics CNV assays — CHISEL/Alleloscope are designed for that regime, and HiScanner's ADO-based bin-size selection assumes high-coverage PTA data
- Goal is germline or clonal variant genotyping rather than somatic CNA benchmarking — SCAN2 excludes candidates with any read support in matched bulk
- Only a single caller's output exists with no matched bulk, ground-truth simulations, or comparator calls — precision/sensitivity and cross-caller concordance cannot be computed

## Inputs

- Simulated SDX benchmark single-cell BAMs and matched bulk WGS at 30x coverage with ground-truth CNAs of 200 kb–5 Mb
- Single-cell BAM for aneuploid neuron 5823PFC-B (29x, PTA-amplified) with paired bulk BAM
- Split reference genome and 100mer mappability track (hg19CRG.100bp from BICseq2)
- SCAN2 somatic SNV/indel output for SNP checking and phasing

## Outputs

- benchmark_metrics.csv with precision and sensitivity per caller, per CNA size, and per CNA type
- Comparison figure of per-method copy-number tracks across the aneuploid neuron
- Table of breakpoint density per method (e.g., CHISEL: 10,322 breakpoints, 0.38 per 100 kb)
- Harmonized bed-like CNA call files with start, end, and allele-specific copy-number state

## How to apply

Run HiScanner's full standard pipeline ('hiscanner run --step all': SNP checking of SCAN2 output, phasing, ADO-based bin-size selection, GC-corrected read-depth normalization via BICseq2-norm, joint BIC segmentation, CNV calling) at 100 kb bin resolution on simulated BAMs and matched bulk WGS at 30x. Run CHISEL (BAF-aware, via bioconda) at both 100 kb and an optimized 5 Mb bin configuration, and the BAF-agnostic callers Ginkgo (command-line version), SCOPE (100 kb resolution, MAPQ threshold 40), and SCYN on identical inputs. Harmonize all callers' calls to a common bed-like format with start, end, and allele-specific copy-number state, then match against ground-truth events by CNA type (single-copy loss, single-copy gain, and other simulated classes) and size bin to count TP/FP/FN and compute precision (TP/(TP+FP)) and sensitivity (TP/(TP+FN)). For the real-data comparison, apply all five callers to the aneuploid neuron 5823PFC-B single-cell BAM with its paired bulk, split reference, and 100mer mappability track, then compile per-method copy-number tracks, tabulate breakpoints per 100 kb, and check BAF shifts in HiScanner loss calls.

## Related tools

- **HiScanner** (Primary caller under evaluation; full RDR+BAF pipeline with joint BIC segmentation at 100 kb bins) — github.com/parklab/HiScanner
- **SCAN2** (Preprocessing; somatic SNV/indel genotyping of PTA-amplified single cells whose output HiScanner checks in the 'snp' step) — github.com/parklab/SCAN2
- **Ginkgo** (BAF-agnostic comparator producing RDR-based copy-number profiles from the same single-cell BAMs) — github.com/robertaboukhalil/ginkgo
- **CHISEL** (BAF-aware comparator run via its bioconda package at 100 kb and 5 Mb bin settings) — github.com/raphael-group/chisel
- **SCOPE** (BAF-agnostic R-package comparator run at 100 kb resolution with MAPQ threshold 40)
- **SCYN** (BAF-agnostic comparator run on the same single-cell inputs)

## Examples

```
hiscanner run --step all && conda install chisel  # run HiScanner full pipeline, then install CHISEL comparator from bioconda
```

## Evaluation signals

- Per-size trends reproduce Fig. 2b-i: HiScanner sensitivity ~0.90 and precision ~0.96 for ≥1 Mb single-copy losses, dropping to ~0.65 sensitivity at 500 kb single-copy gains
- All five callers show reduced sensitivity for CNAs below 500 kb
- On neuron 5823PFC-B, HiScanner detects small CNAs with fewer spurious breakpoints than the four comparators, and CHISEL shows the most fragmented calls (~10,322 breakpoints)
- HiScanner loss calls concordant with BAF evidence show the expected BAF shifts; centromeric regions remain stable across callers
- All callers' calls harmonize to a common bed-like schema (start, end, allele-specific copy-number state) with no state mismatches

## Limitations

- CHISEL required parameter fine-tuning and showed high-ploidy predictions, overfitting, and strong GC bias in RDR at 100 kb bins due to lack of GC normalization in its pipeline
- FDR estimates are conservative since FDR increases as CNA size decreases
- Comparator performance varied by variant type and size rather than showing uniform behavior

## Evidence

- [intro] Objective: benchmark HiScanner against four existing single-cell CNA methods at small size scales (200 kb–5 Mb): "HiScanner outperforms four representative single-cell CNA methods—CHISEL (BAF-aware), Ginkgo, SCOPE, and SCYN (all BAF-agnostic methods)19,22–24—in real and simulated data with greater sensitivity"
- [discussion] SCOPE comparator parameters: "we used the R package to process BAM files with a 100 kb resolution, MAPQ threshold of 40"
- [abstract] In simulated data, HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes: "In simulated data, HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes"
- [discussion] Ginkgo source: "Ginkgo’s command line version was downloaded from https://github.com/robertaboukhalil/ginkgo"
- [discussion] CHISEL GC bias caveat: "there was a strong GC bias in CHISEL-preprocessed RDR for 100 kb bins, likely due to the lack of GC-normalization in the CHISEL pipeline"
- [readme] HiScanner pipeline steps from README: "Standard Pipeline (RDR + BAF): Full analysis using both read depth ratios (RDR) and B-allele frequencies (BAF)"

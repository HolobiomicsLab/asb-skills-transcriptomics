---
name: gc-bias-correction-evaluation
description: Use when preprocessing scWGS read-depth data for CNA calling, or when
  benchmarking CNA callers whose segmentation shows bin-level read-depth trends correlated
  with GC content, e.g. CHISEL-preprocessed 100 kb RDR.
when_to_use_negative:
- The caller's pipeline already performs GC normalization on the same bins (double
  correction can distort RDR).
- Data are not read-depth based (e.g. BAF-only or SNV-based analyses where GC bias
  in depth is irrelevant).
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: HiScanner
  role: Performs nucleotide-resolution read-depth normalization correcting GC bias,
    followed by joint BIC segmentation and CNV calling
  repo: github.com/parklab/HiScanner
- name: BICseq2-norm
  role: Performs GC correction and binning on single-cell and bulk sequencing data
    with default parameters
- name: CHISEL
  role: Benchmarked caller whose 100 kb RDR lacks GC normalization, used as a negative
    example of GC bias in caller-preprocessed read depth
  repo: github.com/raphael-group/chisel
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:3dbeedc53d55b4e49c0a07d5be11ddd051fd2a1274c3a31ab89ebd2622cb8c43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BICseq2-norm
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCAN2
  - SCOPE
  - SCYN
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# GC bias correction evaluation

## Summary

Evaluate and correct GC-content bias in single-cell read-depth ratios before copy-number (CNA) calling, as done in HiScanner via nucleotide-resolution GC normalization with BICseq2-norm, and use residual GC bias as a diagnostic when benchmarking competing callers such as CHISEL at 100 kb bins.

## When to use

Use when preprocessing scWGS read-depth data for CNA calling, or when benchmarking CNA callers whose segmentation shows bin-level read-depth trends correlated with GC content, e.g. CHISEL-preprocessed 100 kb RDR.

## When NOT to use

- The caller's pipeline already performs GC normalization on the same bins (double correction can distort RDR).
- Data are not read-depth based (e.g. BAF-only or SNV-based analyses where GC bias in depth is irrelevant).

## Inputs

- Single-cell scWGS BAM (e.g. PTA-amplified 29x neuron 5823PFC-B)
- Paired bulk WGS BAM
- Per-bin read counts or read-depth ratios (e.g. 100 kb bins)
- Split reference genome (hg19/GRCh37+decoy) for GC content computation

## Outputs

- GC-corrected, binned read-depth ratio tracks per cell
- Caller-preprocessed RDR vs GC-content diagnostic plots
- Bias-corrected inputs for joint BIC segmentation and CNA calling

## How to apply

Correct GC bias on single-cell and bulk BAM-derived read counts using BICseq2-norm with default parameters before binning and segmentation; HiScanner performs read-depth normalization at nucleotide resolution so that downstream joint BIC segmentation and RDR/BAF-based copy-number calling operate on bias-corrected profiles. When comparing methods, plot or regress caller-preprocessed RDR against GC content per bin: in the neuron 5823PFC-B comparison, CHISEL-preprocessed RDR at 100 kb bins showed strong GC bias attributed to the lack of GC normalization in the CHISEL pipeline, which contributed to its fragmented calls (10,322 breakpoints, 0.38 per 100 kb). Treat residual GC bias as evidence that a caller's input normalization is inadequate and as a caveat when interpreting its breakpoint density relative to more conservative callers (HiScanner, Ginkgo, SCYN, SCOPE).

## Related tools

- **HiScanner** (Performs nucleotide-resolution read-depth normalization correcting GC bias, followed by joint BIC segmentation and CNV calling) — github.com/parklab/HiScanner
- **BICseq2-norm** (Performs GC correction and binning on single-cell and bulk sequencing data with default parameters)
- **CHISEL** (Benchmarked caller whose 100 kb RDR lacks GC normalization, used as a negative example of GC bias in caller-preprocessed read depth) — github.com/raphael-group/chisel

## Examples

```
hiscanner run --step all  # GC-corrected RDR + BAF pipeline; compare against raw CHISEL 100 kb RDR for GC bias
```

## Evaluation signals

- Regression of corrected RDR against GC content per bin shows no systematic trend (flat slope, low R^2).
- Comparison figure reproduces Fig. 3: HiScanner detects small CNAs with fewer spurious breakpoints than Ginkgo, CHISEL, SCYN, and SCOPE.
- CHISEL 100 kb run reproduces the reported GC bias in its preprocessed RDR, distinguishing normalization artifacts from genuine CNA signal.
- Breakpoint densities per method are tabulated (e.g. CHISEL 10,322 breakpoints, 0.38 per 100 kb) and biased-caller inflation is flagged.
- HiScanner loss calls concordant with BAF evidence show expected BAF shifts, confirming bias correction did not remove true signal.

## Limitations

- CHISEL-preprocessed RDR at 100 kb bins had strong GC bias likely due to the lack of GC normalization in the CHISEL pipeline; comparisons at that setting conflate caller algorithm with normalization quality.
- CHISEL showed a tendency for high ploidy predictions and overfitting, requiring parameter fine-tuning at an optimized 5 Mb bin configuration.
- GC correction cannot rescue amplification artifacts from non-uniform WGA, which confound CNA detection even at high sequencing depth.

## Evidence

- [intro] Nucleotide-resolution read depth normalization correcting GC bias: "HiScanner performs read depth normalization at nucleotide resolution, followed by joint segmentation of single-cell profiles using the Bayesian information criterion (BIC)"
- [discussion] GC correction with BICseq2-norm: "BICseq2-norm25 with default parameters was used to perform GC correction and binning on single-cell and bulk sequencing data from each individual"
- [discussion] CHISEL GC bias at 100 kb: "there was a strong GC bias in CHISEL-preprocessed RDR for 100 kb bins, likely due to the lack of GC-normalization in the CHISEL pipeline"
- [discussion] CHISEL overfitting tendency: "we noticed a tendency for high ploidy predictions and overfitting, similar to what was reported previously21"
- [results] CHISEL fragmented calls in aneuploid neuron: [evidence span withheld — anchor status: not_found]

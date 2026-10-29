---
name: coverage-downsampling-analysis
description: Decision support for choosing bin size and sequencing depth before running
  a scWGS CNA calling pipeline, especially when balancing detection of small (<500
  kb) or single-copy events against per-bin read-depth noise at lower coverage.
when_to_use_negative:
- Data is real tumor/brain scWGS rather than simulated truth-matched data — precision/sensitivity
  cannot be computed without ground truth
- The pipeline is allele-specific-only by design (e.g., no phased gHETs or paired
  bulk available for BAF), so depth effects on BAF-driven performance are not testable
- Depth is above 30x on PTA-amplified cells where amplification artifacts, not coverage,
  dominate CNA-detection error
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: HiScanner
  role: Primary caller under test; run with --step all over bin-size and coverage
    grids
  repo: https://github.com/parklab/HiScanner
- name: CHISEL
  role: BAF-aware single-cell CNA comparator run on the same simulated inputs
  repo: https://github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic single-cell CNA comparator run on the same simulated inputs
  repo: https://github.com/robertaboukhalil/ginkgo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:1f30a0d1ac98bbb72a4337c898e746f626cefff44fbac86dbaad0483f12b19c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Coverage-downsampling-analysis

## Summary

A benchmarking protocol for quantifying how sequencing depth (0.5x, 10x, 30x) and bin size (50/100/500 kb) jointly affect a single-cell CNA caller's precision and sensitivity, using HiScanner on SDX-simulated scWGS data to identify the optimal parameter combination (100 kb bins at 30x).

## When to use

Decision support for choosing bin size and sequencing depth before running a scWGS CNA calling pipeline, especially when balancing detection of small (<500 kb) or single-copy events against per-bin read-depth noise at lower coverage.

## When NOT to use

- Data is real tumor/brain scWGS rather than simulated truth-matched data — precision/sensitivity cannot be computed without ground truth
- The pipeline is allele-specific-only by design (e.g., no phased gHETs or paired bulk available for BAF), so depth effects on BAF-driven performance are not testable
- Depth is above 30x on PTA-amplified cells where amplification artifacts, not coverage, dominate CNA-detection error

## Inputs

- SDX-simulated paired bulk + single-cell scWGS BAM files
- Coverage-downsampled simulation BAMs (0.5x, 10x, 30x)
- HiScanner config.yaml and metadata table (bamID, bam, singlecell columns)
- Simulated truth CNA event coordinates for matching
- Reference genome with 100mer/150mer mappability tracks (e.g., hg19CRG.100bp, hg38 150mer)

## Outputs

- Per-condition (coverage x bin size) precision and sensitivity tables stratified by CNA type and size class
- HiScanner CNV segment calls for each bin size and depth
- Comparator (CHISEL, Ginkgo) CNV calls on identical inputs
- Identified optimal parameter setting (expected: 100 kb bins at 30x, cf. Supplementary Fig. 3)

## How to apply

Initialize a HiScanner project ('hiscanner init') and validate config.yaml plus a metadata table with bamID, bam, and singlecell columns via 'hiscanner validate'. Run the full pipeline ('hiscanner run --step all', executing GC-corrected read-depth normalization, joint BIC-based segmentation, and CNV calling) over a bin-size grid (50/100/500 kb) on 30x SDX simulations, then repeat the normalize, segment, and cnv steps on downsampled 10x and 0.5x simulations. Run comparators on identical inputs — CHISEL (BAF-aware, with phasing of germline SNPs) and Ginkgo (BAF-agnostic) — under matched conditions. Match each called CNV segment against simulated truth events and compute precision and sensitivity per CNA type and size class per condition; assemble summary tables and pick the condition maximizing the precision–sensitivity trade-off. The rationale is that allele-specific methods like HiScanner use BAF from reference-based phasing, whose informativeness degrades with depth: 50 kb bins raise sub-500 kb sensitivity but add noise that cuts precision, 500 kb bins miss small variants, and 0.5x coverage substantially lowers performance for single-copy and small events.

## Related tools

- **HiScanner** (Primary caller under test; run with --step all over bin-size and coverage grids) — https://github.com/parklab/HiScanner
- **CHISEL** (BAF-aware single-cell CNA comparator run on the same simulated inputs) — https://github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic single-cell CNA comparator run on the same simulated inputs) — https://github.com/robertaboukhalil/ginkgo

## Examples

```
hiscanner init --output ./my_project; hiscanner validate; hiscanner run --step all
```

## Evaluation signals

- The identified optimum reproduces the reported result: 100 kb bins at 30x coverage, consistent with Supplementary Fig. 3
- 'hiscanner validate' passes with no errors on config.yaml and the bamID/bam/singlecell metadata table before any run
- Precision at 50 kb bins is measurably lower than at 100 kb bins (noise penalty), while 500 kb bins show worse sensitivity for small variants
- 0.5x coverage conditions show substantially lower sensitivity than 10x/30x, particularly for single-copy events and smaller variants
- Per-condition tables contain one precision/sensitivity row per (coverage x bin size) combination matched against simulated truth events

## Limitations

- 0.5x coverage substantially lowers performance, particularly for single-copy events and smaller variants, so conclusions at 0.5x may not extend to shallower or single-cell-amplification-free data
- 50 kb bins trade improved sub-500 kb sensitivity for reduced precision via noise — the bin-size optimum is assay- and coverage-dependent
- HiScanner considers assay-specific allelic dropout (ADO) patterns to find an optimal bin size, so a bin size tuned on simulated data may not transfer to a different WGA assay
- Comparator fine-tuning is non-trivial: CHISEL showed high-ploidy predictions, overfitting, and GC bias at 100 kb bins due to lack of GC normalization

## Evidence

- [results] Optimal bin-size grid finding: [evidence span withheld — anchor status: not_found]
- [intro] ADO-driven bin selection rationale: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
- [results] Depth-degradation finding: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline core method: "HiScanner performs read depth normalization at nucleotide resolution, followed by joint segmentation of single-cell profiles using the Bayesian information criterion (BIC)"
- [intro] Comparator benchmark scope: "HiScanner outperforms four representative single-cell CNA methods—CHISEL (BAF-aware), Ginkgo, SCOPE, and SCYN (all BAF-agnostic methods)19,22–24—in real and simulated data"
- [readme] Demo/init invocation: [evidence span withheld — anchor status: external_doc]

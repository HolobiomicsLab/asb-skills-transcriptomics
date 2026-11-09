---
name: bin-size-optimization-scwgs
description: 'Use when configuring scWGS CNA calling resolution: you must choose a
  bin size for RDR/BAF-based single-cell copy-number analysis or decide if a dataset''s
  coverage (e.g., ~0.5× low-coverage vs >20× PTA-amplified) supports sub-Mb CNA detection.'
when_to_use_negative:
- Input data are not scWGS with paired bulk for phasing — e.g., bulk-only tumor samples,
  where bulk callers such as ASCAT or Battenberg are the appropriate choice
- The amplification assay or coverage regime differs fundamentally from the tested
  grid (e.g., non-PTA single-cell WGA with severe amplification artifacts), so the
  simulated optimum may not transfer without re-running the ADO analysis
- The goal is point-mutation (SNV/indel) calling rather than copy-number detection
  — use SCAN2 instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HiScanner
  role: Primary CNA caller whose bin-size and depth sensitivity is being swept; performs
    GC-corrected read-depth normalization, joint BIC-based segmentation, and RDR+BAF
    integer copy-number calling
  repo: github.com/parklab/HiScanner
- name: CHISEL
  role: BAF-aware single-cell comparator run on the identical simulated inputs under
    matched conditions
  repo: github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic single-cell comparator run on the identical simulated inputs
    under matched conditions
  repo: github.com/robertaboukhalil/ginkgo
- name: SCAN2
  role: Somatic SNV/indel genotyper that must be run separately before HiScanner (supplies
    'hiscanner run --step snp' and '--step phase' inputs)
  repo: https://github.com/parklab/SCAN2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:a07d144d340c48c2e3f87bf929ae1d698613dac25d72651316a7e931e0279b0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCAN2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bin-size-optimization-scwgs

## Summary

A benchmarking protocol that sweeps HiScanner bin sizes (50/100/500 kb) and sequencing depths (0.5×/10×/30×) on SDX-simulated single-cell and paired bulk scWGS data to find the bin-size/coverage combination that best trades precision against sensitivity for CNA detection. It grounds HiScanner's ADO-based bin-size selection in quantitative precision/sensitivity curves per CNA type and size class.

## When to use

Use when configuring scWGS CNA calling resolution: you must choose a bin size for RDR/BAF-based single-cell copy-number analysis or decide if a dataset's coverage (e.g., ~0.5× low-coverage vs >20× PTA-amplified) supports sub-Mb CNA detection.

## When NOT to use

- Input data are not scWGS with paired bulk for phasing — e.g., bulk-only tumor samples, where bulk callers such as ASCAT or Battenberg are the appropriate choice
- The amplification assay or coverage regime differs fundamentally from the tested grid (e.g., non-PTA single-cell WGA with severe amplification artifacts), so the simulated optimum may not transfer without re-running the ADO analysis
- The goal is point-mutation (SNV/indel) calling rather than copy-number detection — use SCAN2 instead

## Inputs

- SDX-simulated single-cell scWGS BAM files at 30×, 10×, and 0.5× coverage
- Paired bulk scWGS/WGS BAM files for phasing gHETs
- HiScanner config.yaml and metadata table (bamID, bam, singlecell columns)
- Simulated truth-event annotations (CNA type and size classes) for precision/sensitivity matching
- Mappability tracks (e.g., hg19CRG.100bp from BICseq2) for restricting to uniquely mappable loci

## Outputs

- Per-condition (coverage × bin size) precision and sensitivity tables stratified by CNA type and size class
- Called CNV segments per cell per condition from HiScanner, CHISEL, and Ginkgo
- Identified optimal parameter setting (100 kb bins at 30× depth in SDX simulations), verifiable against Supplementary Fig. 3

## How to apply

Initialize a HiScanner project ('hiscanner init --output ./my_project'), populate config.yaml and the metadata table (bamID, bam, singlecell columns), and validate with 'hiscanner validate'. Run the full pipeline ('hiscanner run --step all', covering GC-corrected read-depth normalization, joint BIC-based segmentation, and CNV calling) on 30× SDX simulations across the 50/100/500 kb bin grid, then repeat the normalize/segment/cnv steps on 10× and 0.5× downsampled simulations. Run comparators (CHISEL, BAF-aware; Ginkgo, BAF-agnostic) on identical inputs, match each condition's called segments against simulated truth events, and compute precision and sensitivity per CNA type and size class. Select the condition maximizing the precision–sensitivity trade-off; in SDX simulations this is 100 kb bins at 30×, since 50 kb bins gain sub-500 kb sensitivity but lose precision to noise, 500 kb bins miss small variants, and 0.5× coverage performs substantially worse, especially for single-copy events and smaller variants. Rationale: HiScanner also considers assay-specific allelic dropout (ADO) patterns — trained via a two-state HMM ('ADO'/'non-ADO') with Baum-Welch — to find an optimal bin size, so the sweep should be re-run per amplification assay.

## Related tools

- **HiScanner** (Primary CNA caller whose bin-size and depth sensitivity is being swept; performs GC-corrected read-depth normalization, joint BIC-based segmentation, and RDR+BAF integer copy-number calling) — github.com/parklab/HiScanner
- **CHISEL** (BAF-aware single-cell comparator run on the identical simulated inputs under matched conditions) — github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic single-cell comparator run on the identical simulated inputs under matched conditions) — github.com/robertaboukhalil/ginkgo
- **SCAN2** (Somatic SNV/indel genotyper that must be run separately before HiScanner (supplies 'hiscanner run --step snp' and '--step phase' inputs)) — https://github.com/parklab/SCAN2

## Examples

```
hiscanner init --output ./my_project && hiscanner validate && hiscanner run --step all
```

## Evaluation signals

- The identified optimum reproduces the reported result: 100 kb bins at 30× depth, consistent with Supplementary Fig. 3
- 50 kb-bin conditions show improved sub-500 kb sensitivity but reduced precision relative to 100 kb (noise-driven false positives), while 500 kb bins degrade small-variant detection
- 0.5× coverage conditions show substantially lower performance than 10×/30×, particularly for single-copy events and smaller variants
- 'hiscanner validate' passes on the config.yaml and metadata table before any pipeline run
- Matched truth-event comparisons show HiScanner outperforming CHISEL and Ginkgo across CNA types and sizes on the same inputs

## Limitations

- 0.5× coverage (as in the 179-cell meningioma dataset) limits detection of single-copy and small events, so low-coverage datasets cannot support the high-resolution optimum
- Bin-size optimum depends on assay-specific allelic dropout patterns; the SDX-derived optimum may not transfer to other WGA chemistries
- CHISEL comparisons required parameter fine-tuning; CHISEL showed high-ploidy predictions, overfitting, and GC bias at 100 kb bins due to lack of GC normalization in its pipeline

## Evidence

- [supplementary] 100 kb bins at 30× depth provided the optimal balance of precision and sensitivity: [evidence span withheld — anchor status: not_found]
- [intro] HiScanner considers assay-specific ADO patterns to select bin size: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
- [discussion] HMM-based ADO analysis for minimum bin size: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [abstract] HiScanner pipeline steps: "HiScanner performs read depth normalization at nucleotide resolution, followed by joint segmentation of single-cell profiles using the Bayesian information criterion (BIC)"
- [readme] HiScanner CLI steps from README/demo: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"

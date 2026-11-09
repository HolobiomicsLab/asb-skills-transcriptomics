---
name: across-plate-uncertainty-aggregation
description: Use when a per-plate swapping-fraction estimate has been derived (gradient
  of swapped reads vs. reads available for swapping) and a single pooled fraction
  with an uncertainty must be reported for the experiment.
when_to_use_negative:
- There is only a single plate or a single library, so no across-plate replicate exists
  to define a standard deviation
- The underlying data come from droplet/10x Genomics experiments where swapped molecules
  are excluded algorithmically rather than estimated by a regression gradient
- Libraries were sequenced on non-patterned-flow-cell instruments such as the HiSeq
  2500, where swapping is negligible and the estimate should be near zero
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: R
  role: Used to fit the per-plate linear regressions whose gradients are aggregated,
    and to compute the mean and standard deviation of swapping fractions across plates
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
- name: HiSeq 4000
  role: Patterned-flow-cell instrument whose lane-swapping produces the barcode-mislabelled
    reads being quantified and aggregated across plates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:1eb132594509af60325d88f4ffb5c48f50183b2ed91bc067c7513cb550b44dd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - HiSeq 4000
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# across-plate-uncertainty-aggregation

## Summary

Aggregates per-plate estimates of the barcode-swapped read fraction by computing the mean and standard deviation of the regression-gradient swapping fractions across plates, yielding a central estimate with an empirical uncertainty (e.g. 2.18 ± 0.08% for Richard et al. libraries sequenced on the HiSeq 4000).

## When to use

Use when a per-plate swapping-fraction estimate has been derived (gradient of swapped reads vs. reads available for swapping) and a single pooled fraction with an uncertainty must be reported for the experiment.

## When NOT to use

- There is only a single plate or a single library, so no across-plate replicate exists to define a standard deviation
- The underlying data come from droplet/10x Genomics experiments where swapped molecules are excluded algorithmically rather than estimated by a regression gradient
- Libraries were sequenced on non-patterned-flow-cell instruments such as the HiSeq 2500, where swapping is negligible and the estimate should be near zero

## Inputs

- Per-plate swapping-fraction estimates (regression gradients) for the two plates of the Richard et al. mouse T-cell dataset (ArrayExpress E-MTAB-6843, HiSeq 4000)
- Standard deviations or standard errors of each per-plate regression gradient (optional)
- Reference HiSeq 4000 swapping fraction (~2.5%) for validation

## Outputs

- A pooled swapped-read fraction estimate with across-plate standard deviation (e.g. 2.18 ± 0.08%), e.g. written to swapping_fraction_estimate.txt
- Deviation of the estimate from the ~2.5% published HiSeq 4000 reference value

## How to apply

After fitting, for each plate, the simple linear regression of observed swapped reads (impossible sample-barcode combinations) on reads available for swapping, take each fitted gradient as that plate's swapping fraction. Collect the per-plate fractions (two plates in the Richard et al. mouse T-cell dataset, E-MTAB-6843, HiSeq 4000) and compute their arithmetic mean as the experiment-level swapping fraction and their standard deviation as the uncertainty. Write the estimate and uncertainty (e.g. to swapping_fraction_estimate.txt, e.g. 2.18 ± 0.08%) and validate against the published reference of approximately 2.5% HiSeq 4000 mislabelling. The rationale is that the two plates are independent libraries whose gradients provide replicate estimates of the same instrument-level swapping rate, so cross-plate spread captures biological/library-preparation variability rather than pure measurement noise.

## Related tools

- **R** (Used to fit the per-plate linear regressions whose gradients are aggregated, and to compute the mean and standard deviation of swapping fractions across plates) — https://github.com/MarioniLab/BarcodeSwapping2017
- **HiSeq 4000** (Patterned-flow-cell instrument whose lane-swapping produces the barcode-mislabelled reads being quantified and aggregated across plates)

## Examples

```
Rscript -e 'f <- read.table("plate_fractions.txt")$V1; cat(sprintf("%.2f +- %.2f%%\n", 100*mean(f), 100*sd(f)))'
```

## Evaluation signals

- The pooled estimate with uncertainty should approximate 2.5% for HiSeq 4000 data (e.g. 2.18 ± 0.08%); report the deviation from this reference value
- Re-running the aggregation on the same per-plate gradients must reproduce the identical mean and standard deviation (deterministic arithmetic)
- The number of fractions aggregated must equal the number of plates processed (two for the Richard et al. two-plate dataset)
- As a negative control, the same aggregation applied to HiSeq 2500-sequenced libraries should yield a swapping fraction near zero
- The standard deviation must be consistent with the spread of the individual plate-level gradients, not inflated or set to zero arbitrarily

## Limitations

- The standard deviation across only two plates is a very low-replicate uncertainty estimate and may understate true variability
- Key details of the swapping-fraction estimation (per-plate regression and across-plate aggregation) are deferred to Supplementary Notes 3 and 4, which are not available in the extracted document
- No correlation was observed between the swapped fraction of reads and free barcode concentration, so this aggregation does not model library-preparation covariates

## Evidence

- [discussion] Mean and standard deviation of swapping fractions computed across plates: "The mean and standard deviation of swapping fractions were then computed across plates."
- [discussion] Gradient of fitted line estimates the swapped-read fraction: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [intro] Objective of quantifying swapped fraction in plate-based datasets: "we robustly quantify the fraction of swapped reads in each of two plate-based single-cell RNA sequencing datasets"
- [discussion] Reproducible ~2.5% HiSeq 4000 estimate as validation reference: "we have reproducibly estimated the fraction of barcode-swapped reads on the HiSeq 4000 to be approximately 2.5%, which is lower than previously reported"
- [readme] Repository hosting the detailed report and analysis code: [evidence span withheld — anchor status: external_doc]

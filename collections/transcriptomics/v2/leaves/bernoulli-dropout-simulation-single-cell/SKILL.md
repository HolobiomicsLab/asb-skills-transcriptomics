---
name: bernoulli-dropout-simulation-single-cell
description: Use when benchmarking single-cell clustering robustness on in silico
  mixtures (e.g., GM12878/K562) by injecting controlled dropout into expression E
  and openness O matrices at rates such as 0, 0.5, 0.8, 0.9.
when_to_use_negative:
- Input is real single-cell data with unknown/irreproducible dropout rather than a
  controlled simulation where true subpopulation labels are known
- The analysis goal is deconvolution or loop deconvolution itself rather than quantifying
  how dropout affects clustering error
- The single-cell modalities are jointly profiled (e.g., sci-CAR); the article notes
  such joint data types are not yet incorporated into DC3's cost function
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Downstream coupled-clustering method whose classification error is evaluated
    on the Bernoulli-dropout-masked E and O matrices
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:fe2bccf855673ed7998a712c2d60ef607e4a1ae21bd04a37d8f53b11fc133eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bernoulli-dropout-simulation-single-cell

## Summary

Simulates missing-value dropout in scRNA-seq expression matrix E and scATAC-seq openness matrix O by Bernoulli sampling per site (gene or enhancer), enabling controlled benchmarking of coupled-clustering methods such as DC3 against NMF baselines at varying dropout severities.

## When to use

Use when benchmarking single-cell clustering robustness on in silico mixtures (e.g., GM12878/K562) by injecting controlled dropout into expression E and openness O matrices at rates such as 0, 0.5, 0.8, 0.9.

## When NOT to use

- Input is real single-cell data with unknown/irreproducible dropout rather than a controlled simulation where true subpopulation labels are known
- The analysis goal is deconvolution or loop deconvolution itself rather than quantifying how dropout affects clustering error
- The single-cell modalities are jointly profiled (e.g., sci-CAR); the article notes such joint data types are not yet incorporated into DC3's cost function

## Inputs

- scRNA-seq expression matrix E (genes x cells) for the in silico mixture
- scATAC-seq openness matrix O (enhancers x cells) for the in silico mixture
- True subpopulation labels (e.g., GM12878 vs K562) per cell
- Set of dropout rates to sweep (e.g., [0, 0.5, 0.8, 0.9])

## Outputs

- Dropout-masked E and O matrices per rate
- Mean classification error rates with standard deviations per method per dropout rate (e.g., dropout_clustering_error_rates.csv)

## How to apply

Build the in silico mixture matrices from public data (e.g., E: 146 cells, 73 per cell line for GM12878/K562; O: 746 cells, 373 per cell line). For each dropout rate in [0, 0.5, 0.8, 0.9], draw an independent Bernoulli per site (gene in E, enhancer in O) and set dropped entries to zero, masking the expression and openness matrices symmetrically. At each dropout level, run DC3 with scRNA-seq + scATAC-seq + bulk HiChIP inputs and the coupled-NMF and per-datatype NMF baselines, using 50 independent runs with different random initializations to average out initialization variance. Compute classification error between true subpopulation labels (GM12878 vs K562) and predicted cluster assignments, averaged over scRNA-seq and scATAC-seq samples and over the 50 runs, recording mean and standard deviation per method per rate. The rationale is that dropout degrades single-cell signals, so retaining bulk HiChIP loop data should stabilize clustering — the paper reports DC3 reduces classification error by more than four folds at dropout rates of 80% or higher.

## Related tools

- **DC3** (Downstream coupled-clustering method whose classification error is evaluated on the Bernoulli-dropout-masked E and O matrices) — github.com/SUwonglab/DC3

## Examples

```
E_drop = E * np.random.binomial(1, 1-p, E.shape); O_drop = O * np.random.binomial(1, 1-p, O.shape)
```

## Evaluation signals

- At 0% dropout, DC3 and coupled NMF error rates should be comparable and low, since dropout masking has no effect
- Incorporating loop data (DC3 vs coupled NMF) must reduce classification error by more than four folds at 80% dropout or higher, matching the paper's reported result
- Per-site dropout masking should preserve matrix dimensions (146 cells in E, 746 cells in O) while zeroing approximately the target fraction of entries
- Standard deviations over 50 independent runs should be reported alongside means; results must be averaged over both scRNA-seq and scATAC-seq samples

## Limitations

- Dropout is simulated with a simple Bernoulli-per-site mechanism, which may not capture real single-cell dropout intensity distributions
- Results depend on 50 independent runs with different initializations; single-run comparisons are unstable
- The in silico mixture (GM12878 vs K562, two subpopulations) is a simplified two-cluster setting; conclusions may not transfer to more heterogeneous populations

## Evidence

- [methods] simulate dropout by drawing from a Bernoulli distribution per site (gene or enhancer) and setting dropped entries in E and O to zero: [evidence span withheld — anchor status: not_found]
- [results] Incorporating loop data via DC3 reduced cluster-assignment classification error by more than four folds at dropout rates of 80% or higher: [evidence span withheld — anchor status: not_found]
- [intro] Based on comprehensive simulation experiments, we show that this method can deconvolve bulk profiles into subpopulation-specific profiles: "Based on comprehensive simulation experiments, we show that this method can deconvolve bulk profiles into subpopulation-specific profiles"
- [readme] python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt: "python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt"

---
name: simulation-parameter-sweep-analysis
description: 'Use when validating a perturbation-effect estimator on synthetic scCRISPR-seq
  benchmarks: construct two-cluster data with known cluster proportions and per-gRNA
  responses, sweep bias-generating parameters (d, r), run many seeded replicates,
  and score estimation loss.'
when_to_use_negative:
- Real scCRISPR-seq data without ground-truth responses — OC against known truth cannot
  be computed, so the sweep loses its validation power; use application-style runs
  (run_scDecouple) instead.
- Systems violating scDecouple's assumptions, e.g. tumor/disease tissues where the
  Gaussian mixture assumption may not hold, or when rare cell types are completely
  absent from infection.
- When the goal is perturbation-response estimation on a single dataset rather than
  characterizing estimator behavior as bias magnitude changes — no parameter sweep
  is needed.
tools:
- name: scDecouple
  role: EM-based maximum-likelihood estimator under evaluation; fits Gaussian mixture
    to decouple cellular response from infected proportion bias
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: Provides the Gaussian mixture EM algorithm used inside the simulations and
    estimation procedure
- name: tidyverse
  role: Used in R to construct the synthetic two-cluster benchmark and aggregate the
    100-replicate OC results
- name: diptest
  role: Dip-statistic multimodality testing underlying scDecouple's PC selection (top.multimodal
    threshold)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:c4b9986a8a86d37fb2d3ebff32fe5632a1f67a57b569959405bba29b1d309b0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - diptest
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# simulation-parameter-sweep-analysis

## Summary

Benchmark scDecouple's EM-based response estimator against a naive observed-change (fold-change) baseline on synthetic two-cluster scCRISPR-seq simulations, sweeping the infected-proportion-bias parameters (cluster distance d, ratio change r) and scoring overall correlation (OC) against ground-truth cellular responses on PC 1 and PC 2.

## When to use

Use when validating a perturbation-effect estimator on synthetic scCRISPR-seq benchmarks: construct two-cluster data with known cluster proportions and per-gRNA responses, sweep bias-generating parameters (d, r), run many seeded replicates, and score estimation loss.

## When NOT to use

- Real scCRISPR-seq data without ground-truth responses — OC against known truth cannot be computed, so the sweep loses its validation power; use application-style runs (run_scDecouple) instead.
- Systems violating scDecouple's assumptions, e.g. tumor/disease tissues where the Gaussian mixture assumption may not hold, or when rare cell types are completely absent from infection.
- When the goal is perturbation-response estimation on a single dataset rather than characterizing estimator behavior as bias magnitude changes — no parameter sweep is needed.

## Inputs

- Synthetic two-cluster control group Z (1000 NT cells, cell-by-gene matrix) with known cluster proportions
- Synthetic perturbation group Y (cell-by-gene matrix) with known per-gRNA ground-truth cellular responses
- Parameter grid: cluster distance d (with r fixed at 0.1) and ratio change r (with d fixed at 3)
- Ground-truth response vectors for computing overall correlation (OC)

## Outputs

- simulation_results.csv — aggregated OC scores for scDecouple and naive OC per grid setting and PC
- loss_curves.png — mean OC versus d and versus r, plotted per PC for both estimators

## How to apply

Construct a synthetic two-cluster benchmark in R with tidyverse: 1000 NT control cells whose expression lives in a 2D PC space embedding known cluster proportions (the source of infected proportion bias) and known per-gRNA cellular responses. Run a two-branch grid sweep — fix response strength r=0.1 and vary cluster distance d, then fix d=3 and vary r — running 100 independent simulations per grid point with the Gaussian mixture EM algorithm (mixtools) seeded with seed_=0 for reproducibility. At each grid point fit the decoupling estimator (scDecouple / run_scDecouple) and the naive observed-change baseline, and score each against ground truth via overall correlation (OC) computed on PC 1 and PC 2 separately. Aggregate OC across the 100 replicates per setting, write the results table to simulation_results.csv, and plot mean OC versus d and versus r as loss curves (loss_curves.png). Rationale: the EM maximum-likelihood formulation explicitly separates cluster proportions from cellular response, so the sweep tests whether that separation holds as multimodality and ratio change grow.

## Related tools

- **scDecouple** (EM-based maximum-likelihood estimator under evaluation; fits Gaussian mixture to decouple cellular response from infected proportion bias) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (Provides the Gaussian mixture EM algorithm used inside the simulations and estimation procedure)
- **tidyverse** (Used in R to construct the synthetic two-cluster benchmark and aggregate the 100-replicate OC results)
- **diptest** (Dip-statistic multimodality testing underlying scDecouple's PC selection (top.multimodal threshold))

## Examples

```
res <- run_scDecouple(Y, Z, seed_ = 0)
```

## Evaluation signals

- Qualitative behavior matches the reported result: naive OC loss increases monotonically as infected proportion bias grows (larger d or larger r), while scDecouple OC on PC 1 (the response-bearing PC) stays accurate across the whole sweep.
- On PC 2 scDecouple shows no advantage over naive OC — an advantage there would indicate the score is not isolating the response-bearing direction.
- Exactly 100 independent replicates per grid setting were run, and OC is reported as an aggregate (mean) over replicates, not single runs.
- Reproducibility: re-running with seed_=0 (mixtools EM seed) yields the same aggregated results written to simulation_results.csv.
- Grid structure is correct: one branch sweeps d with r fixed at 0.1, the other sweeps r with d fixed at 3.

## Limitations

- scDecouple's GMM assumption holds well in certain cell lines and tissues but 'might not remain in intricate systems or disease tissues, such as tumors', limiting transfer of the benchmark's conclusions.
- The assumption that NT and perturbation groups share the same cell clusters fails when rare cell types are completely absent from infection.
- The assumption of similar cellular responses across cell clusters restricts scDecouple to cells with moderate heterogeneity.
- Simulation setup details are only partially documented; the repo README provides usage but not the synthetic-generation code, so the benchmark must be reconstructed from the task specification.

## Evidence

- [abstract] We demonstrated its performance in a series of simulation experiments.: "We demonstrated its performance in a series of simulation experiments."
- [intro] Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation.: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation."
- [discussion] We validated the efficacy of scDecouple through a series of simulations: "We validated the efficacy of scDecouple through a series of simulations"
- [readme] infected proportion bias quantification, cellular response estimation and downstream analytical tools: "infected proportion bias quantification, cellular response estimation and downstream analytical tools"

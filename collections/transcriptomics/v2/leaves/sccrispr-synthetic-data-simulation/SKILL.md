---
name: sccrispr-synthetic-data-simulation
description: Use when validating a perturbation-effect estimator on scCRISPR-seq data
  where true cellular response and cluster proportions are known by construction,
  i.e. simulation benchmarks sweeping bias parameters such as cluster distance d and
  ratio change r.
when_to_use_negative:
- Real scCRISPR-seq data where ground-truth cellular responses and cluster proportions
  are unknown — use the scDecouple estimation pipeline instead of simulation-based
  validation
- Systems where the Gaussian-mixture model assumption is unlikely to hold, such as
  tumors or intricate disease tissues (the simulation assumes a two-cluster GMM structure)
- Situations where rare cell types are completely absent from infection, breaking
  the same-cell-clusters-between-groups assumption the simulation is built on
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: scDecouple
  role: EM-based decoupling estimator being benchmarked on the synthetic benchmark;
    run via run_scDecouple or the step-by-step scDecouple() function
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: Fits the Gaussian mixture model underpinning the EM algorithm used in the
    simulation/estimation
- name: tidyverse
  role: Data construction, aggregation of OC across replicates, and plotting of loss
    curves in R
- name: diptest
  role: Dip-statistic multimodality assessment used in PC selection, relevant to detecting
    the simulated data multimodality
- name: devtools
  role: Installs the scDecouple R package from GitHub for the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:5d236e3d3ccba2103f17f8c65110676c7270f1b1e1e44fc46b60263a3cdc5324
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - devtools
  - diptest
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# scCRISPR-synthetic-data-simulation

## Summary

Constructs two-cluster synthetic scCRISPR-seq benchmarks in R with known ground-truth cluster proportions and per-gRNA cellular responses, so that the accuracy of decoupling estimators like scDecouple can be scored against naive fold-change baselines as infected proportion bias varies.

## When to use

Use when validating a perturbation-effect estimator on scCRISPR-seq data where true cellular response and cluster proportions are known by construction, i.e. simulation benchmarks sweeping bias parameters such as cluster distance d and ratio change r.

## When NOT to use

- Real scCRISPR-seq data where ground-truth cellular responses and cluster proportions are unknown — use the scDecouple estimation pipeline instead of simulation-based validation
- Systems where the Gaussian-mixture model assumption is unlikely to hold, such as tumors or intricate disease tissues (the simulation assumes a two-cluster GMM structure)
- Situations where rare cell types are completely absent from infection, breaking the same-cell-clusters-between-groups assumption the simulation is built on

## Inputs

- Synthetic two-cluster simulation design parameters: cluster distance d (infected proportion bias magnitude) and ratio change r (cellular response strength)
- Ground-truth per-gRNA cellular responses and cluster proportions embedded in the simulation
- Number of control cells (e.g. 1000 NT cells) and number of replicates per grid point (e.g. 100)
- Random seed (e.g. seed_=0) for EM reproducibility

## Outputs

- Per-setting OC results table (e.g. simulation_results.csv) aggregated over replicates
- Loss-curve figure (e.g. loss_curves.png) plotting mean OC versus d and versus r for PC 1 and PC 2, estimator vs baseline

## How to apply

Simulate ~1000 NT control cells whose expression profiles live in a low-dimensional PC space with two clusters, embedding known cluster proportions (which induce infected proportion bias, controlled by cluster distance d) and known per-gRNA cellular responses (controlled by ratio change r). Fix r=0.1 and sweep d, then fix d=3 and sweep r, running 100 independent simulations per grid point; at each point fit the scDecouple EM estimator (Gaussian mixture model via mixtools, seeded with seed_=0 for reproducibility) and a naive observed fold-change (OC) baseline. Score each estimator against the known ground-truth responses via overall correlation (OC) computed on PC 1 and PC 2 separately, aggregate OC across replicates, and plot mean OC versus d and versus r as loss curves. The expected signature is that naive OC degrades as bias grows while the decoupled estimate stays accurate on the response-bearing PC 1.

## Related tools

- **scDecouple** (EM-based decoupling estimator being benchmarked on the synthetic benchmark; run via run_scDecouple or the step-by-step scDecouple() function) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (Fits the Gaussian mixture model underpinning the EM algorithm used in the simulation/estimation)
- **tidyverse** (Data construction, aggregation of OC across replicates, and plotting of loss curves in R)
- **diptest** (Dip-statistic multimodality assessment used in PC selection, relevant to detecting the simulated data multimodality)
- **devtools** (Installs the scDecouple R package from GitHub for the benchmark)

## Examples

```
res.step1 <- data_preprocessing(Y, Z); res.step2 <- pc_selection(Z.pca.mat=res.step1$Z.pca.results$x, Z.pca.sdev=res.step1$Z.pca.results$sdev, using_rank=TRUE); res.step3 <- scDecouple(Z.pca.mat=res.step1$Z.pca.results$x, Z.pca.rotation=res.step1$Z.pca.results$rotation, Y.norm.log.sub=res.step1$Y.norm.log.sub, select.pcs=res.step2)
```

## Evaluation signals

- Reproduced loss curves must show naive OC (fold-change baseline) increasing in error as infected proportion bias grows (larger d or larger r)
- scDecouple OC on PC 1 (the response-bearing PC) must remain accurate across all grid points of d and r
- scDecouple must show no advantage over the baseline on PC 2 (no response signal there)
- Results must be reproducible: 100 independent simulations per grid setting with seed_=0 give stable aggregated OC values
- Output table (simulation_results.csv) must contain OC for both estimators on PC 1 and PC 2 for every (d, r) grid point

## Limitations

- The Gaussian mixture model assumption may not hold in intricate systems or disease tissues such as tumors, so the two-cluster synthetic structure is an idealized benchmark
- Simulation assumes the same cell clusters exist between NT and perturbation groups; it cannot model extreme cases where some rare cell types are completely absent from infection
- Simulations assume similar cellular responses across cell clusters, limiting generalization to cells with moderate heterogeneity

## Evidence

- [abstract] We demonstrated its performance in a series of simulation experiments.: "We demonstrated its performance in a series of simulation experiments."
- [intro] scDecouple leverages a Gaussian mixture model to depict the distribution of gene: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation. scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [intro] naive fold change cannot reflect the true cellular response: "It then causes the traditional perturbation effect estimation by computing the fold change (FC) in the average expression level between the perturbation group and the control group cannot reflect the"
- [readme] run_scDecouple invocation from README: "res <- run_scDecouple(Y, Z)"
- [readme] seed parameter for EM reproducibility: "`seed_` (optional): The random seed for the EM algorithm. The default is 0."

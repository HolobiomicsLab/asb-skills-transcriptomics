---
name: gene-expression-phenotype-simulation
description: 'Use when you need controlled simulation benchmarks for TWAS training
  methods: you have a real genotype panel and want to vary causal variant proportion
  and heritability to compare out-of-sample test R2 and TWAS power across methods.'
when_to_use_negative:
- Your goal is real-data association discovery rather than benchmarking training methods
  — use actual GWAS/eQTL summary data instead of simulated phenotypes
- You only have summary statistics and no individual-level genotype panel on which
  to base the simulation
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: numpy
  role: Simulates gene expression phenotypes on the real ROS/MAP+MSBB WGS genotype
    panel under the six architecture scenarios
- name: scipy
  role: Converts two-sided p-values and beta coefficients to Z-scores via scipy.stats.norm
    (norm.ppf)
- name: pandas
  role: Tabulates out-of-sample test R2 and TWAS power comparisons across methods
    and scenarios
- name: OTTERS
  role: 'Downstream consumer: trains eQTL weights from simulated summary statistics
    (training.py) and runs Stage II gene-based tests (testing.py) with ACAT-O omnibus
    aggregation'
  repo: https://github.com/daiqile96/OTTERS
- name: FUSION
  role: 'Individual-level baseline: trains FUSION eQTL weights on the same reference
    data for comparison of R2 and power'
- name: PLINK
  role: Genotype manipulation for building the simulation and summary-statistic inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:92896941e3cee316ff8f5b06b4e5a9daab8d1527286e532d2cdd709f3d01f2da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - FUSION
  - OTTERS
  - PLINK
  - PRS-CS
  - SDPR
  - lassosum
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# gene-expression-phenotype-simulation

## Summary

Simulate gene expression phenotypes on a real WGS genotype panel (ROS/MAP+MSBB) under a factorial grid of eQTL architectures, generating matched eQTL and GWAS summary statistics to benchmark GReX imputation accuracy and TWAS power.

## When to use

Use when you need controlled simulation benchmarks for TWAS training methods: you have a real genotype panel and want to vary causal variant proportion and heritability to compare out-of-sample test R2 and TWAS power across methods.

## When NOT to use

- Your goal is real-data association discovery rather than benchmarking training methods — use actual GWAS/eQTL summary data instead of simulated phenotypes
- You only have summary statistics and no individual-level genotype panel on which to base the simulation

## Inputs

- Real WGS genotype panel (ROS/MAP+MSBB) as individual-level genotype matrix
- Simulation parameter grid: p_causal in {0.001, 0.01}, h_e2 in {0.01, 0.05, 0.1}
- Simulated expression phenotypes used to derive matched eQTL and GWAS summary statistics (p-values and beta coefficients)

## Outputs

- Simulated gene expression phenotypes per scenario
- Matched eQTL summary statistics (p-value, beta) per gene/scenario
- Matched GWAS summary statistics (p-value, beta) per gene/scenario
- Z-score formatted summary statistics (Z = sign(beta) * abs(norm.ppf(pvalue/2.0)))
- Tabulated out-of-sample test R2 and TWAS power per training method per scenario

## How to apply

Simulate gene expression phenotypes with numpy directly on the real ROS/MAP+MSBB WGS genotype panel, sweeping a six-scenario grid defined by proportion of causal variants p_causal in {0.001, 0.01} and heritability h_e2 in {0.01, 0.05, 0.1}. From each simulation, generate matched eQTL and GWAS summary statistics (two-sided p-values and beta coefficients). Convert summary statistics to Z-scores using scipy.stats.norm via Z = sign(beta) * abs(norm.ppf(pvalue/2.0)) so downstream tools receive a consistent scale. Use the summary-level reference to train eQTL weights (OTTERS Stage I with PT, lassosum, SDPR, PRScs) and the individual-level reference to train FUSION weights as the baseline. Evaluate each method by out-of-sample test R2 and TWAS power per scenario; a correct run shows OTTERS' omnibus ACAT-O test achieving the highest power in all six scenarios.

## Related tools

- **numpy** (Simulates gene expression phenotypes on the real ROS/MAP+MSBB WGS genotype panel under the six architecture scenarios)
- **scipy** (Converts two-sided p-values and beta coefficients to Z-scores via scipy.stats.norm (norm.ppf))
- **pandas** (Tabulates out-of-sample test R2 and TWAS power comparisons across methods and scenarios)
- **OTTERS** (Downstream consumer: trains eQTL weights from simulated summary statistics (training.py) and runs Stage II gene-based tests (testing.py) with ACAT-O omnibus aggregation) — https://github.com/daiqile96/OTTERS
- **FUSION** (Individual-level baseline: trains FUSION eQTL weights on the same reference data for comparison of R2 and power)
- **PLINK** (Genotype manipulation for building the simulation and summary-statistic inputs)

## Examples

```
from scipy.stats import norm; import numpy as np; Z = np.sign(beta) * np.abs(norm.ppf(pvalue/2.0))
```

## Evaluation signals

- All six scenarios are generated: the full factorial of p_causal {0.001, 0.01} x h_e2 {0.01, 0.05, 0.1}
- Z-score round-trip check: sign(beta) * abs(norm.ppf(pvalue/2.0)) reproduces the expected Z scale for every SNP summary row
- eQTL and GWAS summary statistics for each scenario derive from the same simulated expression phenotypes (matched sample structure)
- Benchmark table shows OTTERS omnibus (ACAT-O) power >= every single-method and FUSION power in all six scenarios
- SDPR/PRS-CS outperform FUSION in out-of-sample test R2 within dense-architecture scenarios

## Limitations

- Simulations anchored to ROS/MAP+MSBB genotype panels and summary statistics are mainly derived from individuals of European descent, limiting transferability conclusions
- Benchmark conclusions apply to cis-eQTL-based architectures; OTTERS only considers cis-eQTL effects, so trans-eQTL scenarios cannot be simulated/evaluated with this skill
- Training all five models per gene costs ~20 min, so scaling the simulation to ~20K genome-wide genes across multiple tissues is computationally expensive

## Evidence

- [methods] Simulate gene expression phenotypes with numpy on the real ROS/MAP+MSBB WGS genotype panel under six scenarios: [evidence span withheld — anchor status: not_found]
- [methods] Z-score conversion of simulated summary statistics: [evidence span withheld — anchor status: not_found]
- [intro] Simulation-based claim of superiority over FUSION: "we observe that OTTERS outperforms the popular FUSION20 approach across all simulation settings considered"
- [abstract] Simulation is the evaluation basis of OTTERS: "We show that OTTERS is a practical and powerful TWAS tool by both simulations and application studies."

---
name: twas-power-benchmark-comparison
description: Use when comparing TWAS training frameworks' power or GReX imputation
  accuracy, e.g., testing whether summary-level reference weights outperform individual-level
  baselines like FUSION.
when_to_use_negative:
- Input is real (not simulated) GWAS/eQTL summary data — the benchmarking logic requires
  known ground-truth causal architecture to estimate power
- No individual-level reference data are available to train the FUSION baseline for
  comparison
- The goal is cross-population or trans-eQTL TWAS, which OTTERS explicitly does not
  support
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: OTTERS
  role: 'Runs both benchmark stages: training.py for eQTL weight training from summary-level
    reference and testing.py for TWAS with ACAT-O omnibus aggregation'
  repo: https://github.com/daiqile96/OTTERS
- name: FUSION
  role: Individual-level baseline for eQTL weight training and TWAS comparison
- name: lassosum
  role: Frequentist LASSO-based PRS training method adapted to summary-level eQTL
    data, with pseudo-validation and EUR.hg38 LD blocks
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Bayesian Dirichlet Process Regression-based PRS training method from summary
    statistics
  repo: https://github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous shrinkage MCMC-based PRS training method from summary
    statistics
  repo: https://github.com/getian107/PRScs
- name: PLINK
  role: LD clumping (--r2=0.99) and per-gene genotype extraction
- name: scipy
  role: Converts simulated two-sided p-values and betas to Z-scores via scipy.stats.norm
- name: numpy
  role: Simulates gene expression phenotypes on the genotype panel
- name: pandas
  role: Tabulates test R2 and power comparisons across scenarios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:6069ebe36b8cf6cdb73af118f34ae15ec11ab30126602b925793defad406e578
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# twas-power-benchmark-comparison

## Summary

Benchmark GReX imputation accuracy (out-of-sample test R2) and TWAS power of PRS-based training methods (P+T, lassosum, SDPR, PRS-CS) and the OTTERS omnibus test against FUSION, using simulated eQTL/GWAS summary statistics under multiple genetic architectures.

## When to use

Use when comparing TWAS training frameworks' power or GReX imputation accuracy, e.g., testing whether summary-level reference weights outperform individual-level baselines like FUSION.

## When NOT to use

- Input is real (not simulated) GWAS/eQTL summary data — the benchmarking logic requires known ground-truth causal architecture to estimate power
- No individual-level reference data are available to train the FUSION baseline for comparison
- The goal is cross-population or trans-eQTL TWAS, which OTTERS explicitly does not support

## Inputs

- Real WGS genotype reference panel (e.g., ROS/MAP+MSBB) for simulation
- Simulated eQTL summary statistics (p-values and beta coefficients)
- Simulated GWAS summary statistics
- Individual-level reference genotype/expression data for training FUSION baseline
- LD reference panel (e.g., Berisa & Pickrell EUR.hg38 blocks for lassosum)

## Outputs

- Out-of-sample test R2 per training method per simulation scenario
- Empirical TWAS power per training method and for the OTTERS ACAT-O omnibus test per scenario
- Comparison tables of PRS-based methods vs FUSION across all six scenarios

## How to apply

Simulate gene expression on real genotype panels (e.g., ROS/MAP+MSBB WGS) across a grid of scenarios defined by proportion of causal SNPs (p_causal in {0.001, 0.01}) and heritability (h_e2 in {0.01, 0.05, 0.1}), generating matched eQTL and GWAS summary statistics. Convert two-sided p-values to Z-scores via Z = sign(beta) * abs(norm.ppf(pvalue/2.0)) with scipy.stats.norm. Train eQTL weights with OTTERS Stage I (training.py --models=PT,lassosum,SDPR,PRScs, LD-clumping --r2=0.99, --lassosum_ld_blocks=EUR.hg38) and train FUSION weights on the same individual-level reference as the baseline. Run OTTERS Stage II (testing.py --models=P0.001,P0.05,lassosum,SDPR,PRScs) to obtain method-specific TWAS p values, aggregated via ACAT-O for the omnibus test. Tabulate out-of-sample test R2 and empirical TWAS power per method per scenario; the rationale is that P+T favors sparse architecture while Bayesian methods (SDPR, PRS-CS) favor denser architecture, so the omnibus should be near-optimal across scenarios.

## Related tools

- **OTTERS** (Runs both benchmark stages: training.py for eQTL weight training from summary-level reference and testing.py for TWAS with ACAT-O omnibus aggregation) — https://github.com/daiqile96/OTTERS
- **FUSION** (Individual-level baseline for eQTL weight training and TWAS comparison)
- **lassosum** (Frequentist LASSO-based PRS training method adapted to summary-level eQTL data, with pseudo-validation and EUR.hg38 LD blocks) — https://github.com/tshmak/lassosum
- **SDPR** (Bayesian Dirichlet Process Regression-based PRS training method from summary statistics) — https://github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous shrinkage MCMC-based PRS training method from summary statistics) — https://github.com/getian107/PRScs
- **PLINK** (LD clumping (--r2=0.99) and per-gene genotype extraction)
- **scipy** (Converts simulated two-sided p-values and betas to Z-scores via scipy.stats.norm)
- **numpy** (Simulates gene expression phenotypes on the genotype panel)
- **pandas** (Tabulates test R2 and power comparisons across scenarios)

## Examples

```
python training.py --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- OTTERS omnibus achieves the highest TWAS power in all simulated scenarios (six combinations of p_causal and h_e2)
- SDPR and PRS-CS outperform FUSION in out-of-sample test R2
- Out-of-sample R2 is computed on held-out samples, not in-sample fit of the training reference
- Z-score conversion round-trip: Z = sign(beta) * abs(norm.ppf(pvalue/2.0)) yields two-sided p-values matching the input
- Omnibus ACAT-O p value is near-optimal relative to the best single method in each scenario

## Limitations

- Training GReX imputation models with all five methods costs ~20 min per gene, which is computationally challenging for ~20K genes and multiple tissues
- Available eQTL summary statistics are mainly from European-descent individuals, limiting cross-population transferability
- OTTERS only considers cis-eQTL effects and cannot provide the direction of identified gene-phenotype associations (users must refer to per-method TWAS Z-score signs)

## Evidence

- [other] Simulation grid definition: [evidence span withheld — anchor status: not_found]
- [discussion] Omnibus near-optimality: "this test had near-optimal performance across all simulation scenarios considered"
- [discussion] Method architecture tradeoff: "P+T methods generally performing better for sparse architecture, whereas the Bayesian methods performing better for denser architecture"
- [other] Z-score conversion: [evidence span withheld — anchor status: not_found]
- [intro] OTTERS benchmark claim: "we observe that OTTERS outperforms the popular FUSION20 approach across all simulation settings considered"
- [discussion] Training cost caveat: "training GReX imputation models by all individual methods on average cost ~20 min for all five training models per gene"

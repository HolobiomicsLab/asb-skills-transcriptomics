---
name: twas-simulation-execution
description: Use when the task requires simulating a complex trait from real 1000G
  Phase 3 PLINK genotypes and producing GWAS/TWAS statistics for evaluating TWAS method
  performance or power at settings like --ngwas 100000 and --nqtl 500.
when_to_use_negative:
- The goal is fine-mapping of real GWAS summary statistics into causal gene sets rather
  than simulation — use FOCUS/MA-FOCUS instead
- The task only needs weight construction or fine-mapping on real molecular data (e.g.,
  SuSiE/susieR or SuShiE workflows) without a simulated trait
- GWAS sample size is extremely large and wall-clock/memory budget is tight while
  fast-mode approximations are unacceptable — standard individual-level GWAS simulation
  will be prohibitively slow (41x more CPU time at N=500K)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: twas_sim
  role: The simulator framework installed via conda environment.yml; hosts sim.py
    which samples genomic regions, simulates eQTL/GWAS data, and computes TWAS statistics
  repo: https://github.com/mancusolab/twas_sim
- name: sim.py
  role: The actual sim.py simulator invoked (as in example.sh) to generate a single
    TWAS statistic with flags like --ngwas, --nqtl, --linear-model, --fast-gwas-sim
  repo: https://github.com/mancusolab/twas_sim
- name: PLINK
  role: Provides the PLINK-formatted genotype data format (prefix inputs) for reference
    eQTL/GWAS LD information consumed by sim.py
- name: susieR
  role: 'Example external prediction tool: external_r.py calls external.R to call
    susieR for a sparse fit, demonstrating the dynamic-import fit-function interface'
  repo: https://github.com/stephenslab/susieR
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad288
    title: twas_sim, a Python-based tool for simulation and power analysis of transcriptome-wide
      association analysis
  source_content_hash: sha256:e7ed0aa7a5f57ac7d089678ac6786c6c537f418caa262194690ea25476143487
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mancusolab/twas_sim
  tools:
  - PLINK
  - R
  - conda
  - sim.py
  - susieR
  - twas_sim
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# twas-simulation-execution

## Summary

Execute end-to-end TWAS simulations with twas_sim's sim.py: sample a 1000G genomic region, filter genotypes (HapMap3, MAF>=1%, HWE>=1e-5), simulate eQTL/GWAS data, fit prediction weights, and compute TWAS statistics and power.

## When to use

Use when the task requires simulating a complex trait from real 1000G Phase 3 PLINK genotypes and producing GWAS/TWAS statistics for evaluating TWAS method performance or power at settings like --ngwas 100000 and --nqtl 500.

## When NOT to use

- The goal is fine-mapping of real GWAS summary statistics into causal gene sets rather than simulation — use FOCUS/MA-FOCUS instead
- The task only needs weight construction or fine-mapping on real molecular data (e.g., SuSiE/susieR or SuShiE workflows) without a simulated trait
- GWAS sample size is extremely large and wall-clock/memory budget is tight while fast-mode approximations are unacceptable — standard individual-level GWAS simulation will be prohibitively slow (41x more CPU time at N=500K)

## Inputs

- 1000G Phase 3 PLINK-formatted reference genotype files (recommended; HapMap3-restricted)
- Optional PLINK-formatted eQTL LD reference prefix (--eqtl-prefix) and test LD prefix (--test-prefix)
- External Python module defining a fit function for eQTL weight estimation (--external-module)
- Simulation parameters: --ngwas, --nqtl, --ncausal, --eqtl-h2, --h2ge, --ld-ridge, --seed, --IDX

## Outputs

- OUTPUT.summary.tsv (per-run summary incl. linear_model field, twas.z, twas.p; real.time and cpu.time fields)
- OUTPUT.scan.tsv (TWAS scan results)
- Optional gzip-compressed outputs via -c/--compress

## How to apply

Clone mancusolab/twas_sim and activate its conda environment (conda env create --file environment.yml; conda activate twas_sim), then drive the simulator sim.py on 1000G Phase 3 PLINK reference files. sim.py first samples a genomic region uniformly at random, subsets reference genotypes to that region while removing non-biallelic variants, MAF < 1%, HWE < 1e-5, and missingness > 10%, and restricts to HapMap3 variants. Choose the GWAS mode deliberately: standard mode simulates summary statistics from individual-level genotype/phenotype data, whereas --fast-gwas-sim draws GWAS summary statistics directly from a multivariate normal parameterized by LD — the fast path is preferable at large --ngwas (it needed 6x/36x less memory and 8x/41x less CPU time at GWAS N = 100K/500K). Set effect parameters such as --eqtl-h2 0.1 (default), --h2ge (0.001 to 0.01 are reasonable), --ncausal (counts, '1pct'-style percentages, or '1avg' Poisson averages), and the LD diagonal offset --ld-ridge 0.1. Select a prediction model with --linear-model (lasso, enet, ridge, trueqtl, or external); external tools are integrated via dynamic import by writing a Python module exposing a fit function and passing it with --external-module. Collect outputs and judge completion by the presence of OUTPUT.summary.tsv and OUTPUT.scan.tsv with the linear_model recorded and finite twas.z and twas.p values.

## Related tools

- **twas_sim** (The simulator framework installed via conda environment.yml; hosts sim.py which samples genomic regions, simulates eQTL/GWAS data, and computes TWAS statistics) — https://github.com/mancusolab/twas_sim
- **sim.py** (The actual sim.py simulator invoked (as in example.sh) to generate a single TWAS statistic with flags like --ngwas, --nqtl, --linear-model, --fast-gwas-sim) — https://github.com/mancusolab/twas_sim
- **PLINK** (Provides the PLINK-formatted genotype data format (prefix inputs) for reference eQTL/GWAS LD information consumed by sim.py)
- **susieR** (Example external prediction tool: external_r.py calls external.R to call susieR for a sparse fit, demonstrating the dynamic-import fit-function interface) — https://github.com/stephenslab/susieR

## Examples

```
python sim.py 1000G_ref --ngwas 100000 --nqtl 500 --eqtl-h2 0.1 --h2ge 0.001 --fast-gwas-sim --linear-model external --external-module my_module --seed IDX -o OUTPUT
```

## Evaluation signals

- OUTPUT.summary.tsv exists for the run and its linear_model field records the model actually requested (e.g., external, enet, lasso)
- OUTPUT.summary.tsv contains finite twas.z and twas.p values, confirming the TWAS statistic was computed end-to-end
- Benchmark CSV from scalability runs contains exactly four records (2 modes x 2 sample sizes: standard/fast at N=100K and 500K) with real.time and cpu.time fields present and fast/standard speedup ratios > 1 (paper reports 6x/36x memory and 8x/41x CPU-time savings, consistent with Fig 1C and Supplementary Fig S7)
- Simulated genotype subset respects the filter steps: only biallelic, MAF >= 1%, HWE >= 1e-5, missingness <= 10%, HapMap3 variants remain
- Dynamic import succeeds: a custom module exposing fit (e.g., my_module.py) is loaded by --linear-model external and the run completes emitting both summary and scan TSVs

## Limitations

- Standard (non-fast) GWAS simulation uses individual-level genotype and phenotype data and scales poorly: at GWAS N=100K/500K it required 6x/36x more memory and 8x/41x more CPU time than --fast-gwas-sim
- The LD matrix diagonal offset (--ld-ridge, default 0.1) must be set when LD matrices are near-singular; wrong choices can destabilize simulation
- External prediction models must match the expected interface exactly — a single Python fit function (or an R wrapper like external_r.py/external.R) — or dynamic import fails
- Out of the box, linear models cover Elastic Net, LASSO, GBLUP, and true eQTL effect sizes only; anything else requires the external module mechanism

## Evidence

- [readme] A python software leveraging real genotype data to simulate a complex trait as a function of latent expression, fit eQTL weights in independent data, and perform GWAS/TWAS on the complex trait.: "A python software leveraging real genotype data to simulate a complex trait as a function of latent expression, fit eQTL weights in independent data, and perform GWAS/TWAS on the complex trait."
- [methods] removing genetic variants that are non-biallelic, MAF < 1%, HWE < 1e-5, and variant missingness > 10%: "removing genetic variants that are non-biallelic, MAF < 1%, HWE < 1e-5, and variant missingness > 10%"
- [methods] standard GWAS simulates GWAS summary statistics using individual-level genotype and phenotype data. Fast GWAS simulates GWAS summary statistics directly using the multivariate normal: "standard GWAS simulates GWAS summary statistics using individual-level genotype and phenotype data. Fast GWAS simulates GWAS summary statistics directly using the multivariate normal"
- [methods] twas_sim computes TWAS test statistics using LD, GWAS Z-scores, and estimated eQTL effect sizes.: "twas_sim computes TWAS test statistics using LD, GWAS Z-scores, and estimated eQTL effect sizes."
- [readme] We recommend to use data from [1000G](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz) as reference genotypes: "We recommend to use data from [1000G](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz) as reference genotypes"
- [methods] git clone https://github.com/mancusolab/twas_sim.git: "git clone https://github.com/mancusolab/twas_sim.git
    cd twas_sim

then,

    conda env create --file environment.yml
    conda activate twas_sim"

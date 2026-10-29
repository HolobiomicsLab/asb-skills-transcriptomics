---
name: twas-power-estimation-across-ld-panels
description: Use when evaluating how TWAS power or type-I error changes when eQTL
  weight-training LD and TWAS testing LD come from a different reference panel than
  the GWAS LD, or when planning TWAS power analyses.
when_to_use_negative:
- The goal is gene fine-mapping of real TWAS summary statistics rather than simulation-based
  power/calibration analysis — use FOCUS/MA-FOCUS instead.
- Individual-level GWAS genotype and phenotype data are available and standard GWAS
  simulation is desired rather than the fast multivariate-normal shortcut.
- GWAS, eQTL, and TWAS testing necessarily share a single LD panel and the question
  is not sensitivity to LD reference choice.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: twas_sim (sim.py)
  role: Simulates GWAS summary statistics, fits eQTL weights (enet), and computes
    TWAS test statistics under configurable LD reference panels and h2ge values
  repo: https://github.com/mancusolab/twas_sim
- name: PLINK
  role: Provides the PLINK-formatted 1000G reference genotype files used as GWAS/eQTL/testing
    LD panels
  repo: https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad288
    title: twas_sim, a Python-based tool for simulation and power analysis of transcriptome-wide
      association analysis
  source_content_hash: sha256:a66fb20994c713c5fbc44192440b9091e399c891bb0808de180db8e6b9b50c16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mancusolab/twas_sim
  tools:
  - PLINK
  - conda
  - sim.py
  - twas_sim
  - twas_sim (sim.py)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# twas-power-estimation-across-ld-panels

## Summary

Estimate TWAS power and calibration under matched vs. mismatched LD reference panels by simulating GWAS summary statistics and TWAS test statistics with twas_sim's sim.py, sweeping h2ge over null and alternative values across many seeded replicates.

## When to use

Use when evaluating how TWAS power or type-I error changes when eQTL weight-training LD and TWAS testing LD come from a different reference panel than the GWAS LD, or when planning TWAS power analyses.

## When NOT to use

- The goal is gene fine-mapping of real TWAS summary statistics rather than simulation-based power/calibration analysis — use FOCUS/MA-FOCUS instead.
- Individual-level GWAS genotype and phenotype data are available and standard GWAS simulation is desired rather than the fast multivariate-normal shortcut.
- GWAS, eQTL, and TWAS testing necessarily share a single LD panel and the question is not sensitivity to LD reference choice.

## Inputs

- 1000 Genomes Phase 3 PLINK-formatted EUR reference genotypes (503 individuals), split into two disjoint panels (244 and 245)
- simulator parameters: GWAS sample size (--ngwas 100000), eQTL sample size (--nqtl 500), number of causal SNPs (--ncausal 1), eQTL SNP heritability (--eqtl-h2 0.1), phenotypic variance explained by gene expression (--h2ge from 0 to 0.01)
- linear prediction model choice (--linear-model enet, or lasso/ridge/trueqtl)

## Outputs

- Per-replicate *.summary.tsv files containing the simulated TWAS statistic and twas.p
- Empirical type-I error / null p-value calibration estimates for each LD configuration
- TWAS power estimates (fraction of replicates with twas.p below threshold) for h2ge > 0, per LD configuration
- Tab-delimited comparison table of matched vs. mismatched LD calibration and power (Supplementary Table S3 / Figure S6 analogue)

## How to apply

Split the 1000G Phase 3 PLINK-formatted EUR reference (503 individuals) into two disjoint panels (e.g., 244 and 245) so GWAS LD, eQTL LD, and TWAS testing LD can be decoupled. Run sim.py with --fast-gwas-sim so GWAS summary statistics are drawn directly from the multivariate normal parameterized by the GWAS-panel LD, passing --eqtl-prefix and --test-prefix to point at the mismatched panel for eQTL weight fitting (--linear-model enet, --nqtl 500, --eqtl-h2 0.1) and TWAS testing. Sweep --h2ge from 0 (null) to 0.001-0.01 (alternative) across many replicates, each with a distinct --seed, writing *.summary.tsv files. Under the null collect twas.p across replicates to compute empirical type-I error/calibration; under alternatives compute power as the fraction of replicates with twas.p below the significance threshold. Repeat omitting --eqtl-prefix/--test-prefix so GWAS, eQTL, and testing share one panel, giving the correctly-specified baseline; compare calibration and power between matched and mismatched LD configurations in a tab-delimited table (Supplementary Table S3 / Figure S6 style).

## Related tools

- **twas_sim (sim.py)** (Simulates GWAS summary statistics, fits eQTL weights (enet), and computes TWAS test statistics under configurable LD reference panels and h2ge values) — https://github.com/mancusolab/twas_sim
- **PLINK** (Provides the PLINK-formatted 1000G reference genotype files used as GWAS/eQTL/testing LD panels) — https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz

## Examples

```
python sim.py <prefix> --eqtl-prefix <panel2> --test-prefix <panel2> --ngwas 100000 --nqtl 500 --ncausal 1 --eqtl-h2 0.1 --fast-gwas-sim --linear-model enet --h2ge 0.005 --seed 1 --output <prefix>
```

## Evaluation signals

- Under the null (h2ge=0), empirical type-I error from the collected twas.p distribution matches the nominal TWAS significance level for both matched and mismatched panels.
- Mismatched-LD power is significantly lower than matched-LD power under the causal model (α ≠ 0); the reference result shows power reduction P = 2.2E-16, with inflation P = 0.049 and FWER P = 0.005.
- Under the null, same-panel and misspecified-panel statistics remain largely consistent (null KS P = 0.57).
- Each *.summary.tsv contains a valid twas.p value for every seeded replicate, and the h2ge sweep spans both 0 and alternative values (0.001-0.01).
- The aggregated table mirrors Supplementary Table S3 and the power-reduction trend of Figure S6.

## Limitations

- LD misspecification using mismatched reference panels significantly reduces TWAS power under a causal model (α ≠ 0), so power estimates from a single mismatched panel are pessimistic relative to matched LD.
- Fast GWAS simulation assumes summary statistics are drawn from the multivariate normal parameterized by the GWAS-panel LD; it does not simulate individual-level phenotypes.
- Results depend on simulation settings (eQTL sample size nqtl=500, eqtl-h2=0.1, ncausal=1); power estimates are not directly transferable to other weight-training or locus configurations.

## Evidence

- [abstract] we present twas_sim, a computationally scalable and easily extendable tool for simplified performance evaluation and power analysis for TWAS methods: "we present twas_sim, a computationally scalable and easily extendable tool for simplified performance evaluation and power analysis for TWAS methods"
- [readme] LD panel flexibility: "twas_sim supports the option to use different LD reference panels across GWAS and eQTL simulations in addition to TWAS testing."
- [readme] fast GWAS simulation: "Fast GWAS simulates GWAS summary statistics directly using the multivariate normal distribution parameterized by LD."
- [methods] TWAS statistic inputs: "twas_sim computes TWAS test statistics using LD, GWAS Z-scores, and estimated eQTL effect sizes."
- [methods] reference genotypes: "Optional prefix to PLINK-formatted data for eQTL LD information. Otherwise use GWAS LD. (default: None)"
- [methods] variant filtering: "removing genetic variants that are non-biallelic, MAF < 1%, HWE < 1e-5, and variant missingness > 10%"

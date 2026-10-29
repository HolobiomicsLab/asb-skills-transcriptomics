---
name: sparse-and-bayesian-polygenic-model-training
description: Use when only summary-level eQTL statistics plus an LD reference panel
  are available (no individual-level expression genotypes) and per-gene GReX imputation
  weights are needed for downstream TWAS Stage II testing.
when_to_use_negative:
- Individual-level reference expression and genotype data are available — FUSION/PrediXcan-style
  individual-level training is the conventional alternative.
- The goal is trans-eQTL weight estimation — OTTERS currently considers only cis-eQTL
  effects.
- Reference eQTL summary data are from a non-European population — available summary
  statistics are mainly from individuals of European descent, limiting cross-population
  transferability.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: 'Integrated Stage I driver (training.py): performs LD-clumping, prepares P+T/lassosum/SDPR/PRS-CS
    inputs, and runs all four weight-training methods per gene'
  repo: github.com/daiqile96/OTTERS
- name: lassosum
  role: Frequentist LASSO/Elastic Net regression on summary statistics accounting
    for LD via reference panel; pseudo-validation via fdrtool
  repo: github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics-based Dirichlet Process Regression Bayesian weight training;
    requires MKL and GSL dynamic libraries on LD_LIBRARY_PATH
  repo: github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage (CS) prior posterior SNP effect-size inference
    used to train eQTL weights
  repo: github.com/getian107/PRScs
- name: PLINK
  role: PLINK 1.9 binary reference panel format and per-gene variant extraction (call_PLINK_extract)
- name: TABIX
  role: Indexed extraction of eQTL summary statistics per target gene region (extract_vcf_region_with_tabix)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:7ce390d62e503859f2847c265313837d2b419c08bb7507256b60ef415a0ddde3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OTTERS
  - PLINK
  - PRS-CS
  - SDPR
  - TABIX
  - fdrtool
  - lassosum
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Sparse and Bayesian polygenic-model training

## Summary

OTTERS Stage I: train cis-eQTL weights per gene from summary-level eQTL statistics and a PLINK-binary LD reference panel by adapting four PRS methods — P+T with LD clumping (P0.001, P0.05), frequentist lassosum (fdrtool pseudo-validation), Bayesian SDPR (Dirichlet Process Regression), and Bayesian continuous-shrinkage PRS-CS — because eQTL effect sizes are effectively SNP effect sizes from a reference 'GWAS' of gene expression.

## When to use

Use when only summary-level eQTL statistics plus an LD reference panel are available (no individual-level expression genotypes) and per-gene GReX imputation weights are needed for downstream TWAS Stage II testing.

## When NOT to use

- Individual-level reference expression and genotype data are available — FUSION/PrediXcan-style individual-level training is the conventional alternative.
- The goal is trans-eQTL weight estimation — OTTERS currently considers only cis-eQTL effects.
- Reference eQTL summary data are from a non-European population — available summary statistics are mainly from individuals of European descent, limiting cross-population transferability.

## Inputs

- eQTL summary statistics file, pre-sorted by chromosome then ascending SNP position
- Gene annotation file (per-gene cis-window boundaries)
- PLINK-binary (bed/bim/fam) LD reference panel
- lassosum LD block definitions (EUR.hg19/hg38, ASN, AFR from Berisa & Pickrell 2015)
- Thread count / parallelism configuration

## Outputs

- Trained eQTL weight files per gene for all five model variants (P0.001, P0.05, lassosum, SDPR, PRScs) in the Results output directory
- Genetically regulated expression (GReX) imputations via optional imputing.py

## How to apply

Pre-sort the eQTL summary statistics by chromosome then ascending SNP position, and provide a gene annotation file plus a PLINK-binary LD reference panel. Run OTTERS Stage I via training.py (e.g. on chromosome 4) with LD-clumping threshold r2=0.99 and models=PT,lassosum,SDPR,PRScs; the tool integrates TABIX and PLINK 1.9 to extract per-gene input data, sets lassosum LD blocks via --lassosum_ld_blocks (default EUR.hg38), and writes trained weights per method to the Results directory. Prepare the runtime before execution: prepend SDPR's MKL and GSL directories to LD_LIBRARY_PATH (export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:gsl/lib/:MKL/lib/), pin MKL_NUM_THREADS/NUMEXPR_NUM_THREADS/OMP_NUM_THREADS to the requested thread count, and load R with lassosum plus fdrtool for pseudo-validation. The four PRS methods span sparse (P+T, lassosum) and dense Bayesian (SDPR, PRS-CS) genetic architectures; parallel computation is enabled to train multiple genes simultaneously.

## Related tools

- **OTTERS** (Integrated Stage I driver (training.py): performs LD-clumping, prepares P+T/lassosum/SDPR/PRS-CS inputs, and runs all four weight-training methods per gene) — github.com/daiqile96/OTTERS
- **lassosum** (Frequentist LASSO/Elastic Net regression on summary statistics accounting for LD via reference panel; pseudo-validation via fdrtool) — github.com/tshmak/lassosum
- **SDPR** (Summary-statistics-based Dirichlet Process Regression Bayesian weight training; requires MKL and GSL dynamic libraries on LD_LIBRARY_PATH) — github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage (CS) prior posterior SNP effect-size inference used to train eQTL weights) — github.com/getian107/PRScs
- **PLINK** (PLINK 1.9 binary reference panel format and per-gene variant extraction (call_PLINK_extract))
- **TABIX** (Indexed extraction of eQTL summary statistics per target gene region (extract_vcf_region_with_tabix))

## Examples

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:gsl/lib/:MKL/lib/; python training.py --chr=4 --gene_annot genes.txt --ld_ref chr4 --eqtl_summary eqtl.sorted.txt --r2 0.99 --models PT,lassosum,SDPR,PRScs --lassosum_ld_blocks EUR.hg38
```

## Evaluation signals

- The Stage I Results directory contains trained eQTL weights for all five model variants (P0.001, P0.05, lassosum, SDPR, PRScs) for every annotated gene on the chromosome.
- Input eQTL summary statistics were confirmed sorted by chromosome then ascending SNP position before training.
- lassosum pseudo-validation completed using fdrtool with the requested --lassosum_ld_blocks setting (e.g. EUR.hg38).
- SDPR loaded without dynamic-library errors after MKL/GSL paths were added to LD_LIBRARY_PATH; per-gene training time is ~20 min for all five models, so large deviations suggest misconfiguration.
- Thread-count environment variables (MKL_NUM_THREADS, NUMEXPR_NUM_THREADS, OMP_NUM_THREADS) matched the requested parallelism and genes trained without thread oversubscription.

## Limitations

- Training GReX imputation models by all five methods costs on average ~20 min per gene, which is computationally challenging for multiple tissues and ~20K genome-wide genes.
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability.
- SDPR was only tested on linux x86 and explicitly uses SSE instructions; it fails if the redistributed gsl/lib and MKL/lib dynamic libraries are altered.

## Evidence

- [abstract] adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data: "adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data"
- [intro] PRS methods are adaptable because eQTL effect sizes are SNP effect sizes from a reference GWAS of expression: [evidence span withheld — anchor status: not_found]
- [readme] lassosum requires LD-aware reference panel and pseudo-validation via fdrtool: [evidence span withheld — anchor status: not_found]
- [readme] SDPR requires MKL/GSL dynamic libraries on LD_LIBRARY_PATH: [evidence span withheld — anchor status: external_doc]
- [methods] lassosum LD blocks selectable via --lassosum_ld_blocks with EUR.hg38 default: "Added `--lassosum_ld_blocks` argument"
- [discussion] Stage I training is computationally intensive (~20 min per gene for all five models): "training GReX imputation models by all individual methods on average cost ~20 min for all five training models per gene"

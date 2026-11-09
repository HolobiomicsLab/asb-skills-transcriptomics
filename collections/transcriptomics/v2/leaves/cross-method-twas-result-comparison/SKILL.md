---
name: cross-method-twas-result-comparison
description: Use when two TWAS frameworks have been run on identical reference (e.g.,
  GTEx V8 whole blood, n=574) and GWAS data and you must count independently significant
  genes per method and their overlap.
when_to_use_negative:
- The two methods were trained on different reference panels or applied to different
  GWAS summary statistics — the comparison no longer isolates method differences
- OTTERS summary-level eQTL reference is not available for the tissue, so only FUSION-style
  individual-level training is possible
- The goal is cross-population TWAS or trans-eQTL integration — OTTERS only considers
  cis-eQTL effects from mainly European-descent summary data
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: OTTERS
  role: Trains eQTL weights from summary-level reference data (P+T, lassosum, SDPR,
    PRS-CS) and performs Stage II gene-based association with ACAT-O omnibus combination
  repo: https://github.com/daiqile96/OTTERS
- name: FUSION
  role: Baseline TWAS method trained from individual-level genotypes and expression
    of the same reference samples for the head-to-head benchmark
- name: lassosum
  role: One of the four PRS-based training methods integrated into OTTERS Stage I
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics-based Dirichlet Process Regression training method integrated
    into OTTERS Stage I
  repo: https://github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage PRS training method integrated into OTTERS Stage
    I
  repo: https://github.com/getian107/PRScs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:7bd236bbfbc3016e6ed74afdbc5d2c40d5efb97a9767dc9f4190095321c05997
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - FUSION
  - OTTERS
  - PRS-CS
  - SDPR
  - lassosum
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-method-twas-result-comparison

## Summary

Head-to-head comparison of independently significant TWAS genes identified by two transcriptome-wide association methods (OTTERS vs FUSION) trained on the same reference and applied to the same GWAS, to quantify method-specific discovery gains and overlap.

## When to use

Use when two TWAS frameworks have been run on identical reference (e.g., GTEx V8 whole blood, n=574) and GWAS data and you must count independently significant genes per method and their overlap.

## When NOT to use

- The two methods were trained on different reference panels or applied to different GWAS summary statistics — the comparison no longer isolates method differences
- OTTERS summary-level eQTL reference is not available for the tissue, so only FUSION-style individual-level training is possible
- The goal is cross-population TWAS or trans-eQTL integration — OTTERS only considers cis-eQTL effects from mainly European-descent summary data

## Inputs

- OTTERS inputs: annotation file (CHROM, GeneStart, GeneEnd, TargetID); eQTL summary statistics sorted by chromosome then ascending SNP position (CHROM, POS, A1, A2, Zscore, TargetID, N); GWAS summary statistics (need not be sorted); LD reference panel genotype data
- FUSION inputs: individual-level genotypes and expression from the same GTEx V8 whole blood samples (n=574)
- Identical GWAS summary statistics for both methods (UKBB cardiovascular disease)
- Per-method TWAS p values and independently significant gene lists at the study threshold

## Outputs

- Count of independently significant TWAS genes per method (OTTERS 34, FUSION 21 in the benchmark)
- Count and identity of shared significant genes (14 shared)
- Comparison figure (Supplementary Fig. S11 equivalent) of the OTTERS vs FUSION gene sets

## How to apply

Run both TWAS pipelines against the identical GWAS summary statistics (e.g., UKBB cardiovascular GWAS) using the same reference data — OTTERS Stage I (training.py with --models=PT,lassosum,SDPR,PRScs, LD-clumping r2=0.99) then Stage II (testing.py with --models=P0.001,P0.05,lassosum,SDPR,PRScs) with ACAT-O omnibus combination, versus FUSION weights trained from the same individual-level GTEx V8 whole blood genotypes and expression. For each method, declare independently significant genes at the study's significance threshold and count the gene-set overlap (14 shared in the published benchmark). Tabulate per-method counts (OTTERS 34 vs FUSION 21) and render the comparison as a figure equivalent to Supplementary Fig. S11. Rationale: identical reference and GWAS inputs isolate the training-framework difference (summary-level PRS-based weights vs individual-level model training) as the sole cause of discovery-count differences.

## Related tools

- **OTTERS** (Trains eQTL weights from summary-level reference data (P+T, lassosum, SDPR, PRS-CS) and performs Stage II gene-based association with ACAT-O omnibus combination) — https://github.com/daiqile96/OTTERS
- **FUSION** (Baseline TWAS method trained from individual-level genotypes and expression of the same reference samples for the head-to-head benchmark)
- **lassosum** (One of the four PRS-based training methods integrated into OTTERS Stage I) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics-based Dirichlet Process Regression training method integrated into OTTERS Stage I) — https://github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage PRS training method integrated into OTTERS Stage I) — https://github.com/getian107/PRScs

## Examples

```
python training.py --models=PT,lassosum,SDPR,PRScs --r2=0.99 && python testing.py --models=P0.001,P0.05,lassosum,SDPR,PRScs
```

## Evaluation signals

- Reproduced counts match Supplementary Fig. S11: 34 independently significant genes for OTTERS, 21 for FUSION, 14 shared
- Both methods were applied to the identical GWAS summary statistics and matched reference samples (GTEx V8 whole blood, n=574)
- OTTERS omnibus p values are the ACAT-O combination of the per-method TWAS p values (P0.001, P0.05, lassosum, SDPR, PRScs)
- Independently significant gene sets are declared at a single pre-specified significance threshold applied identically to both methods

## Limitations

- Training GReX imputation models by all five methods costs ~20 min per gene, making multi-tissue genome-wide (~20K genes) comparisons computationally heavy
- OTTERS cannot provide the direction of identified gene-phenotype associations; users must consult the sign of the TWAS Z-score per training method
- OTTERS considers only cis-eQTL effects, and available summary-level eQTL references are mainly from European-descent individuals, limiting cross-population transferability

## Evidence

- [results] head-to-head counts: "when the same individual-level GTEx V8 reference data of whole blood samples were used"
- [methods] ACAT-O omnibus: [evidence span withheld — anchor status: not_found]
- [readme] OTTERS workflow: "perform TWAS Stage I: run P+T, lassosum, SDPR, and PRS-CS to train eQTL weights.
 -  perform TWAS Stage II: gene-based association test using GWAS summary statistics."
- [intro] comparison rationale: "OTTERS using large summary-level reference datasets and multiple gene expression imputation models can successfully reveal potential risk genes missed by FUSION"
- [discussion] direction limitation: "our OTTERS cannot provide the direction of the identified gene-phenotype associations"

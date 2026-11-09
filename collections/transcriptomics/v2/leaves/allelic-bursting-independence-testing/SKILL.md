---
name: allelic-bursting-independence-testing
description: Use when you have allele-specific single-cell count matrices and want
  to test per-gene whether maternal/paternal bursting events are independent (logOR
  ≈ 0), positively correlated, or anti-correlated, with FDR control.
when_to_use_negative:
- Input is raw read alignments rather than scBASE-estimated allelic counts — run the
  scBASE workflow (unique-read or weighted-allocation counting, beta-binomial P/B/M
  classification, optional partial pooling) first.
- The goal is testing differential burst frequency or burst size between two cell
  populations (e.g., txburstTEST.py), not within-gene allelic burst correlation.
- Data are bulk or UMI-only without allelic assignment (e.g., Larsson et al. UMI counts
  without allele-resolved counts), so maternal/paternal bursting cannot be correlated.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: SCALE
  role: Fits the bursting model to scBASE-estimated counts and computes per-gene burst-correlation
    logOR and independence tests
- name: scBASE
  role: Produces the allele-specific count matrices (unique-read, weighted allocation,
    partial pooling) used as SCALE input
  repo: github.com/churchill-lab/scBASE
- name: txburst
  role: Alternative burst-kinetics inference framework referenced by the paper; weighted
    allocation was not applied to those data
  repo: https://github.com/sandberg-lab/txburst
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:f0b0f4232809e29f3599f5105cf513031b7cdcdf9ff7d6d9b5f6de6d1c509ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCALE
  - scBASE
  - txburst
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# allelic-bursting-independence-testing

## Summary

Fit the SCALE bursting model to scBASE-estimated allele-specific count matrices and compute per-gene burst-correlation log odds ratios (logOR) to test whether maternal and paternal allelic bursting are statistically independent or coordinated.

## When to use

Use when you have allele-specific single-cell count matrices and want to test per-gene whether maternal/paternal bursting events are independent (logOR ≈ 0), positively correlated, or anti-correlated, with FDR control.

## When NOT to use

- Input is raw read alignments rather than scBASE-estimated allelic counts — run the scBASE workflow (unique-read or weighted-allocation counting, beta-binomial P/B/M classification, optional partial pooling) first.
- The goal is testing differential burst frequency or burst size between two cell populations (e.g., txburstTEST.py), not within-gene allelic burst correlation.
- Data are bulk or UMI-only without allelic assignment (e.g., Larsson et al. UMI counts without allele-resolved counts), so maternal/paternal bursting cannot be correlated.

## Inputs

- scBASE-estimated allele-specific count matrix (unique-reads method) for 8,290 analyzed genes
- scBASE-estimated allele-specific count matrix (EM-based weighted allocation of multi-reads)
- scBASE-estimated allele-specific count matrices after partial pooling (unique+PP and weighted+PP)
- Cell set of 122 blastocyst cells (early, mid, late) from Deng et al. CAST×B6 F1 hybrid

## Outputs

- Per-gene burst-correlation log odds ratio (logOR) and independence-test p-values for each of the four count matrices
- Benjamini–Hochberg FDR-adjusted p-values at 5% threshold
- Summary table scale_independence_results.csv: counts of significant genes (logOR > 0) and genes with |logOR| > 2 per method

## How to apply

Load the four scBASE-estimated allelic count matrices (unique-read, weighted-allocation, unique+partial-pooling, weighted+partial-pooling) for the ~8,290 analyzed genes from the blastocyst cells. For each matrix, fit the SCALE bursting model to estimate allele-specific bursting kinetics and compute the burst-correlation log odds ratio (logOR) per gene, then test deviation from independence and adjust p-values at 5% FDR with Benjamini–Hochberg. Tabulate per scBASE method the count of significant genes with logOR > 0 and the count with |logOR| > 2, comparing before vs. after pooling. Rationale: weighting and partial pooling reduce sampling zeros and variability in allelic proportions, and the paper found bursting is dependent and positively correlated (logOR > 0) for the majority of genes, rejecting both perfect independence and perfect dependence. Expect larger significant-gene counts after pooling (3,381 → 6,068 unique; 4,815 → 6,761 weighted), since partial pooling has the biggest impact when read coverage is low and the number of cells is large.

## Related tools

- **SCALE** (Fits the bursting model to scBASE-estimated counts and computes per-gene burst-correlation logOR and independence tests)
- **scBASE** (Produces the allele-specific count matrices (unique-read, weighted allocation, partial pooling) used as SCALE input) — github.com/churchill-lab/scBASE
- **txburst** (Alternative burst-kinetics inference framework referenced by the paper; weighted allocation was not applied to those data) — https://github.com/sandberg-lab/txburst

## Evaluation signals

- Reproduced significant-gene counts at 5% FDR must match 3,381 (unique), 4,815 (weighted), 6,068 (unique+PP), 6,761 (weighted+PP) out of 8,290 genes.
- |logOR| > 2 counts must match 2,845/3,763 before pooling and 5,622/6,209 after pooling (Fig 5, Supplementary Fig 2).
- Significant genes should have logOR > 0 (positive burst correlation), consistent with dependence rejecting perfect independence.
- After pooling, significant-gene counts must be larger than before pooling for both counting methods (pooling reduces sampling zeros).
- Output CSV scale_independence_results.csv must contain one row set per scBASE method with significant and |logOR|>2 tallies.

## Limitations

- MCMC settings (iterations, burn-in, priors) for the Bayesian model are deferred to Supplementary Methods, limiting out-of-the-box reproducibility.
- Partial pooling has the biggest impact when read coverage is low and the number of cells is large; with high coverage it changes little.
- Unique-read counting discards multi-mapping reads, leading to higher variability in allelic proportions, more sampling zeros, and potentially spurious dynamic/monoallelic findings.
- No data accession IDs (e.g., GEO/SRA) are given for the Deng et al. 286-cell dataset beyond platform GPL12112, complicating input acquisition.

## Evidence

- [discussion] Allelic bursting is statistically dependent and positively correlated (logOR > 0) for the majority of genes, rejecting both perfect independence and perfect dependence: [evidence span withheld — anchor status: not_found]
- [discussion] Weighted allocation resolves multi-reads via EM before counting: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [intro] Partial pooling improves cell-level estimates across cells in the same state: "Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state."
- [discussion] Partial pooling impact depends on coverage and cell number: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [abstract] Multi-read discarding inflates variability and sampling zeros: "discarding multi-mapping reads leads to higher variability in estimates of allelic proportions, an increased frequency of sampling zeros, and can lead to spurious findings of dynamic and monoallelic"
- [discussion] SCALE consumes scBASE-estimated counts: [evidence span withheld — anchor status: not_found]

---
name: bayesian-model-averaging-posterior-inclusion-probability
description: Use when gene-level DE summary statistics (beta coefficients and variances)
  from tools like MAST, edgeR, or zingeR are available and you want to strengthen
  per-gene DE calls by aggregating evidence across gene set fits rather than testing
  one gene at a time.
when_to_use_negative:
- No gene set annotations are available or the coverage rate (annotated genes / tested
  genes) falls below min_precent_annot=0.0025, so gene sets cannot be fitted
- Only raw single-cell count matrices are available and no per-gene summary statistics
  (beta and var(beta)) have been computed by a standalone DE method
- The analysis goal is gene set enrichment testing alone (use iDEA.fit with iDEA.louis
  / iDEA.FDR for calibrated gene set p-values rather than BMA gene aggregation)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Provides the BMA framework; iDEA.BMA aggregates DE evidence for given genes
    across all fitted gene sets
  repo: https://github.com/xzhoulab/iDEA
- name: MAST
  role: Standalone DE method whose beta/variance summary statistics are input to the
    BMA aggregation
- name: edgeR
  role: Standalone DE method (with ZINB-WaVE weighting) whose summary statistics are
    input to the BMA aggregation
- name: zingeR
  role: Standalone DE method (zinbwaveZinger workflow) whose summary statistics are
    input to the BMA aggregation
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:69f320d52f3ac30022fab4c627ca06b6579bc3534df339fe5421ff3a084cdb30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CreateiDEAObject
  - MAST
  - edgeR
  - iDEA
  - iDEA.BMA
  - iDEA.fit
  - zingeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bayesian-model-averaging-posterior-inclusion-probability

## Summary

Aggregates gene-level differential expression evidence across all gene sets using iDEA's Bayesian model averaging (BMA) approach, which computes posterior inclusion probabilities for genes across fitted gene set models. This augments standalone DE tools (MAST, edgeR, zingeR) by borrowing consistent DE evidence across similar genes in gene sets.

## When to use

Use when gene-level DE summary statistics (beta coefficients and variances) from tools like MAST, edgeR, or zingeR are available and you want to strengthen per-gene DE calls by aggregating evidence across gene set fits rather than testing one gene at a time.

## When NOT to use

- No gene set annotations are available or the coverage rate (annotated genes / tested genes) falls below min_precent_annot=0.0025, so gene sets cannot be fitted
- Only raw single-cell count matrices are available and no per-gene summary statistics (beta and var(beta)) have been computed by a standalone DE method
- The analysis goal is gene set enrichment testing alone (use iDEA.fit with iDEA.louis / iDEA.FDR for calibrated gene set p-values rather than BMA gene aggregation)

## Inputs

- Gene-level DE summary statistics (beta coefficient and its variance) from MAST, edgeR with ZINB-WaVE weighting, or zingeR
- Gene set annotations (e.g., humanGeneSets.RData or mouseGeneSets.RData from MSigDB-derived databases: BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome)
- Fitted iDEA object with gene set parameters (output of iDEA.fit with fit_noGS=FALSE)

## Outputs

- Aggregated gene-level DE evidence (BMA-combined per-gene statistics) for each gene across all gene set models
- DE gene calls usable for power/FDR evaluation at true FDR 5%
- Top-1500 DE gene lists for cross-method consistency (Jaccard index) comparison

## How to apply

First build an iDEA object with CreateiDEAObject (max_var_beta=100, min_precent_annot=0.0025) from the DE summary statistics and gene annotations, then fit the hierarchical Bayesian model with iDEA.fit (init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, fit_noGS=FALSE, modelVariant=FALSE). The iDEA.fit function fits gene sets jointly with DE, and iDEA.BMA then applies Bayesian model averaging to aggregate DE evidence for each gene across all available gene sets, producing combined gene-level inclusion evidence. The rationale is that analyzing one gene at a time leads to power loss because it fails to exploit consistent DE evidence across similar genes in gene sets. Finally compute empirical DE power at true FDR 5% and, for consistency, the Jaccard index of top-1500 DE gene lists between methods.

## Related tools

- **iDEA** (Provides the BMA framework; iDEA.BMA aggregates DE evidence for given genes across all fitted gene sets) — https://github.com/xzhoulab/iDEA
- **MAST** (Standalone DE method whose beta/variance summary statistics are input to the BMA aggregation)
- **edgeR** (Standalone DE method (with ZINB-WaVE weighting) whose summary statistics are input to the BMA aggregation)
- **zingeR** (Standalone DE method (zinbwaveZinger workflow) whose summary statistics are input to the BMA aggregation) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
idea_obj <- CreateiDEAObject(beta, var_beta, gene_id, annotation, max_var_beta=100, min_precent_annot=0.0025); idea_fit <- iDEA.fit(idea_obj, fit_noGS=FALSE, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE); idea_bma <- iDEA.BMA(idea_fit)
```

## Evaluation signals

- At tau1=5 and CR=10%, BMA-augmented power at true FDR 5% should be ~81% (zingeR-based), ~61% (MAST-based), and ~83% (edgeR-based), versus 65%, 52%, and 67% standalone
- Jaccard index of the top-1500 DE gene lists between methods should increase from ~0.59 (standalone) to ~0.77 (after iDEA BMA)
- iDEA object creation filters pass: gene variance < max_var_beta=100 and gene set coverage rate >= min_precent_annot=0.0025
- iDEA.fit converges within fit.tol=1e-5 over em_iter=15 EM iterations with min_degene=5 detected DE genes required
- Estimated FDR at the reported thresholds should be controlled near the nominal 5% level

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets
- The z-score modeling effectively assumes the prior distribution of true effect sizes is dependent on the standard errors, and subsequently the sample size
- DE analysis performed on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors

## Evidence

- [methods] BMA aggregates gene evidence across gene sets: "we developed a Bayesian model averaging (BMA) approach to aggregate DE evidence for any given genes across all available gene sets"
- [intro] one-gene-at-a-time rationale: "Analyzing one gene at a time can lead to potential power loss, as this approach fails to exploit consistent DE evidence across similar genes"
- [methods] object filter parameters: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [abstract] workflow: "iDEA uses only DE summary statistics as input, enabling effective data modeling through complementing and pairing with various existing DE methods"
- [methods] model fitting entry point: "The function iDEA.fit fits the iDEA model on the input data."

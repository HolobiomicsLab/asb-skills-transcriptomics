---
name: gene-level-pvalue-table-preparation
description: Use when a downstream pareg regression call needs a gene-by-pvalue dataframe
  and you only have raw matched tumor/normal expression data (e.g. GSE62944 TCGA-BRCA)
  that still requires a differential expression test per gene.
when_to_use_negative:
- A gene-level P-value table already exists in pareg's expected format — go directly
  to the pareg regression call instead of re-running limma.
- The analysis goal is pathway-level enrichment scoring itself (pareg's regression
  or enrichmentplot visualization), not producing its gene-level input.
- The P-values are synthetic (e.g. sampled from Beta/Uniform distributions as in the
  pareg vignette benchmark) and no real differential expression test is needed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: limma
  role: fits the tumor-vs-normal linear model with patient-pair blocking and eBayes
    moderation to produce raw per-gene P-values
- name: GEOquery
  role: downloads the GSE62944 expression matrix and sample metadata from GEO
- name: tidyverse
  role: subsets the 113 matched tumor/normal samples and reshapes the results into
    the gene/pvalue dataframe
- name: pareg
  role: downstream consumer of de_pvalues.csv; regresses the differential expression
    p-values to estimate pathway enrichment scores
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: implementation language for the GEOquery/limma/tidyverse workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:50ad1b0777ce12ad2a76cdb2c0830db12ea3681742e20696487e9d648d4b1383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - GEOquery
  - R
  - limma
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gene-level-pvalue-table-preparation

## Summary

Produces the gene-level differential expression P-value table that pareg consumes as input, by running a paired limma tumor-vs-normal analysis on TCGA-BRCA (GSE62944) expression data and exporting a two-column gene/pvalue CSV (de_pvalues.csv).

## When to use

Use when a downstream pareg regression call needs a gene-by-pvalue dataframe and you only have raw matched tumor/normal expression data (e.g. GSE62944 TCGA-BRCA) that still requires a differential expression test per gene.

## When NOT to use

- A gene-level P-value table already exists in pareg's expected format — go directly to the pareg regression call instead of re-running limma.
- The analysis goal is pathway-level enrichment scoring itself (pareg's regression or enrichmentplot visualization), not producing its gene-level input.
- The P-values are synthetic (e.g. sampled from Beta/Uniform distributions as in the pareg vignette benchmark) and no real differential expression test is needed.

## Inputs

- GSE62944 expression matrix from GEO (tumor and matched normal TCGA-BRCA breast cancer samples)
- GEO sample metadata / TCGA sample annotations identifying 113 tumor-normal matched patient pairs
- gene identifier mapping for the expression matrix rows

## Outputs

- de_pvalues.csv with columns gene and pvalue (one row per gene, raw limma/eBayes P-values in [0, 1])

## How to apply

Download the GSE62944 expression matrix and sample metadata from GEO with the GEOquery R package, then subset to the 113 tumor and matched normal TCGA-BRCA breast cancer samples using the TCGA sample annotations. Fit a limma linear model comparing tumor versus normal expression for every gene, blocking on the patient pair so matched samples are treated as within-patient comparisons, and apply empirical Bayes moderation (eBayes) to obtain moderated t-statistics and raw per-gene P-values. Export the resulting table as de_pvalues.csv with exactly the columns gene and pvalue, which is the dataframe format pareg expects when it regresses differential expression p-values onto pathway membership. Use tidyverse for subsetting and reshaping; keep P-values raw (unadjusted) so they fall in [0, 1] as required by pareg's beta/bernoulli-style regression response.

## Related tools

- **limma** (fits the tumor-vs-normal linear model with patient-pair blocking and eBayes moderation to produce raw per-gene P-values)
- **GEOquery** (downloads the GSE62944 expression matrix and sample metadata from GEO)
- **tidyverse** (subsets the 113 matched tumor/normal samples and reshapes the results into the gene/pvalue dataframe)
- **pareg** (downstream consumer of de_pvalues.csv; regresses the differential expression p-values to estimate pathway enrichment scores) — https://github.com/cbg-ethz/pareg
- **R** (implementation language for the GEOquery/limma/tidyverse workflow)

## Examples

```
fit <- eBayes(lmFit(expr, model.matrix(~group), block = patient)); write.csv(data.frame(gene = rownames(fit), pvalue = fit$p.value), "de_pvalues.csv", row.names = FALSE)
```

## Evaluation signals

- de_pvalues.csv exists and contains exactly the gene and pvalue columns pareg expects
- all exported P-values lie in [0, 1] and are raw (unadjusted), so the table can be passed directly to the pareg regression call
- the table covers one P-value per gene with no missing values after the limma/eBayes fit on the 113 matched pairs
- patient-pair blocking was included in lmFit, so the design uses the matched tumor-normal structure rather than an unpaired two-group test

## Limitations

- The workflow is tied to the specific GSE62944 TCGA-BRCA cohort of 113 tumor and matched normal samples; other datasets need their own pairing/blocking annotations.
- pareg expects raw P-values in [0, 1]; exporting adjusted P-values or an incompatible column schema will break the regression call.
- As an exploratory TCGA breast cancer analysis, the resulting enrichment targets are hypothesis-generating rather than validated findings.

## Evidence

- [methods] It estimates pathway enrichment scores by regressing differential expression p-values: "It estimates pathway enrichment scores by regressing differential expression p-values"
- [readme] BiocManager install line: "Install the latest stable version from Bioconductor:
```r
BiocManager::install("pareg")
```"
- [abstract] exploratory TCGA BRCA analysis: "demonstrate the ability of our method to recover known pathways as well as to suggest novel treatment targets in an exploratory analysis using breast cancer samples from TCGA"

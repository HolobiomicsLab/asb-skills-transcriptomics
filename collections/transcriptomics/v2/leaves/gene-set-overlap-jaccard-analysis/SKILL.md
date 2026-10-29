---
name: gene-set-overlap-jaccard-analysis
description: Use when comparing significant gene-set or DE gene call lists from two
  or more enrichment/DE methods and you need a set-level concordance metric to confirm
  that a more method's extra hits are consistent with existing calls.
when_to_use_negative:
- When you only have p-value distributions and no discrete call sets — use genomic
  control (lambda_gc) or FDR curves instead
- When gene sets have wildly different sizes and overlap is dominated by large annotated
  sets; the article notes iDEA does not explicitly account for gene set overlap causing
  non-independence among gene sets
- When comparing methods run on different inputs or annotations — Jaccard is only
  interpretable when all methods share the same zingeR summary statistics and gene
  set universe
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Produces the integrative DE + GSE call sets (iDEA.fit, iDEA.louis, iDEA.FDR,
    iDEA.BMA) that are compared against competitor call sets via Jaccard overlap
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competitive GSE method whose significant gene set list (1,742 sets at empirical
    FDR 5%) provides the strongest competitor for the Jaccard overlap comparison
- name: CAMERA
  role: Competitive GSE method contributing a significant gene set list to the pairwise
    overlap analysis
- name: PAGE
  role: Competitive GSE method contributing a significant gene set list to the pairwise
    overlap analysis
- name: GSEA
  role: Competitive GSE method contributing a significant gene set list to the pairwise
    overlap analysis
- name: zingeR
  role: Generates the shared per-gene summary statistics (beta, beta variance) on
    which all compared methods run, ensuring Jaccard overlap is computed on a common
    basis
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:f9311ea3f5cbb7ee1d0f22b4401ac78e2e5632f940c809a7f9bbac7c85ec6608
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - GSEA
  - PAGE
  - fGSEA
  - iDEA
  - zingeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-set-overlap-jaccard-analysis

## Summary

Quantify the pairwise Jaccard overlap between significant gene sets (or DE genes) reported by competing enrichment/DE methods, to measure how concordant their discoveries are. In the iDEA Nature Communications 2020 benchmark this Jaccard trajectory (rising from ~0.10 to ~0.25 on the human ESC DEC vs EC scRNA-seq data) demonstrates that iDEA's larger call sets overlap rather than diverge from fGSEA, CAMERA, PAGE, GSEA, and hypergeometric results.

## When to use

Use when comparing significant gene-set or DE gene call lists from two or more enrichment/DE methods and you need a set-level concordance metric to confirm that a more powerful method's extra hits are consistent with existing calls.

## When NOT to use

- When you only have p-value distributions and no discrete call sets — use genomic control (lambda_gc) or FDR curves instead
- When gene sets have wildly different sizes and overlap is dominated by large annotated sets; the article notes iDEA does not explicitly account for gene set overlap causing non-independence among gene sets
- When comparing methods run on different inputs or annotations — Jaccard is only interpretable when all methods share the same zingeR summary statistics and gene set universe

## Inputs

- Significant gene set lists per method (FDR ≤ 5%) from iDEA, fGSEA, CAMERA, PAGE, GSEA, and over-representation analysis
- Significant DE gene lists per method (FDR ≤ 1%), e.g. 2,753 iDEA genes vs 1,673 for the best competitor
- Human gene set annotations (MSigDB-derived GO, KEGG, Reactome, BioCarta, PubChem Compound, ImmuneSigDB, PID on GRCh37) from humanGeneSets.RData
- zingeR-DESeq2 summary statistics (beta coefficient and beta variance) from the Chu et al. human ESC DEC vs EC scRNA-seq data

## Outputs

- Pairwise Jaccard overlap matrix between methods' significant gene set call sets
- Jaccard trajectory across ranking thresholds (e.g. 0.10 → 0.25) as reported in Fig. 4
- Rows appended to idea_dec_ec_results.csv alongside FDR-threshold call counts and lambda_gc genomic control factors

## How to apply

Run each competing GSE/DE method (iDEA via iDEA.fit plus iDEA.louis and iDEA.BMA, versus fGSEA, CAMERA, PAGE, GSEA, and Fisher-style over-representation) on the same zingeR-DESeq2 summary statistics for the DEC vs EC comparison. For each method, define its call set as gene sets passing the empirical FDR threshold (5% for GSE, computed via iDEA.fit.null permutation and iDEA.FDR) or genes passing DE FDR 1%. Compute pairwise Jaccard indices J(A,B) = |A∩B|/|A∪B| between all method pairs (and between nested subsets such as endoderm-differentiation genes), writing the trajectory to idea_dec_ec_results.csv. Interpretation: a rising Jaccard (here 0.10 → 0.25) alongside iDEA's higher absolute counts (2,106 vs fGSEA's 1,742 gene sets at FDR 5%) indicates the integrative method recovers existing findings while adding consistent new ones rather than discordant hits.

## Related tools

- **iDEA** (Produces the integrative DE + GSE call sets (iDEA.fit, iDEA.louis, iDEA.FDR, iDEA.BMA) that are compared against competitor call sets via Jaccard overlap) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competitive GSE method whose significant gene set list (1,742 sets at empirical FDR 5%) provides the strongest competitor for the Jaccard overlap comparison)
- **CAMERA** (Competitive GSE method contributing a significant gene set list to the pairwise overlap analysis)
- **PAGE** (Competitive GSE method contributing a significant gene set list to the pairwise overlap analysis)
- **GSEA** (Competitive GSE method contributing a significant gene set list to the pairwise overlap analysis)
- **zingeR** (Generates the shared per-gene summary statistics (beta, beta variance) on which all compared methods run, ensuring Jaccard overlap is computed on a common basis) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
jaccard <- function(a, b) length(intersect(a, b)) / length(union(a, b)); jac <- jaccard(idea_sets_fdr5, fgsea_sets_fdr5)
```

## Evaluation signals

- On the human ESC DEC vs EC data, the pairwise Jaccard between iDEA's FDR 5% gene set call set and each competitor's should follow the reported trajectory from ~0.10 to ~0.25 within reasonable numerical tolerance of Fig. 4
- The overlap denominator must use the union of each method's significant sets over the same gene set universe (MSigDB GRCh37 annotations, min_precent_annot = 0.0025) — check set counts match FDR 5% counts (2,106 iDEA, 1,742 fGSEA) before computing Jaccard
- Jaccard should be recomputed on the DE gene call sets at FDR 1% (2,753 iDEA vs 1,673 best competitor) and on the endoderm-differentiation gene subsets (1,119 vs 706) for consistency
- J(A,B) must be symmetric and bounded in [0,1], and J(A,A) = 1 for every method's call set — a basic schema/invariant check on the computed matrix
- All quantities written to idea_dec_ec_results.csv should reproduce the published lambda_gc values (1.13, 1.02, 0.96, 1.01, 0.29) from the same run to confirm the call sets feeding the Jaccard computation are the intended ones

## Limitations

- Jaccard overlap treats all gene sets equally and does not correct for gene set overlap / non-independence among gene set tests, which the authors flag as a limitation not explicitly accounted for by iDEA
- Overlap statistics depend on the FDR thresholds used; changing the 5% GSE or 1% DE cutoffs changes both set sizes and Jaccard, so the reported 0.10 → 0.25 trajectory is threshold-specific
- The comparison is valid only when all methods consume identical summary statistics and the same gene set annotation database; results are dataset-specific (human ESC DEC vs EC here)

## Evidence

- [results] iDEA identified 2,106 significantly enriched gene sets, 20.9% more than the next best method fGSEA (1,742).: [evidence span withheld — anchor status: not_found]
- [abstract] iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods on three scRNA-seq data sets: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [methods] The function iDEA.FDR calculates the estimated FDR for each gene set given the permuted null distribution of p-values for gene sets.: "The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets."
- [discussion] GSE methods that do not explicitly account for gene-gene correlation appear to provide more calibrated p-values.: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] devtools::install_github('xzhoulab/iDEA'): "# install the iDEA package
devtools::install_github('xzhoulab/iDEA')"

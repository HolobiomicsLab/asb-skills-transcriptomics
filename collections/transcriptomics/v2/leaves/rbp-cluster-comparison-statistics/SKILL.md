---
name: rbp-cluster-comparison-statistics
description: Use when a merged CRIT feature matrix plus NMF cluster membership labels
  are available and the goal is to rank features (e.g., Zg splicing similarity, Zp
  alteration proportion) that separate predicted circRNA regulators from other RBPs.
when_to_use_negative:
- The goal is to predict/cluster circRNA regulators from raw multi-omics data rather
  than characterize an already-produced NMF cluster — use the full CRIT NMF.R pipeline
  instead
- Input is not a merged CRIT feature matrix (e.g., raw TCGA alteration files or MiOncoCirc
  FPKM tables that have not passed through the GET_*.R feature scripts)
- There is only one group (no cluster labels) so no two-sample comparison is defined
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Language in which the per-feature Welch t tests and Benjamini-Hochberg correction
    over the 51 CRIT features are implemented
- name: CRIT
  role: Pipeline whose feature matrix (MATRIX.txt) and NMF cluster labels are the
    inputs being statistically compared
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:30755b38d34d40644b36b206e8246f82ecba2471f1f37820ae2929a2115ad484
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# rbp-cluster-comparison-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A per-feature two-sample Welch t-test procedure in R that compares the 51 CRIT features (ρm, Zp, Zs, Zg, C value) between NMF-predicted circRNA-regulating RBPs (90 genes) and the remaining 916 RBPs, with Benjamini-Hochberg correction, to identify which feature categories distinguish authentic regulators.

## When to use

Use when a merged CRIT feature matrix plus NMF cluster membership labels are available and the goal is to rank features (e.g., Zg splicing similarity, Zp alteration proportion) that separate predicted circRNA regulators from other RBPs.

## When NOT to use

- The goal is to predict/cluster circRNA regulators from raw multi-omics data rather than characterize an already-produced NMF cluster — use the full CRIT NMF.R pipeline instead
- Input is not a merged CRIT feature matrix (e.g., raw TCGA alteration files or MiOncoCirc FPKM tables that have not passed through the GET_*.R feature scripts)
- There is only one group (no cluster labels) so no two-sample comparison is defined

## Inputs

- MATRIX.txt — merged CRIT feature matrix (51 feature columns: ρm, Zp, Zs, Zg, C value) for 1006 RBPs
- NMFgroup_XXclusterYYrun.txt — NMF cluster membership file distinguishing the 90 predicted RBPs from the remaining 916 RBPs

## Outputs

- feature_ttest_results.csv — per-feature Welch t statistics, raw p-values, and BH-adjusted p-values ranked by adjusted significance

## How to apply

Load MATRIX.txt (the merged candidate-regulator feature matrix produced by GET_feauters_MATRIX.R) and the NMF cluster output (NMFgroup_XXclusterYYrun.txt) that labels the 90 predicted RBPs versus the 916 remaining RBPs. Split rows into the predicted-regulator group and the non-predicted group using the cluster labels. In R, run an independent two-sample Welch t test for each of the 51 features — the ρm (mean top-50 gene-circRNA Spearman correlation), Zp (gene alteration proportion Z-score), Zs (survival p-value Z-score), Zg (GO:0000398 mRNA splicing similarity Z-score from GoSemSim), and C value (number of bound circRNAs from RNAct) columns. Apply Benjamini-Hochberg correction across all 51 tests and rank features by adjusted significance. Accept the result when Zg and Zp emerge as significantly different between groups (adjusted p < 0.05), consistent with the finding that authentic regulators are core splicing proteins/peripheral factors harboring more cancer alterations.

## Related tools

- **R** (Language in which the per-feature Welch t tests and Benjamini-Hochberg correction over the 51 CRIT features are implemented)
- **CRIT** (Pipeline whose feature matrix (MATRIX.txt) and NMF cluster labels are the inputs being statistically compared) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
Rscript -e 'm<-read.delim("MATRIX.txt"); g<-read.delim("NMFgroup_k2cluster1run.txt"); fs<-grep("^(rho_m|Zp|Zs|Zg|C_value)",names(m),value=TRUE); p<-sapply(fs,function(f) t.test(m[[f]]~g$group)$p.value); out<-data.frame(feature=fs,p=p,padj=p.adjust(p,"BH")); write.csv(out,"feature_ttest_results.csv",row.names=FALSE)'
```

## Evaluation signals

- Zg (GO:0000398 splicing similarity) and Zp (alteration proportion) both show BH-adjusted p < 0.05 between the predicted-regulator and non-predicted groups
- Exactly 51 independent t tests are run (one per CRIT feature column) and 51 BH-adjusted p-values are written to feature_ttest_results.csv
- Group sizes derived from cluster labels match the expected 90 predicted RBPs vs 916 remaining RBPs
- Adjusted p-values lie in [0,1] and are monotone with the ranked raw p-values (Benjamini-Hochberg property)
- Directionality check: the regulator group shows higher Zg (closer to core splicing GO:0000398) and higher Zp (more alterations across the 12 cancer types/19 TCGA batches)

## Limitations

- The comparison is restricted to the 51 CRIT features computed from 12 cancer types; features rely on RNAct/catRAPID predictions and CircBase/Gencode alignments, so misalignment-driven C values propagate into the test
- CRIT's generalization beyond cancers (e.g., neurodegenerative disease) is untested, so feature contrasts may not transfer to other disease contexts
- Welch t tests assume approximate normality of each feature across groups; strongly skewed features (e.g., C value counts) may need nonparametric alternatives

## Evidence

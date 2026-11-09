---
name: fdr-based-hit-calling-thresholding
description: Use when you have per-cluster pseudobulk differential state results (gene,
  cluster_id, logFC, p-value) from muscat::pbDS/edgeR and must declare significant
  hits, e.g. filtering the mouse cortex LPS dataset at FDR < 0.05 and |logFC| > 1.
when_to_use_negative:
- Cell-level mixed-model results (mmDS) output when the analysis goal is sample-level
  pseudobulk inference rather than per-cell testing
- Pseudobulk differential detection (pbDD / fun='num.detected') results, where the
  hit definition targets detection changes rather than expression state changes
- Single-sample comparisons with no biological replicates, where FDR adjustment across
  genes cannot be meaningfully anchored to a sample-level design
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: Provides pbDS (pseudobulk differential state testing with edgeR) and bbhw/gBH
    for FDR-based hit calling on per-subpopulation hypotheses
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Bulk RNA-seq differential engine applied to pseudobulk counts, supplying the
    logFC and p-values that are thresholded at FDR < 0.05, |logFC| > 1
- name: R
  role: Implementation environment for aggregation, pbDS testing, and thresholded
    hit extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:6f1e667929e9c6fef8401e04d62cef0d624bc5bd3c2ba6580cecb03c79b9e536
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - muscat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fdr-based-hit-calling-thresholding

## Summary

Converts per-subpopulation pseudobulk differential state test results (edgeR p-values from muscat::pbDS) into a call set of differential state hits by filtering on false discovery rate (FDR) adjusted p-values and a minimum effect size. In the muscat mouse cortex LPS snRNA-seq case study this reduces genome-wide tests to ~915 DS genes at FDR < 0.05 and |logFC| > 1.

## When to use

Use when you have per-cluster pseudobulk differential state results (gene, cluster_id, logFC, p-value) from muscat::pbDS/edgeR and must declare significant hits, e.g. filtering the mouse cortex LPS dataset at FDR < 0.05 and |logFC| > 1.

## When NOT to use

- Cell-level mixed-model results (mmDS) output when the analysis goal is sample-level pseudobulk inference rather than per-cell testing
- Pseudobulk differential detection (pbDD / fun='num.detected') results, where the hit definition targets detection changes rather than expression state changes
- Single-sample comparisons with no biological replicates, where FDR adjustment across genes cannot be meaningfully anchored to a sample-level design

## Inputs

- Per-subpopulation differential state result table from muscat::pbDS (gene, cluster_id, logFC, p-value) as a data.frame
- Pseudobulk SingleCellExperiment/SummarizedExperiment produced by muscat::aggregateData
- Edge design: condition groups (LPS vs. vehicle) and sample IDs for contrast specification

## Outputs

- Filtered set of differential state hits (genes passing FDR < 0.05 and |logFC| > 1) per subpopulation
- Full per-subpopulation results table written as lps_ds_results.csv with columns gene, cluster_id, logFC, p-value, adjusted p-value

## How to apply

After aggregating single-cell counts to pseudobulk sums per sample-by-subpopulation and running muscat::pbDS with edgeR testing LPS vs. vehicle within each subpopulation, apply multiple-testing correction across genes per subpopulation to obtain adjusted p-values. Define hits as genes with FDR < 0.05 AND |logFC| > 1 in at least one subpopulation; for the mouse cortex snRNA-seq dataset (four replicates per condition) this recovers on the order of ~915 DS genes, most detected in a single subpopulation. Inspect the surviving set for the expected biology: a distinct gene set upregulated across subpopulations (e.g. cluster_id 3) and endothelial cells showing the strongest response. Retain the full table of gene, cluster_id, logFC, p-value, and adjusted p-value for reporting rather than only the thresholded subset. If independent bulk RNA-seq data are available, muscat's bulk-based hypothesis weighing (bbhw, with gBH correction) can be layered on top of pbDS results to re-weight FDR adjustment.

## Related tools

- **muscat** (Provides pbDS (pseudobulk differential state testing with edgeR) and bbhw/gBH for FDR-based hit calling on per-subpopulation hypotheses) — github.com/HelenaLC/muscat
- **edgeR** (Bulk RNA-seq differential engine applied to pseudobulk counts, supplying the logFC and p-values that are thresholded at FDR < 0.05, |logFC| > 1)
- **R** (Implementation environment for aggregation, pbDS testing, and thresholded hit extraction)

## Examples

```
pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")); ds_pb <- pbDS(pb, method = "edgeR")
```

## Evaluation signals

- Hit count invariant: filtering at FDR < 0.05 and |logFC| > 1 yields approximately 915 DS genes detected in at least one subpopulation
- Specificity invariant: 751 of the DS hits are detected in only a single subpopulation, so most clusters should have short gene lists
- Biological sanity check: consensus cluster_id 3 shows upregulation across subpopulations and endothelial cells exhibit the strongest response
- Schema check: output table lps_ds_results.csv contains gene, cluster_id, logFC, p-value, and adjusted p-value columns with adjusted p-values <= raw p-values
- Every called hit satisfies both thresholds (max adjusted p < 0.05; min |logFC| > 1) with no rows retained on a single criterion

## Limitations

- LogFC estimates can be attenuated for some combinations depending on the scale of data analyzed, so |logFC| > 1 filtering may be conservative
- Statistical power is potentially strongly biased toward larger cell populations, with decreased power for rarer subpopulations
- FDR control shows instability with respect to the number of cells, and the origins of this instability are not yet understood

## Evidence

- [other] Pseudobulk DS analysis with edgeR identified 915 genes with differential state (FDR < 0.05, |logFC| > 1) in at least one subpopulation: [evidence span withheld — anchor status: not_found]
- [readme] README quick guide pseudobulk commands: "# run pseudobulk (aggregation-based) DS analysis
ds_pb <- pbDS(pb, method = "edgeR")"
- [intro] Article intro LPS case study objective: "we applied pseudobulk DS analysis to unravel subpopulation-specific responses within brain cortex tissue from mice treated with lipopolysaccharide (LPS)."
- [methods] bbhw FDR machinery on pbDS hypotheses: "`bbhw` can be executed on top of any per-celltype differential state analysis as produced by `pbDS` or `mmDS`"

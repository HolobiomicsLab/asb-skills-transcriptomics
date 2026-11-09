---
name: log2fc-intensity-computation
description: Use when inferring potential LR associations between cell clusters from
  scRNA-seq data and you need per-pair intensity and p-value statistics from cluster-pair
  differential expression results.
when_to_use_negative:
- When the goal is pathway-filtered or functionally prioritized LR interactions —
  this skill only computes expression-based statistics; pathway screening requires
  findLRpath/scorePath/filterLR downstream
- When the input expression matrix is raw counts rather than library-size-normalized,
  log-transformed values, since log2FC-based intensities would be distorted
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: CommPath
  role: R package whose findLRmarker and findLRpairs functions compute the log2FC-based
    interaction intensity and combined p-value per LR pair
  repo: https://github.com/yingyonghui/CommPath
- name: R
  role: Runtime environment for installing CommPath via devtools and executing the
    LR statistics computation and CSV export
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:60b9120dce5f6fa125e5400d1496ea675f8289198d276be568543f47acfad286
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yingyonghui/CommPath
  repo_url: https://github.com/yingyonghui/CommPath
  tools:
  - CommPath
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# log2fc-intensity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies ligand-receptor association strength between two cell clusters in CommPath by multiplying the log2FC of the ligand in the sender cluster with the log2FC of the receptor in the receiver cluster, and combining differential-expression p-values as P_lr,AB = 1 − (1−P_l,A)(1−P_r,B).

## When to use

Use when inferring potential LR associations between cell clusters from scRNA-seq data and you need per-pair intensity and p-value statistics from cluster-pair differential expression results.

## When NOT to use

- When the goal is pathway-filtered or functionally prioritized LR interactions — this skill only computes expression-based statistics; pathway screening requires findLRpath/scorePath/filterLR downstream
- When the input expression matrix is raw counts rather than library-size-normalized, log-transformed values, since log2FC-based intensities would be distorted

## Inputs

- CommPath object created via createCommPath(expr.mat, cell.info, species = 'hsapiens') from a library-size-normalized, log-transformed gene × cell expression matrix (e.g. tumor.expr) and a matching cell label vector (e.g. tumor.label)
- Differential expression results for marker ligands and receptors from findLRmarker(method = 'wilcox.test')
- Thresholds logFC.thre = 0 and p.thre = 0.01 for findLRpairs

## Outputs

- Per-LR-pair statistics table with columns log2FC.LR (product of ligand and receptor log2FC), P.val.LR (combined p-value), and P.val.adj.LR for each sender-receiver cluster pair (interact[['InteractGene']])
- Counts of significant LR pairs and overall interaction intensity per cluster pair (interact[['InteractNumer']])
- Exported CSV (e.g. LR_pairs_statistics.csv) of the filtered cluster-pair LR statistics

## How to apply

First run findLRmarker with method='wilcox.test' to obtain ligand log2FC and adjusted p-values in sender cluster CA and receptor log2FC and adjusted p-values in receiver cluster CB from differential expression tests. Then run findLRpairs(object, logFC.thre = 0, p.thre = 0.01) on the CommPath object so only significantly highly expressed markers enter the computation. For each candidate LR pair, compute the interaction intensity as the product Intensity_lr,AB = log2FC_l,A × log2FC_r,B (stored as log2FC.LR) and combine the two p-values as P_lr,AB = 1 − (1−P_l,A)(1−P_r,B) (stored as P.val.LR, adjusted as P.val.adj.LR). Per-pair statistics are stored in tumor.obj@interact[['InteractGene']] and pair counts/intensities per cluster pair in tumor.obj@interact[['InteractNumer']]. Export the filtered cluster-pair table (e.g. to LR_pairs_statistics.csv) keeping only pairs passing logFC.thre = 0 and p.thre = 0.01.

## Related tools

- **CommPath** (R package whose findLRmarker and findLRpairs functions compute the log2FC-based interaction intensity and combined p-value per LR pair) — https://github.com/yingyonghui/CommPath
- **R** (Runtime environment for installing CommPath via devtools and executing the LR statistics computation and CSV export) — https://github.com/yingyonghui/CommPath

## Examples

```
tumor.obj <- findLRpairs(object = tumor.obj, logFC.thre = 0, p.thre = 0.01); stats <- tumor.obj@interact[['InteractGene']]; write.csv(stats, 'LR_pairs_statistics.csv')
```

## Evaluation signals

- Output table exists and contains the columns log2FC.LR, P.val.LR, and P.val.adj.LR
- Every retained pair passes the thresholds logFC.thre = 0 and p.thre = 0.01
- log2FC.LR equals the product of the ligand's log2FC in the sender cluster and the receptor's log2FC in the receiver cluster
- P.val.LR equals 1 − (1−P_l,A)(1−P_r,B) using the adjusted differential expression p-values from findLRmarker
- Pair counts and overall intensity per cluster pair in interact[['InteractNumer']] are consistent with the per-pair rows in interact[['InteractGene']]

## Limitations

- The intensity is purely expression-based (product of log2FC values); it does not consider activated pathways in the receiver cell, so pairs are not functionally prioritized at this stage
- Only significant marker ligands and receptors identified by findLRmarker enter the computation, so lowly expressed but functionally relevant pairs may be missed
- Interaction counts/intensities reflect statistical significance of expression, not proven physical protein interaction

## Evidence

- [methods] CommPath computes an intensity score as the product of log2FC values of ligand and receptor (Intensity_lr,AB = log2FC_l,A × log2FC_r,B) and a combined p-value P_lr,AB = 1 − (1−P_l,A)(1−P_r,B): [evidence span withheld — anchor status: not_found]

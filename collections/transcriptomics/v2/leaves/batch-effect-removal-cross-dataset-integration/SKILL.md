---
name: batch-effect-removal-cross-dataset-integration
description: Use when combining two scRNA-Seq datasets generated in different labs/protocols
  (e.g., Hockley GSE102962 target with Usoskin GSE59739 source) prior to clustering,
  and you want to test whether batch correction improves cluster recovery.
when_to_use_negative:
- When source and target share only a small set of transcripts — the paper notes a
  loss of (probably) vital information is inevitable since only genes present in both
  datasets can be used
- When you only have a single dataset with no second dataset to integrate — batch-effect
  removal for cross-dataset combination is not applicable and plain SC3 clustering
  suffices
- When the comparison question is about imputation of zero-inflated values rather
  than cross-lab technical effects — MAGIC imputation, not Seurat batch removal, is
  the corresponding condition
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: CCA-based batch effect removal applied to combine the Hockley and Usoskin
    datasets as a robustness pre-processing condition
  repo: https://github.com/nicococo/scRNA/blob/master/R/seurat_cca_hockley_usoskin.R
- name: SC3
  role: Downstream clustering method applied to the batch-corrected data into k=7
    clusters, repeated 1000 times to assess robustness
  repo: https://github.com/nicococo/scRNA
- name: MAGIC
  role: Alternative pre-processing condition (missing-value imputation) compared against
    Seurat batch removal in the robustness table
  repo: https://github.com/nicococo/scRNA/blob/master/matlab/magic_hockley_usoskin.m
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:77906585aeb7114bcca33c4d56bda0ee4703aeabb46062e06963097761f40237
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAGIC
  - NMF
  - SC3
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# batch-effect-removal-cross-dataset-integration

## Summary

Optional pre-processing step from Mieth et al. (2019) in which Seurat's CCA-based batch effect removal (Butler et al.) is applied to combine the Hockley et al. 2018 colonic DRG target dataset with the Usoskin et al. 2014 DRG source dataset before transfer-learning-based SC3 clustering, testing whether correction of cross-dataset technical effects improves cluster recovery.

## When to use

Use when combining two scRNA-Seq datasets generated in different labs/protocols (e.g., Hockley GSE102962 target with Usoskin GSE59739 source) prior to clustering, and you want to test whether batch correction improves cluster recovery.

## When NOT to use

- When source and target share only a small set of transcripts — the paper notes a loss of (probably) vital information is inevitable since only genes present in both datasets can be used
- When you only have a single dataset with no second dataset to integrate — batch-effect removal for cross-dataset combination is not applicable and plain SC3 clustering suffices
- When the comparison question is about imputation of zero-inflated values rather than cross-lab technical effects — MAGIC imputation, not Seurat batch removal, is the corresponding condition

## Inputs

- Hockley et al. 2018 colonic DRG scRNA-Seq target dataset (GEO GSE102962, TPM, 314 cells)
- Usoskin et al. 2014 DRG scRNA-Seq source dataset (GEO GSE59739, CPM, 622 cells)
- Gene expression matrix restricted to the 4402 genes common to both datasets after cell filter, gene filter, and log-transformation with pseudo-count 1
- Source dictionary Hsrc learned via NMF with elastic net regularization on the Usoskin data

## Outputs

- Seurat batch-corrected combined Hockley+Usoskin expression matrix
- SC3 cluster assignments (7 clusters) for TransferCluster, TargetCluster, and ConcatenateCluster under the Seurat pre-processing condition
- robustness_table.csv with counts of mNP/mNFa separation and pPEP (cluster 4) / pNF (cluster 2) identification over 1000 SC3 replications per method per pre-processing condition

## How to apply

Run Seurat batch effect removal as one of three pre-processing conditions (original, Seurat, MAGIC) in the robustness experiment, using the provided R script (R/seurat_cca_hockley_usoskin.R) on the Hockley (TPM, 314 cells) and Usoskin (CPM, 622 cells) datasets after the standard three pre-processing steps (cell filter, ubiquitous/rare gene filter, log-transformation with pseudo-count 1) on the 4402 shared genes. Feed the batch-corrected data into the three clustering variants: TransferCluster (SC3 on theta-weighted mixture Xtrg_new = theta*Hsrc*W'trg + (1-theta)*Xtrg with theta chosen via KTA), TargetCluster (SC3 on target alone), and ConcatenateCluster (SC3 on concatenated data). Because SC3 is non-convex, repeat each method 1000 times and count how often the mNP/mNFa clusters are separated and how often the pPEP (cluster 4) and pNF (cluster 2) clusters are identified. Tabulate counts per method per pre-processing condition (original, Seurat, MAGIC) via scripts/plots/evaluate_hockley_robustness_magic_seurat.py and save as robustness_table.csv; the rationale is that removing batch effects between the two datasets should make the transferred source structure more compatible with the target.

## Related tools

- **Seurat** (CCA-based batch effect removal applied to combine the Hockley and Usoskin datasets as a robustness pre-processing condition) — https://github.com/nicococo/scRNA/blob/master/R/seurat_cca_hockley_usoskin.R
- **SC3** (Downstream clustering method applied to the batch-corrected data into k=7 clusters, repeated 1000 times to assess robustness) — https://github.com/nicococo/scRNA
- **MAGIC** (Alternative pre-processing condition (missing-value imputation) compared against Seurat batch removal in the robustness table) — https://github.com/nicococo/scRNA/blob/master/matlab/magic_hockley_usoskin.m

## Examples

```
Rscript R/seurat_cca_hockley_usoskin.R; python scripts/experiments/main_wrapper_hockley_robustness_seurat.py
```

## Evaluation signals

- robustness_table.csv shows, for the Seurat pre-processing condition, how often TransferCluster separates the mNP/mNFa clusters and identifies pPEP (cluster 4) and pNF (cluster 2) out of 1000 replications
- TransferCluster should outperform both TargetCluster (SC3 on target alone) and ConcatenateCluster (SC3 on concatenated data) on at least the mNP/mNFa split
- Under MAGIC imputation the paper reports 919/1000 mNP/mNFa, 944/1000 pPEP, and 831/1000 pNF successes for TransferCluster — a benchmark range for other pre-processing conditions
- The batch-corrected data must retain the same 4402 shared genes and the same cell counts (314 Hockley, 622 Usoskin) as the original pre-processed matrices
- All three methods are run under identical cluster count k=7 so differences in the table are attributable to pre-processing, not to k

## Limitations

- Only the set of genes present in both datasets (4402 of the original) can be used; the paper states that if Xsrc and Xtrg share only a small set of transcripts, a loss of (probably) vital information is inevitable
- No software version numbers were stated for SC3, Seurat, or MAGIC, which limits exact reproducibility of the batch-removal step
- SC3 is non-convex, so single runs are unreliable; robustness conclusions require the 1000-replication protocol

## Evidence

- [methods] Here, we applied Seurat batch effect removal56 to combine the Hockley and the Usoskin data: "Here, we applied Seurat batch effect removal56 to combine the Hockley and the Usoskin data"
- [discussion] The performance of the proposed method can be further improved by applying appropriate pre-processing batch effect removal or imputation before clustering.: "The performance of the proposed method can be further improved by applying appropriate pre-processing batch effect removal or imputation before clustering."
- [readme] Robustness experiments on Hockley data with usoskin as source data and pre-processing through Seurats batch effect removal method: "Robustness experiments on Hockley data with usoskin as source data and pre-processing through Seurats batch effect removal method"
- [methods] MAGIC78, a widely used method for imputing missing values to overcome zero-inflation in scRNA-Seq data, was applied to both datasets: "MAGIC78, a widely used method for imputing missing values to overcome zero-inflation in scRNA-Seq data, was applied to both datasets"

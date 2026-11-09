---
name: lrp-based-differential-gene-ranking
description: Use when scMDC (or a similar pretrained deep clustering model) has produced
  cluster assignments on single-cell multi-omics data and you need per-cluster marker
  genes or pathway-level enrichment from a directed gene rank.
when_to_use_negative:
- You need classical DE p-values between labeled groups rather than a model-based
  minimal-perturbation ranking for GSEA input
- No trained deep clustering model (e.g., scMDC weights) is available for the dataset,
  so no group assignment exists to perturb
- Input is raw unfiltered counts that have not gone through Scanpy preprocessing and
  the scMDC clustering stage
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scMDC
  role: Provides the pretrained multimodal deep clustering model whose cluster assignments
    the LRP perturbation analysis operates on
- name: Pytorch
  role: Framework in which the scMDC autoencoder and run_LRP.py minimal-perturbation
    differential analysis are implemented
- name: Scanpy
  role: Preprocesses and normalizes mRNA and ADT data before model training and LRP
    analysis
- name: fgsea
  role: Consumes the directed gene rank per cluster to compute Hallmark GSEA NES and
    adjusted p-values
- name: msigdbr
  role: Supplies the Hallmark gene sets used by fgsea for enrichment testing
- name: h5py
  role: Loads the h5 input containing RNA counts X1 and ADT counts X2 in Python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:fdab3065fa110def2e57422aed9f1bad6ce4b553f3ca182a52a9352a82238381
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pytorch
  - Scanpy
  - fgsea
  - h5py
  - msigdbr
  - scMDC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lrp-based-differential-gene-ranking

## Summary

Derive a directed gene ranking per cell cluster using minimal-perturbation deep differential analysis (run_LRP.py) on the trained scMDC model, then use the rank for downstream GSEA (fgsea with msigdbr Hallmark sets). This converts unsupervised cluster assignments into interpretable, direction-aware gene importances.

## When to use

Use when scMDC (or a similar pretrained deep clustering model) has produced cluster assignments on single-cell multi-omics data and you need per-cluster marker genes or pathway-level enrichment from a directed gene rank.

## When NOT to use

- You need classical DE p-values between labeled groups rather than a model-based minimal-perturbation ranking for GSEA input
- No trained deep clustering model (e.g., scMDC weights) is available for the dataset, so no group assignment exists to perturb
- Input is raw unfiltered counts that have not gone through Scanpy preprocessing and the scMDC clustering stage

## Inputs

- Preprocessed single-cell multi-omics data in h5 format with RNA counts (X1) and ADT counts (X2), Scanpy-normalized
- Well-trained scMDC model weights (--ae_weights)
- scMDC cluster assignments (per-cell cluster labels)
- Hallmark gene sets from msigdbr (v7.4.1) in R

## Outputs

- Per-cluster directed gene rank (gene importances with log-fold-change direction)
- Per-cluster GSEA results with NES and adjusted p-values (fgsea_hallmark_gsea_results.csv)
- Fig. 8-style dot plots of enriched Hallmark pathways per cluster and a pathway heatmap

## How to apply

Load the preprocessed multimodal h5 data (e.g., BMNC GSE128639/bmcite with RNA counts X1 and ADT counts X2 prepared with Scanpy) together with the well-trained scMDC model weights (--ae_weights) and cluster assignments. Run the LRP deep differential analysis (run_LRP.py) per cluster against the rest: the algorithm finds the minimal perturbation that alters the group assignment from a source group (s) to the target group(s) (t), yielding gene importances per cluster. Combine the importances with log-fold-change direction vectors to build a directed gene rank per cluster. In R, feed each directed rank to fgsea (v1.19.4) with Hallmark gene sets from msigdbr (v7.4.1), collecting NES and adjusted p-values per cluster into a results table (e.g., fgsea_hallmark_gsea_results.csv).

## Related tools

- **scMDC** (Provides the pretrained multimodal deep clustering model whose cluster assignments the LRP perturbation analysis operates on)
- **Pytorch** (Framework in which the scMDC autoencoder and run_LRP.py minimal-perturbation differential analysis are implemented)
- **Scanpy** (Preprocesses and normalizes mRNA and ADT data before model training and LRP analysis)
- **fgsea** (Consumes the directed gene rank per cluster to compute Hallmark GSEA NES and adjusted p-values)
- **msigdbr** (Supplies the Hallmark gene sets used by fgsea for enrichment testing)
- **h5py** (Loads the h5 input containing RNA counts X1 and ADT counts X2 in Python)

## Examples

```
python run_LRP.py --ae_weights scmdc_bmnc_weights.h5 --data bmnc.h5 --cluster 4
```

## Evaluation signals

- fgsea_hallmark_gsea_results.csv exists and contains per-cluster NES and padj fields
- Dot plots and heatmap render enriched Hallmark pathways consistent with the published Fig. 8
- In the BMNC Hallmark GSEA, the complement system pathway has the highest enrichment score in CD14 monocytes
- Each cluster yields a non-empty directed gene rank with both gene importances and log-fold-change direction vectors

## Limitations

- The number of clusters K must be specified by the user before clustering; no automatic estimation is provided
- All experiments in the study were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- The ranking is derived from the model's minimal-perturbation criterion rather than a conventional statistical DE test, so interpretability depends on the trained scMDC model quality

## Evidence

- [discussion] Marker gene detection via minimal-perturbation deep differential expression ranking: "this algorithm will find the minimal perturbation that alters the group assignment from a source group (s) to the target group(s) (t)"
- [discussion] GSEA performed with fgsea and msigdbr on the directed gene rank: "Based on the directed gene rank, gene set enrichment analysis (GSEA) is performed by the package fgsea (v1.19.4) and msigdbr (v7.4.1) in R"
- [results] In the Hallmark GSEA of the BMNC clusters, the complement system pathway has the highest enrichment score in CD14 monocytes: [evidence span withheld — anchor status: not_found]
- [methods] Input data preparation in h5 format: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"

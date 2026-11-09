---
name: scenic-coexpression-module-detection
description: Use when you have a normalized, clustered scRNA-seq dataset (e.g. Louvain
  clusters in Seurat) and need to identify cluster-specific regulons and transcription
  factor activation patterns via GRN reconstruction.
when_to_use_negative:
- The input is scATAC-seq data rather than scRNA-seq — regulon inference here is performed
  on scRNA-seq clusters via AUCell AUC scores
- The dataset is not human or mouse — SCALA's article states 'the analysis of only
  human and mouse datasets is currently supported'
- Only raw (unnormalized, unclustered) count matrices are available — the loom export
  requires a normalized, scaled expression matrix with pre-computed Louvain cluster
  annotations
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Host application whose GRN analysis tab imports auc.*.loom and visualizes
    cluster-level regulon activation
  repo: https://github.com/PavlopoulosLab/SCALA
- name: SCENIC
  role: Original method for inferring gene regulatory networks and cell types from
    scRNA-seq; deprecated in favor of pySCENIC
  repo: https://github.com/aertslab/SCENIC
- name: pySCENIC
  role: Python implementation of the SCENIC pipeline executed via the pyscenic_local.py
    wrapper with default parameters
  repo: https://github.com/aertslab/pySCENIC
- name: SCopeLoomR
  role: R package used to export the normalized expression matrix and Seurat/Louvain
    clusters into a .loom file compatible with the pyscenic wrapper
  repo: https://github.com/aertslab/SCopeLoomR
- name: Seurat
  role: Source of the normalized, scaled expression matrix and Louvain cluster annotations
    exported to loom
- name: AUCell
  role: Computes the per-cell regulon activity AUC scores from which average AUC and
    per-cluster RSS are derived
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:272c813599593980bd84862cad686084f16606c2b1048b52ee1b42f0219f7e77
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - AUCell
  - R
  - SCALA
  - SCENIC
  - SCopeLoomR
  - Seurat
  - pySCENIC
  - pyscenic_local.py wrapper
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# SCENIC coexpression module detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers gene regulatory networks (regulons) from scRNA-seq clusters by running the SCENIC/pySCENIC workflow via SCALA's pyscenic_local.py wrapper, then scoring regulon activity (AUCell AUC) per cell and per cluster. In SCALA's hTNFtg scRNA-seq use case this detected 133 regulons whose activation was preferential at the cluster level.

## When to use

Use when you have a normalized, clustered scRNA-seq dataset (e.g. Louvain clusters in Seurat) and need to identify cluster-specific regulons and transcription factor activation patterns via GRN reconstruction.

## When NOT to use

- The input is scATAC-seq data rather than scRNA-seq — regulon inference here is performed on scRNA-seq clusters via AUCell AUC scores
- The dataset is not human or mouse — SCALA's article states 'the analysis of only human and mouse datasets is currently supported'
- Only raw (unnormalized, unclustered) count matrices are available — the loom export requires a normalized, scaled expression matrix with pre-computed Louvain cluster annotations

## Inputs

- Normalized (log-transformed, scaled) scRNA-seq expression matrix with cluster annotations (e.g. PBMC3k exported via SCopeLoomR as .loom)
- cistarget feather ranking databases for hg38/hg19 (or mm10) from resources.aertslab.org
- motif2tf annotation tbl file (motifs-v9-nr, e.g. motifs-v9-nr.hgnc-m0.001-o0.0.tbl)
- Curated transcription factor list (annotation folder, passed via -a)

## Outputs

- adjacencies.*.tsv (GRN coexpression module / adjacency tables)
- nes.score.*.csv (motif enrichment / NES scores)
- auc.*.loom (AUCell regulon activity scores importable into SCALA)
- regulon_auc_heatmap.png (heatmap of z-score-scaled AUC values for top regulons per cluster)

## How to apply

Export the normalized (log-transformed, scaled) expression matrix with Louvain cluster annotations into a .loom file using the SCopeLoomR R package, as required by the pyscenic wrapper. Run the pyscenic_local.py wrapper (Python >= 3.7, preferably in a conda environment with pip-installed pyscenic) with default parameters, passing the loom file (-l), hg38/hg19 cistarget feather ranking databases, the motifs-v9-nr.hgnc-m0.001-o0.0.tbl motif2tf annotation file, and the curated human TF list (-a annotation folder); this yields adjacencies.*.tsv, nes.score.*.csv and auc.*.loom in the output folder. Upload the resulting auc.*.loom into SCALA's scRNA-seq Gene Regulatory Network analysis tab to import the inferred regulons. Compute average AUC values per cell and Regulon Specificity Score (RSS) per cluster from the AUCell AUC scores to identify cluster-specific regulons, then render a heatmap of z-score-scaled AUC values for the top regulons per cluster.

## Related tools

- **SCALA** (Host application whose GRN analysis tab imports auc.*.loom and visualizes cluster-level regulon activation) — https://github.com/PavlopoulosLab/SCALA
- **SCENIC** (Original method for inferring gene regulatory networks and cell types from scRNA-seq; deprecated in favor of pySCENIC) — https://github.com/aertslab/SCENIC
- **pySCENIC** (Python implementation of the SCENIC pipeline executed via the pyscenic_local.py wrapper with default parameters) — https://github.com/aertslab/pySCENIC
- **SCopeLoomR** (R package used to export the normalized expression matrix and Seurat/Louvain clusters into a .loom file compatible with the pyscenic wrapper) — https://github.com/aertslab/SCopeLoomR
- **Seurat** (Source of the normalized, scaled expression matrix and Louvain cluster annotations exported to loom)
- **AUCell** (Computes the per-cell regulon activity AUC scores from which average AUC and per-cluster RSS are derived)

## Examples

```
python pyscenic_local.py -l pbmc3k.loom --feather hg38__refseq-r80__10kb_up_and_down_tss.mc9nr.feather --motifs motifs-v9-nr.hgnc-m0.001-o0.0.tbl -a annotations/ -o output/
```

## Evaluation signals

- The three pyscenic output families exist in the output folder: adjacencies.*.tsv, nes.score.*.csv and auc.*.loom
- The auc.*.loom uploads successfully into SCALA's scRNA-seq Gene Regulatory Network analysis tab and imports the inferred regulons
- Per-cluster RSS values identify a distinct set of cluster-specific regulons (e.g. 133 regulons in the hTNFtg use case)
- regulon_auc_heatmap.png exists, is a valid image, and shows z-score-scaled AUC values (high yellow, low purple) with cluster-specific patterns consistent with a Fig. 2G-style output
- Hierarchical clustering of z-scored AUC values across clusters reproduces biologically meaningful separations, e.g. sublining clusters separating from intermediate and lining clusters in the hTNFtg data

## Limitations

- SCALA's online version restricts dataset size to < 2 GB and 2 CPU threads per session, and some steps cannot be completed for datasets larger than 50,000 cells — prefer the desktop version for large datasets
- Only human and mouse datasets are currently supported for this analysis
- The original R SCENIC is deprecated; pySCENIC is the maintained implementation
- SCALA requires Python 3.7 or newer (preferably in a conda environment) and a specific ordering of R library installations for the pipeline to run

## Evidence

- [readme] pyscenic is a lightning-fast python implementation of the SCENIC pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] SCopeLoomR creates loom files with SCENIC regulons: [evidence span withheld — anchor status: external_doc]
- [other] Workflow and outputs of the pyscenic wrapper: [evidence span withheld — anchor status: not_found]
- [other] Validation of regulon heatmap output: [evidence span withheld — anchor status: not_found]

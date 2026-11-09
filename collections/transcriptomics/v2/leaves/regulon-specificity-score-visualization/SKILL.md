---
name: regulon-specificity-score-visualization
description: Use when a pyscenic auc.*.loom (AUCell regulon AUC scores over clustered
  scRNA-seq cells) is available and you need to identify and visualize which regulons
  are preferentially active per cluster.
when_to_use_negative:
- The input is raw counts without normalization, scaling, and pre-computed Louvain
  clusters — the pyscenic wrapper expects an exported loom with normalized expression
  and cluster annotations.
- The dataset is not human or mouse — SCALA currently supports analysis of only human
  and mouse datasets.
- The goal is scATAC-seq regulon inference rather than scRNA-seq regulon activity
  scoring.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: SCALA
  role: Web/desktop application whose scRNA-seq Gene Regulatory Network analysis tab
    imports the auc.*.loom and visualizes regulon activity per cluster
  repo: https://github.com/PavlopoulosLab/SCALA
- name: SCopeLoomR
  role: R package used to export the normalized expression matrix and Louvain clusters
    into the loom file required by the pyscenic wrapper
  repo: https://github.com/aertslab/SCopeLoomR
- name: pySCENIC
  role: Python implementation of the SCENIC pipeline run via the pyscenic_local.py
    wrapper (Python >= 3.7, conda environment) to infer regulons and produce auc.*.loom
  repo: https://github.com/aertslab/SCENIC
- name: AUCell
  role: Bioconductor package supplying the AUC scores from which per-cell averages
    and per-cluster RSS are computed
- name: Seurat
  role: Source of the normalized, scaled expression matrix and Louvain cluster annotations
    prior to loom export
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:56b0b7a9e3a327a3cd38c8c8dc935c523d8a462c532801172fb0c058eaa8aa91
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
  - SCopeLoomR
  - Seurat
  - pySCENIC
  - pyscenic_local.py wrapper
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# regulon-specificity-score-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Regulon Specificity Scores (RSS) and per-cell average AUCell AUC values from a pyscenic-inferred regulon AUC loom and renders a z-scored regulon-activity heatmap, exposing cluster-specific regulons in the SCALA scRNA-seq GRN analysis module.

## When to use

Use when a pyscenic auc.*.loom (AUCell regulon AUC scores over clustered scRNA-seq cells) is available and you need to identify and visualize which regulons are preferentially active per cluster.

## When NOT to use

- The input is raw counts without normalization, scaling, and pre-computed Louvain clusters — the pyscenic wrapper expects an exported loom with normalized expression and cluster annotations.
- The dataset is not human or mouse — SCALA currently supports analysis of only human and mouse datasets.
- The goal is scATAC-seq regulon inference rather than scRNA-seq regulon activity scoring.

## Inputs

- Normalized (log-transformed, scaled) scRNA-seq expression matrix with Louvain cluster annotations (e.g., PBMC3k), exported as a .loom file via SCopeLoomR
- hg38/hg19 cistarget feather ranking databases (resources.aertslab.org)
- motif2tf annotation file (motifs-v9-nr.hgnc-m0.001-o0.0.tbl)
- Curated human TF list (annotation folder passed via -a)
- auc.*.loom file with AUCell regulon AUC scores produced by pyscenic

## Outputs

- adjacencies.*.tsv (GRN adjacencies from pyscenic)
- nes.score.*.csv (motif enrichment / NES scores)
- auc.*.loom (AUCell regulon activity matrix importable into SCALA)
- Per-cluster RSS values and per-cell average AUC values identifying cluster-specific regulons
- regulon_auc_heatmap.png: heatmap of z-scored AUC values for top regulons per cluster

## How to apply

Export the normalized (log-transformed, scaled) expression matrix plus Louvain cluster annotations to a loom file with SCopeLoomR, then run the pyscenic_local.py wrapper (Python >= 3.7 in a conda env, pip install pyscenic) against the hg38/hg19 cistarget feather ranking databases, the motifs-v9-nr.hgnc-m0.001-o0.0.tbl motif2tf file, and the curated human TF list, yielding adjacencies.*.tsv, nes.score.*.csv and auc.*.loom. Upload the auc.*.loom into SCALA's scRNA-seq Gene Regulatory Network analysis tab to import the inferred regulons. Compute average AUC values per cell and the Regulon Specificity Score (RSS) per cluster from the AUCell AUC scores to identify cluster-specific regulons (in the hTNFtg scRNA-seq use case, 133 regulons were detected). Finally render a heatmap of z-score-scaled AUC values for the top regulons per cluster, in the Fig. 2G style where high scaled AUC is yellow and low is purple.

## Related tools

- **SCALA** (Web/desktop application whose scRNA-seq Gene Regulatory Network analysis tab imports the auc.*.loom and visualizes regulon activity per cluster) — https://github.com/PavlopoulosLab/SCALA
- **SCopeLoomR** (R package used to export the normalized expression matrix and Louvain clusters into the loom file required by the pyscenic wrapper) — https://github.com/aertslab/SCopeLoomR
- **pySCENIC** (Python implementation of the SCENIC pipeline run via the pyscenic_local.py wrapper (Python >= 3.7, conda environment) to infer regulons and produce auc.*.loom) — https://github.com/aertslab/SCENIC
- **AUCell** (Bioconductor package supplying the AUC scores from which per-cell averages and per-cluster RSS are computed)
- **Seurat** (Source of the normalized, scaled expression matrix and Louvain cluster annotations prior to loom export)

## Examples

```
conda create -n pyscenic python=3.7 && pip install pyscenic && python pyscenic_local.py -l pbmc3k.loom -d hg38_rankings.feather -m motifs-v9-nr.hgnc-m0.001-o0.0.tbl -a annotations/
```

## Evaluation signals

- regulon_auc_heatmap.png exists in the output folder and is a valid image file
- Heatmap displays z-score-scaled AUC values with high values rendered yellow and low values purple (Fig. 2G-style)
- Heatmap shows cluster-specific patterns of regulon activity consistent with RSS-per-cluster ranking of top regulons
- pyscenic run produced the full output set: adjacencies.*.tsv, nes.score.*.csv and auc.*.loom
- auc.*.loom imports successfully into SCALA's scRNA-seq Gene Regulatory Network analysis tab (as in the hTNFtg use case with 133 detected regulons)

## Limitations

- The online SCALA version restricts dataset size to < 2 GB and allocates no more than 2 CPU threads per session; datasets larger than 50,000 cells may not complete some steps and should use the desktop version.
- Only human and mouse datasets are currently supported.
- The original aertslab/SCENIC repository is deprecated in favor of pySCENIC.
- SCALA requires a patched Seurat (for PHATE support) and specific package install order; the SCENIC workflow additionally needs Python 3.7+ in a conda environment with pyscenic installed.

## Evidence

- [other] Compute average AUC values per cell and Regulon Specificity Score (RSS) per cluster from the AUCell AUC scores to identify cluster-specific regulons.: [evidence span withheld — anchor status: not_found]
- [results] SCALA's GRN module detects regulons whose activation is preferential at the cluster level; in the hTNFtg scRNA-seq use case it identified 133 regulons: [evidence span withheld — anchor status: not_found]

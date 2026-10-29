---
name: single-cell-gene-regulatory-network-inference
description: Use when scRNA-seq data has been normalized, scaled and Louvain-clustered
  and the goal is to identify transcription-factor regulons whose activation is cluster-specific,
  e.g. reproducible Fig. 2G-style regulon activity heatmaps.
when_to_use_negative:
- Input is scATAC-seq or unclustered raw count data — the workflow requires a normalized,
  scaled, cluster-annotated scRNA-seq matrix exported to loom
- Species other than human or mouse — SCALA notes that 'in both modalities, the analysis
  of only human and mouse datasets is currently supported'
- Datasets larger than ~50,000 cells on the online SCALA web server, where some steps
  cannot be completed
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCALA
  role: Web/desktop application whose scRNA-seq Gene Regulatory Network analysis tab
    imports the auc.*.loom regulons and visualizes cluster-level regulon activity
- name: SCENIC
  role: Underlying GRN inference and cell-type clustering method (R version; deprecated
    in favor of pySCENIC)
  repo: https://github.com/aertslab/SCENIC
- name: SCopeLoomR
  role: R package used to create generic .loom files and extend them with SCENIC regulons
    and Seurat clusters for the pyscenic wrapper
  repo: https://github.com/aertslab/SCopeLoomR
- name: Seurat
  role: Provides the normalized, scaled expression matrix and Louvain cluster annotations
    exported into the loom file
  repo: https://github.com/scottgigante/seurat
- name: pySCENIC
  role: Python implementation of the SCENIC pipeline executed via the pyscenic_local.py
    wrapper (Python >= 3.7 in a conda environment)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:6e86f66f262a76c8201af0f9142ca4fd9efb2a28972994ed0c960d99e5ca645b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/aertslab/SCENIC
  tools:
  - R
  - SCALA
  - SCENIC
  - SCopeLoomR
  - Seurat
  - pySCENIC
  - pyscenic_local.py wrapper
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-gene-regulatory-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and evaluate gene regulatory networks (GRNs) from scRNA-seq clusters using the SCENIC/pySCENIC workflow behind SCALA, producing regulons and cluster-level AUC activity scores that can be visualized as z-scored heatmaps.

## When to use

Use when scRNA-seq data has been normalized, scaled and Louvain-clustered and the goal is to identify transcription-factor regulons whose activation is cluster-specific, e.g. reproducible Fig. 2G-style regulon activity heatmaps.

## When NOT to use

- Input is scATAC-seq or unclustered raw count data — the workflow requires a normalized, scaled, cluster-annotated scRNA-seq matrix exported to loom
- Species other than human or mouse — SCALA notes that 'in both modalities, the analysis of only human and mouse datasets is currently supported'
- Datasets larger than ~50,000 cells on the online SCALA web server, where some steps cannot be completed

## Inputs

- Normalized (log-transformed, scaled) scRNA-seq expression matrix with Louvain cluster annotations (e.g. PBMC3k exported as .loom via SCopeLoomR)
- cistarget ranking databases as feather files (hg38/hg19, e.g. from resources.aertslab.org)
- motif2tf annotation tbl file (motifs-v9-nr.hgnc-m0.001-o0.0.tbl)
- Curated human transcription factor list

## Outputs

- adjacencies.*.tsv (GRN co-expression module adjacencies)
- nes.score.*.csv (motif enrichment / regulon specificity scores)
- auc.*.loom (AUCell regulon activity scores importable into SCALA)
- regulon_auc_heatmap.png (heatmap of z-score-scaled AUC values for top regulons per cluster)
- Regulon Specificity Score (RSS) per cluster

## How to apply

Export the normalized (log-transformed, scaled) expression matrix together with Louvain cluster annotations into a .loom file with the SCopeLoomR R package, since the pyscenic wrapper requires the loom format. Run the pyscenic_local.py wrapper in a conda environment (Python >= 3.7, pip install pyscenic) with default parameters, passing the loom file (-l), the hg38/hg19 cistarget feather ranking databases, the motifs-v9-nr.hgnc-m0.001-o0.0.tbl motif2tf annotation file, and the curated human TF list (-a annotation folder); this yields adjacencies.*.tsv, nes.score.*.csv and auc.*.loom outputs. Upload the resulting auc.*.loom into SCALA's scRNA-seq GRN analysis tab to import inferred regulons. Compute average AUC values per cell and Regulon Specificity Score (RSS) per cluster from the AUCell AUC scores to flag cluster-specific regulons, then render a heatmap of z-score-scaled AUC values for the top regulons per cluster. In the SCALA use case this identified 133 regulons in hTNFtg scRNA-seq data, with hierarchical clustering of regulon activity separating sublining from intermediate and lining clusters.

## Related tools

- **SCALA** (Web/desktop application whose scRNA-seq Gene Regulatory Network analysis tab imports the auc.*.loom regulons and visualizes cluster-level regulon activity)
- **SCENIC** (Underlying GRN inference and cell-type clustering method (R version; deprecated in favor of pySCENIC)) — https://github.com/aertslab/SCENIC
- **SCopeLoomR** (R package used to create generic .loom files and extend them with SCENIC regulons and Seurat clusters for the pyscenic wrapper) — https://github.com/aertslab/SCopeLoomR
- **Seurat** (Provides the normalized, scaled expression matrix and Louvain cluster annotations exported into the loom file) — https://github.com/scottgigante/seurat
- **pySCENIC** (Python implementation of the SCENIC pipeline executed via the pyscenic_local.py wrapper (Python >= 3.7 in a conda environment))

## Examples

```
conda create -n pyscenic python=3.7; pip install pyscenic; python pyscenic_local.py -l pbmc3k.loom -d hg38-ranking.feather -m motifs-v9-nr.hgnc-m0.001-o0.0.tbl -a annotation_folder -o out/
```

## Evaluation signals

- regulon_auc_heatmap.png exists and is a valid image
- Heatmap displays cluster-specific patterns of regulon activity (scaled AUC z-scores, high values yellow, low values purple), consistent with the Fig. 2G-style output
- auc.*.loom, adjacencies.*.tsv and nes.score.*.csv are all produced in the pyscenic output folder
- auc.*.loom imports successfully into SCALA's scRNA-seq GRN analysis tab
- Average AUC per cell and RSS per cluster are computed and highlight regulons preferentially active at the cluster level (e.g. 133 regulons in the hTNFtg case)

## Limitations

- Only human and mouse datasets are currently supported for GRN analysis in SCALA
- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session; datasets > 50,000 cells should use the desktop version
- Original SCENIC R implementation is deprecated in favor of pySCENIC; SCALA's pyscenic setup requires specific cistarget feather databases and motif2tf tbl annotations matching hg38/hg19 or mm9/mm10

## Evidence

- [results] SCALA's GRN module detects regulons whose activation is preferential at the cluster level: [evidence span withheld — anchor status: not_found]
- [readme] SCENIC is a computational method to infer Gene Regulatory Networks and cell types from single-cell RNA-seq data: [evidence span withheld — anchor status: not_found]
- [readme] An R package (compatible with SCope) to create generic .loom files and extend them with other data e.g.: SCENIC regulons, Seurat clusters and markers: [evidence span withheld — anchor status: external_doc]

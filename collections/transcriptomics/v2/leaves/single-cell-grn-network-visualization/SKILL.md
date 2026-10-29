---
name: single-cell-grn-network-visualization
description: Use when you have pySCENIC/SCENIC GRN outputs from plant single-cell
  data and need to visualize which regulon TFs characterize specific cell clusters.
when_to_use_negative:
- Input is raw unprocessed scRNA-seq counts without prior Seurat clustering and cell-type
  annotation — GRN visualization requires an annotated object
- Data are from non-plant organisms with no species-specific cisTarget database available
  in the scPlant auxiliary bundle
- The goal is differential expression or trajectory inference rather than regulon/GRN
  interpretation
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: R framework providing ras_exp_hmp, topRegulons and SpecificityRank visualization
    functions and the RunGRN.sh GRN wrapper
  repo: https://github.com/compbioNJU/scPlant
- name: pySCENIC
  role: Python implementation of SCENIC invoked by RunGRN.sh to infer TFs, regulons
    and gene regulatory networks from single-cell RNA-seq
  repo: https://github.com/aertslab/pySCENIC
- name: Seurat
  role: Provides the annotated Seurat object (SeuratObj.rds) that is the input to
    GRN construction and cluster-level visualization
- name: SCopeLoomR
  role: R package used in the RunGRN.sh workflow to handle pySCENIC loom outputs (pyscenicOutput.loom)
  repo: https://github.com/aertslab/SCopeLoomR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:e641d907ed37d9aa6d22f9c1c30c789f174ac406451aa3dbc58fda99bdc2c496
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - SCopeLoomR
  - Seurat
  - pySCENIC
  - scPlant
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-grn-network-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize cell-type-specific gene regulatory networks (GRNs) inferred with pySCENIC/SCENIC inside the scPlant R framework, using regulon activity matrices, regulon specificity scores, and TF-target links to identify cell-type-characterizing regulon TFs (e.g., PIF5 for photosynthetic cells in the Arabidopsis root atlas).

## When to use

Use when you have pySCENIC/SCENIC GRN outputs from plant single-cell data and need to visualize which regulon TFs characterize specific cell clusters.

## When NOT to use

- Input is raw unprocessed scRNA-seq counts without prior Seurat clustering and cell-type annotation — GRN visualization requires an annotated object
- Data are from non-plant organisms with no species-specific cisTarget database available in the scPlant auxiliary bundle
- The goal is differential expression or trajectory inference rather than regulon/GRN interpretation

## Inputs

- Annotated Seurat object (.rds) from a plant single-cell atlas (e.g., Zhang et al. 2019 Arabidopsis root scRNA-seq)
- Species-specific cisTarget databases (e.g., Arabidopsis TAIR10 from scPlantData.zip)
- SCENIC/pySCENIC outputs: rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt

## Outputs

- Regulon activity / TF expression heatmap per cluster (ras_exp_hmp)
- Network plot of top regulons per cluster ranked by regulon specificity score (topRegulons)
- Dot plot of top regulons for a chosen cluster (SpecificityRank)
- Ranked list of top regulon TFs per cell cluster (e.g., PIF5 first for photosynthetic cells)

## How to apply

Run the scPlant GRN wrapper (RunGRN.sh) on an annotated Seurat object with the species-specific cisTarget databases (e.g., Arabidopsis thaliana TAIR10 from the scPlantData.zip Figshare bundle), which invokes pySCENIC 0.12.1 and post-processing to produce pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt and binary_mtx.txt. Load rasMat.rds (mean regulon activity per cluster), rssMat.rds (regulon specificity scores) and tf_target.rds (TF-target links) into R with library(scPlant). Use ras_exp_hmp to render a heatmap of mean regulon activity and TF expression per cluster, topRegulons to build the network of top regulons per cluster ranked by RSS, and SpecificityRank to draw a dot plot of the top regulons for a chosen cluster. Validate by confirming the expected regulon TF ranks first for the target cluster (e.g., PIF5 for photosynthetic cells, matching Figure 6A-6C of the paper).

## Related tools

- **scPlant** (R framework providing ras_exp_hmp, topRegulons and SpecificityRank visualization functions and the RunGRN.sh GRN wrapper) — https://github.com/compbioNJU/scPlant
- **pySCENIC** (Python implementation of SCENIC invoked by RunGRN.sh to infer TFs, regulons and gene regulatory networks from single-cell RNA-seq) — https://github.com/aertslab/pySCENIC
- **Seurat** (Provides the annotated Seurat object (SeuratObj.rds) that is the input to GRN construction and cluster-level visualization)
- **SCopeLoomR** (R package used in the RunGRN.sh workflow to handle pySCENIC loom outputs (pyscenicOutput.loom)) — https://github.com/aertslab/SCopeLoomR

## Examples

```
bash RunGRN.sh -f SeuratObj.rds -s At -t 2 -d cisTarget_databases -o output
```

## Evaluation signals

- All declared output files exist in the output directory: pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt and binary_mtx.txt
- The RSS-based topRegulons network ranks PIF5 as the top regulon TF for the photosynthetic cell cluster
- The SpecificityRank dot plot for the photosynthetic cluster places PIF5 first, matching Figure 6A-6C of the paper
- ras_exp_hmp heatmap shows concordance between regulon activity and TF expression per cluster
- Conda environment resolves pySCENIC version 0.12.1 as specified in the workflow

## Limitations

- GRN construction depends on precomputed or self-built species-specific cisTarget databases; organisms without such databases (e.g., built for human/mouse/fly only) require constructing databases with create_cisTarget_databases, which is memory-intensive
- Ranking confirmation is expert/visual against published figures, not fully automated
- Creating cisTarget databases can require roughly 3 times the RAM of the actual database size

## Evidence

- [other] Load rasMat.rds, rssMat.rds and tf_target.rds into R with library(scPlant) and visualize the GRNs using ras_exp_hmp (mean regulon activity and TF expression heatmap per cluster): [evidence span withheld — anchor status: not_found]
- [results] SCENIC identified PIF5 as top regulon TF for photosynthetic cells: [evidence span withheld — anchor status: not_found]
- [readme] pySCENIC enables biologists to infer transcription factors, gene regulatory networks and cell types from single-cell RNA-seq data: [evidence span withheld — anchor status: external_doc]

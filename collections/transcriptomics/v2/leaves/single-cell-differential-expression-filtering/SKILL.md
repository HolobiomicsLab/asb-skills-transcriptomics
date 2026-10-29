---
name: single-cell-differential-expression-filtering
description: Use when a single-cell RNA-seq dataset has been clustered (e.g., with
  scPlant/Seurat) and you need cluster-specific marker genes as input for promoter
  motif-enrichment analysis such as PMET on SCENIC regulons.
when_to_use_negative:
- You have no cluster/cell-type assignments yet — run scPlant/Seurat clustering first,
  since the filters define cluster-specific expression via pct.1 vs pct.2
- You need genome-wide differential expression between conditions rather than per-cluster
  marker genes for promoter motif enrichment
- Your analysis is bulk RNA-seq deconvolution — scPlant's decomposeBulk, not marker
  filtering, is the appropriate module
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPlant
  role: Framework whose Seurat-based pipeline supplies the clustered object and from
    which cluster-specific DEGs are extracted as the PMET gene input
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Provides the marker-detection methods (pct.1, pct.2, p_val, avg_logFC columns)
    whose filters define the cluster-specific gene set
- name: PMET
  role: Downstream Paired Motif Enrichment Test that consumes the filtered DEG.txt
    as the -m gene input file
  repo: https://github.com/kate-wa/PMET-software-python
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:275ec5857494642cf69fb8b1d7526330cbfba585e4498ce210866c93c4bdac0f
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
  - PMET
  - Seurat
  - scPlant
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-differential-expression-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract cluster-specific differentially expressed marker genes from a Seurat-analyzed single-cell atlas (e.g., the Arabidopsis root atlas of Zhang et al., 2019) using Seurat marker filters, and format them as a tab-delimited DEG.txt gene input file for downstream motif-enrichment tools such as PMET in the scPlant workflow.

## When to use

Use when a single-cell RNA-seq dataset has been clustered (e.g., with scPlant/Seurat) and you need cluster-specific marker genes as input for promoter motif-enrichment analysis such as PMET on SCENIC regulons.

## When NOT to use

- You have no cluster/cell-type assignments yet — run scPlant/Seurat clustering first, since the filters define cluster-specific expression via pct.1 vs pct.2
- You need genome-wide differential expression between conditions rather than per-cluster marker genes for promoter motif enrichment
- Your analysis is bulk RNA-seq deconvolution — scPlant's decomposeBulk, not marker filtering, is the appropriate module

## Inputs

- Clustered Seurat object from a plant scRNA-seq atlas (e.g., Arabidopsis thaliana root atlas, Zhang et al., 2019)
- Cluster identities/cell-type labels assigned during scPlant/Seurat processing

## Outputs

- DEG.txt: a 2-column tab-delimited file with cluster ID in column 1 and gene name in column 2
- Cluster-specific marker gene table passing pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25, avg_logFC > 0.25

## How to apply

After clustering (e.g., scPlant RunSeurat with resolution = 0.3), extract differentially expressed genes per cluster using the Seurat marker filters pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25 and avg_logFC > 0.25, applied to the Arabidopsis root atlas data. Write the filtered genes to a 2-column tab-delimited file DEG.txt with the cluster ID in column 1 and the gene name in column 2. This DEG file then serves as the gene input (-m) to the Paired Motif Enrichment Test, e.g., 'bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At' with species At for Arabidopsis thaliana (TAIR10). The stringency of the four filters ensures only cluster-enriched, statistically significant markers feed the promoter enrichment, so spurious cluster-agnostic genes do not dilute the motif signal.

## Related tools

- **scPlant** (Framework whose Seurat-based pipeline supplies the clustered object and from which cluster-specific DEGs are extracted as the PMET gene input) — https://github.com/compbioNJU/scPlant
- **Seurat** (Provides the marker-detection methods (pct.1, pct.2, p_val, avg_logFC columns) whose filters define the cluster-specific gene set)
- **PMET** (Downstream Paired Motif Enrichment Test that consumes the filtered DEG.txt as the -m gene input file) — https://github.com/kate-wa/PMET-software-python

## Examples

```
bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At
```

## Evaluation signals

- DEG.txt exists and is a 2-column tab-delimited file with cluster ID in column 1 and gene in column 2
- Every retained gene satisfies all four Seurat filters: pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25, avg_logFC > 0.25
- Markers for known cell types (e.g., photosynthetic cluster 23 in the Arabidopsis root atlas) are biologically plausible for that cluster
- Running RunPMET.sh with the DEG.txt produces output/output.txt containing PMET results for the target cluster (e.g., cluster 23)
- Downstream PMET visualizations (PMEThmp topn=5, triHmp triangle heatmap) reproduce the TCP14/TCP15 and ABI5-ABF motif-pair enrichments for cluster 23 as in the paper's Figure 6G

## Limitations

- Step 2 of the RunPMET.sh script that consumes the DEG input is time-consuming, so expect long runtimes
- Filter thresholds (pct.1 > 0.25, avg_logFC > 0.25) are tuned for this Arabidopsis atlas workflow; other datasets may require recalibration
- PMET requires external dependencies (samtools, bedtools, meme, parallel) and species-specific databases (e.g., TAIR10 genome FASTA and gff3 from EnsemblPlants release 55), so the DEG gene identifiers must match that genome annotation

## Evidence

- [methods] extracting cluster-specific differentially expressed genes with the Seurat marker filters pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25 and avg_logFC > 0.25: [evidence span withheld — anchor status: not_found]
- [methods] writing a 2-column tab-delimited file DEG.txt (cluster ID in column 1, gene in column 2): [evidence span withheld — anchor status: not_found]
- [methods] Run the Paired Motif Enrichment Test with 'bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At': [evidence span withheld — anchor status: not_found]

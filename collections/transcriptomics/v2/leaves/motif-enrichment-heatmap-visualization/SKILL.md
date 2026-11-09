---
name: motif-enrichment-heatmap-visualization
description: Use when you have raw PMET paired-motif enrichment output (output/output.txt)
  from a plant scRNA-seq cluster analysis and need to identify and visualize cluster-specific
  enriched TF motif pairs as heatmaps and network plots.
when_to_use_negative:
- Input is not PMET paired-motif output (e.g. single-motif cisTarget/SCENIC rankings
  or JASPAR/PlantTFDB motif collections alone) — processPMET expects the PMET output
  table format
- You need to run the enrichment test itself rather than visualize it — use RunPMET.sh
  (with samtools, bedtools, meme, parallel dependencies) instead of the visualization
  functions
- The organism is not one of the supported species databases (e.g. not At, Osa, or
  Zma with prepared scPlantData/PMET resources)
edam_operation: http://edamontology.org/operation_0566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scPlant
  role: Provides processPMET, PMEThmp, triHmp and topPairsNet functions for processing
    and visualizing PMET results
  repo: https://github.com/compbioNJU/scPlant
- name: PMET
  role: Paired Motif Enrichment Tool that generates the raw motif-pair enrichment
    table being visualized
- name: Seurat
  role: Used upstream to extract cluster-specific differentially expressed genes that
    serve as PMET gene input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:7a342fbd021b7fca78336a83dafe9769baa0e3ca04a276d5b401c023963f4173
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
  edam_operation: http://edamontology.org/operation_0566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# motif-enrichment-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize paired motif enrichment test (PMET) results from plant single-cell regulon analysis using scPlant's processPMET, PMEThmp, triHmp and topPairsNet functions to reproduce cluster-specific motif-pair enrichments such as the TCP14/TCP15 and ABI5-ABF pairings in the Arabidopsis root atlas photosynthetic cluster.

## When to use

Use when you have raw PMET paired-motif enrichment output (output/output.txt) from a plant scRNA-seq cluster analysis and need to identify and visualize cluster-specific enriched TF motif pairs as heatmaps and network plots.

## When NOT to use

- Input is not PMET paired-motif output (e.g. single-motif cisTarget/SCENIC rankings or JASPAR/PlantTFDB motif collections alone) — processPMET expects the PMET output table format
- You need to run the enrichment test itself rather than visualize it — use RunPMET.sh (with samtools, bedtools, meme, parallel dependencies) instead of the visualization functions
- The organism is not one of the supported species databases (e.g. not At, Osa, or Zma with prepared scPlantData/PMET resources)

## Inputs

- Raw PMET result table output/output.txt (paired motif enrichment per cluster)
- DEG.txt: 2-column tab-delimited file of cluster ID and cluster-specific differentially expressed genes (used upstream by RunPMET.sh)
- scPlantData/PMET/ databases (scPlantData.zip from the scPlant Figshare deposit) with TAIR10 genome FASTA and gff3 from EnsemblPlants release 55

## Outputs

- PMETresult: processed PMET enrichment table from processPMET
- PMEThmp heatmap of top motif pairs per cluster (topn = 5)
- Triangle heatmap (triHmp) for the target cluster (e.g. cluster 23)
- topPairsNet network plot of the top 20 motif pairs for the selected cluster

## How to apply

After running Paired Motif Enrichment Test via 'bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At' on cluster-specific differentially expressed genes (Seurat markers with pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25, avg_logFC > 0.25), load the raw result table output/output.txt and pre-process it with the scPlant function processPMET to obtain PMETresult. Then visualize enrichment with the scPlant heatmap function PMEThmp (topn = 5) across clusters, draw the triangle heatmap for the target cluster with triHmp (e.g. cluster 23, the photosynthetic cluster), and display the top 20 motif pairs of that cluster with topPairsNet. Validation is done by expert review against the paper's published figure: the triangle heatmap for cluster 23 should reproduce the TCP14/TCP15 (light-response TFs) and ABI5-ABF motif-pair enrichments reported in Figure 6G.

## Related tools

- **scPlant** (Provides processPMET, PMEThmp, triHmp and topPairsNet functions for processing and visualizing PMET results) — https://github.com/compbioNJU/scPlant
- **PMET** (Paired Motif Enrichment Tool that generates the raw motif-pair enrichment table being visualized)
- **Seurat** (Used upstream to extract cluster-specific differentially expressed genes that serve as PMET gene input)

## Examples

```
bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At; PMETresult <- processPMET('output/output.txt'); PMEThmp(PMETresult, topn = 5); triHmp(PMETresult, cluster = 23); topPairsNet(PMETresult, cluster = 23)
```

## Evaluation signals

- output/output.txt exists and contains PMET results for the target cluster (cluster 23)
- processPMET runs on output.txt without errors and returns a non-empty PMETresult table
- Triangle heatmap for cluster 23 reproduces the TCP14/TCP15 motif-pair enrichment reported in Figure 6G
- ABI5-ABF motif pairing appears among the enriched pairs in the cluster-23 heatmap
- topPairsNet displays the top 20 motif pairs for the cluster, consistent with the PMEThmp topn = 5 ranking

## Limitations

- Step 2 of RunPMET.sh is time-consuming, so re-running the enrichment on new gene sets is costly before re-visualization
- Validation is by expert review against the published figure rather than an automated metric
- PMET depends on external tools (samtools, bedtools, meme, parallel) and pre-built species databases; missing dependencies or genomes block the pipeline upstream of visualization

## Evidence

- [results] PMET on the photosynthetic cell cluster (cluster 23) identified a highly significant TCP-family motif pairing specific to that cluster (including TCP14 and TCP15, involved in light responses), as well as enriched ABI5-ABF motif pairing.: [evidence span withheld — anchor status: not_found]
- [other] Visualization workflow uses scPlant functions on the PMET output: [evidence span withheld — anchor status: not_found]
- [methods] RunPMET.sh invocation and heatmap parameters: [evidence span withheld — anchor status: not_found]
- [readme] PMET tool description from repository README: [evidence span withheld — anchor status: external_doc]

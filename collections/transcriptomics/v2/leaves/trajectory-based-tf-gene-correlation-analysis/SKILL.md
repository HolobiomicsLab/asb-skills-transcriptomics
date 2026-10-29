---
name: trajectory-based-tf-gene-correlation-analysis
description: Use when single-cell multiome (RNA+ATAC) data contain an ordered cell
  state transition (e.g. naive-to-memory CD4 T cells, SCARA5+ fibroblasts to myofibroblasts)
  and you need TF-gene regulatory relationships along that trajectory.
when_to_use_negative:
- Data lack an ordered biological process (trajectory) — scMEGA's eGRN inference depends
  on correlating TF activity and expression along pseudotime, so static or unstructured
  cell populations cannot support it.
- Input is unimodal (RNA-only or ATAC-only) — the method needs motif accessibility
  plus gene expression from the same or paired cells.
- Goal is generic single-modal GRN inference without enhancer/peak-to-gene links;
  the enhancer-based design requires motif matching and peak accessibility evidence.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scMEGA
  role: Provides AddMotifs, RunChromVAR, SelectTFs, SelectGenes, GetTFGeneCorrelation,
    GetGRN, GRNHeatmap, GRNPlot used to infer enhancer-based eGRNs
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat v5
  role: Underlying single-cell multiome object model and co-embedding/CCA integration
    for trajectory-ordered analysis
  repo: https://github.com/satijalab/seurat
- name: Signac
  role: ATAC-seq processing (peak accessibility, gene annotations via EnsDb.Hsapiens.v86)
    feeding peak-to-gene links
  repo: https://github.com/stuart-lab/signac
- name: ArchR
  role: Approach used for pseudo-time trajectory inference (AddTrajectory) prior to
    TF-gene correlation
- name: igraph
  role: Backend for GRN network construction and visualization (GRNHeatmap/GRNPlot)
- name: ggraph
  role: Graphics layer used with igraph for GRNPlot network plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:156c4dd4003a15ad51a5ddc4ed4874bd749205a6de293f34096f2c1635516ad1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Seurat v5
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# trajectory-based-tf-gene-correlation-analysis

## Summary

Infer enhancer-based gene regulatory networks (eGRNs) from single-cell multiome data by correlating TF activity (chromVAR deviation scores) with gene expression along a pseudotime trajectory, and anchor candidate TFs/genes to motif and peak-to-gene links.

## When to use

Use when single-cell multiome (RNA+ATAC) data contain an ordered cell state transition (e.g. naive-to-memory CD4 T cells, SCARA5+ fibroblasts to myofibroblasts) and you need TF-gene regulatory relationships along that trajectory.

## When NOT to use

- Data lack an ordered biological process (trajectory) — scMEGA's eGRN inference depends on correlating TF activity and expression along pseudotime, so static or unstructured cell populations cannot support it.
- Input is unimodal (RNA-only or ATAC-only) — the method needs motif accessibility plus gene expression from the same or paired cells.
- Goal is generic single-modal GRN inference without enhancer/peak-to-gene links; the enhancer-based design requires motif matching and peak accessibility evidence.

## Inputs

- Seurat/Signac single-cell multiome object (e.g. 10x PBMC multiome, pbmc_granulocyte_sorted_10k) with a trajectory added via AddTrajectory
- JASPAR2020 motif matrix set (PFMs) and BSgenome.Hsapiens.UCSC.hg38 genome
- Peak-to-gene link annotations (e.g. EnsDb.Hsapiens.v86-derived gene annotations)
- Predicted vs true cell-pair assignments for benchmarking (optional)

## Outputs

- TF-gene correlation table along pseudotime (chromVAR TF activity vs RNA expression)
- Assembled eGRN (motif + TF-gene correlation + peak-to-gene links), filtered at correlation > 0.5
- GRNHeatmap/GRNPlot (igraph/ggraph) network visualizations
- Recovery-rate summary of predicted-pair vs true-pair networks (e.g. cd4_tcell_grn_recovery_rates.csv)

## How to apply

First estimate per-cell TF activity by matching JASPAR2020 CORE vertebrate motifs against the genome (AddMotifs with BSgenome.Hsapiens.UCSC.hg38) and computing chromVAR deviation scores (RunChromVAR). Select candidate TFs with SelectTFs using a correlation cutoff of 0.4 and candidate genes with SelectGenes by correlating peak accessibility with gene expression along the trajectory. Compute TF-gene correlations along the pseudotime with GetTFGeneCorrelation (tf.assay = 'chromvar', gene.assay = 'RNA', trajectory.name = 'Trajectory'), then assemble the eGRN with GetGRN by combining motif matches, TF-gene correlations, and peak-to-gene links. Filter network edges at correlation > 0.5 and visualize with GRNHeatmap/GRNPlot (igraph/ggraph). If a pairing-benchmark variant exists (predicted vs true cell pairs), build the network twice and compute recovery rates for TFs, genes, and TF-gene regulations.

## Related tools

- **scMEGA** (Provides AddMotifs, RunChromVAR, SelectTFs, SelectGenes, GetTFGeneCorrelation, GetGRN, GRNHeatmap, GRNPlot used to infer enhancer-based eGRNs) — https://github.com/CostaLab/scMEGA
- **Seurat v5** (Underlying single-cell multiome object model and co-embedding/CCA integration for trajectory-ordered analysis) — https://github.com/satijalab/seurat
- **Signac** (ATAC-seq processing (peak accessibility, gene annotations via EnsDb.Hsapiens.v86) feeding peak-to-gene links) — https://github.com/stuart-lab/signac
- **ArchR** (Approach used for pseudo-time trajectory inference (AddTrajectory) prior to TF-gene correlation)
- **igraph** (Backend for GRN network construction and visualization (GRNHeatmap/GRNPlot))
- **ggraph** (Graphics layer used with igraph for GRNPlot network plots)

## Examples

```
obj <- RunChromVAR(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38); cor <- GetTFGeneCorrelation(obj, tf.assay = 'chromvar', gene.assay = 'RNA', trajectory.name = 'Trajectory')
```

## Evaluation signals

- Candidate TFs pass SelectTFs correlation cutoff (cor.cutoff = 0.4).
- Retained network edges exceed the correlation > 0.5 filter threshold.
- In the CD4 T cell pairing benchmark, recovery rates meet reported thresholds: >75% of TFs, 83% of genes, and 60% of TF-gene regulations recovered from predicted pairs vs true pairs.
- chromVAR deviation scores computed for all JASPAR2020-matched TFs across cells; edge tables and recovery-rate summary saved (e.g. cd4_tcell_grn_recovery_rates.csv).
- Final GRN integrates all three evidence layers: motif matching, TF-gene correlations, and peak-to-gene links.

## Limitations

- Software/package versions not specified in the tutorials (only sessionInfo() noted).
- Depends on quality of upstream cell pairing/integration — the benchmark showed TF-gene regulation recovery of only ~60% when using predicted pairs instead of true pairs.
- Requires trajectory inference quality (AddTrajectory on embedding such as MOJITOO_UMAP); poor trajectory ordering degrades TF-gene correlations.

## Evidence

- [methods] TF activity estimation via motif matching and chromVAR deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [methods] Pseudo-time trajectory inference: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
- [methods] AddTrajectory usage: "obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3),"
- [methods] RunChromVAR usage: "obj <- RunChromVAR(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38,"
- [methods] JASPAR2020 motif source: "pfm <- getMatrixSet(x = JASPAR2020,"
- [readme] CD4 T cell tutorial scope: "* [CD4 T cells activcation using 10X multiome data](https://costalab.github.io/scMEGA/articles/pbmc_10x_multiome.html)"

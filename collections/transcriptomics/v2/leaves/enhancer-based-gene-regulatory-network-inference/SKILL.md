---
name: enhancer-based-gene-regulatory-network-inference
description: Use when you have paired or integrated scRNA-seq/scATAC-seq (or 10x Multiome)
  data for a cell trajectory and need to infer which TFs regulate which genes via
  enhancer binding, e.g. reconstructing GRNs across pseudotime or validating predicted
  cell pairings.
when_to_use_negative:
- You only have single-modality scRNA-seq or scATAC-seq without paired/integratable
  multiomic data — the enhancer-based rule requires motif accessibility linked to
  gene expression
- You need a genome-wide static regulatory map rather than a trajectory-resolved eGRN
  for a specific differentiation axis
- Input is already a finished network or feature table and no trajectory/TF-activity
  computation is required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMEGA
  role: 'End-to-end eGRN inference: CoembedData, PairCells, AddTrajectory, AddMotifs,
    RunChromVAR, SelectTFs/SelectGenes, GetTFGeneCorrelation, GetGRN, GRNHeatmap,
    GRNPlot'
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: 'Underlying single-cell framework: CCA-based co-embedding, FindClusters, Assay5
    object handling'
  repo: github.com/satijalab/seurat
- name: Signac
  role: Chromatin/ATAC assay handling, motif object and peak-gene infrastructure within
    the paired Seurat object
  repo: github.com/stuart-lab/signac
- name: ArchR
  role: Trajectory inference approach underlying AddTrajectory pseudotime
- name: chromVAR
  role: Computes per-cell TF deviation scores from motif accessibility after AddMotifs
- name: Nebulosa
  role: Kernel-density visualization of gene/peak features to recover dropout-affected
    signals during annotation
  repo: https://github.com/powellgenomicslab/Nebulosa
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:b3448f54582132f775d5df47aa5a9906a9c44a732ee98ba85f8efe802c0e4bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - JASPAR2020
  - Nebulosa
  - R
  - Seurat
  - Signac
  - TFBSTools
  - chromVAR
  - destiny
  - devtools
  - ggraph
  - igraph
  - optmatch
  - scMEGA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.72
n_claims: 25
---

# enhancer-based-gene-regulatory-network-inference

## Summary

Infer enhancer-based gene regulatory networks (eGRNs) from single-cell multiomic/multimodal data with scMEGA, which assembles TF-gene edges only when a gene's associated enhancer (peak) is bound by that TF, combining motif matching, chromVAR TF activity, and peak-to-gene links along an inferred pseudotime trajectory.

## When to use

Use when you have paired or integrated scRNA-seq/scATAC-seq (or 10x Multiome) data for a cell trajectory and need to infer which TFs regulate which genes via enhancer binding, e.g. reconstructing GRNs across pseudotime or validating predicted cell pairings.

## When NOT to use

- You only have single-modality scRNA-seq or scATAC-seq without paired/integratable multiomic data — the enhancer-based rule requires motif accessibility linked to gene expression
- You need a genome-wide static regulatory map rather than a trajectory-resolved eGRN for a specific differentiation axis
- Input is already a finished network or feature table and no trajectory/TF-activity computation is required

## Inputs

- Seurat objects with RNA and ATAC assays (e.g. snRNA.rds, snATAC.rds, gene.activity.rds from the fibroblast case study)
- 10x Genomics PBMC multiome data (pbmc_granulocyte_sorted_10k) as a Seurat object
- JASPAR2020 CORE vertebrate motif PFMs
- BSgenome.Hsapiens.UCSC.hg38 genome (hg38 annotations, e.g. via EnsDb.Hsapiens.v86)
- Predicted-pair assignments from cell-pairing benchmarks (for recovery-rate comparisons)

## Outputs

- Enhancer-based GRN edge list (tf, gene, correlation/weights columns), e.g. final_grn.Rds
- TF-gene correlation table from GetTFGeneCorrelation
- Peak-to-gene link table from SelectGenes
- Regulatory module heatmaps (GRNHeatmap) and network plots (GRNPlot/igraph/ggraph)
- Recovery-rate summaries comparing predicted-pair vs true-pair networks (e.g. cd4_tcell_grn_recovery_rates.csv)

## How to apply

Co-embed the RNA and ATAC modalities (scMEGA CoembedData wraps Seurat CCA, optionally with RunHarmony batch correction on variables like patient, region, and tech), then perform sub-clustering (FindClusters, e.g. resolution 1 and 0.2) and pair cells across modalities with PairCells/CreatePairedObject. Infer pseudotime with AddTrajectory (e.g. trajectory = c(2, 1, 0, 3) from SCARA5+ progenitors to myofibroblasts, or the naive-to-memory CD4 T cell trajectory on MOJITOO_UMAP). Estimate per-cell TF activity by matching JASPAR2020 CORE vertebrate motifs against the genome (AddMotifs with BSgenome.Hsapiens.UCSC.hg38) and computing chromVAR deviation scores (RunChromVAR); select candidate TFs with SelectTFs (cor.cutoff = 0.4) and genes with SelectGenes based on peak-to-gene accessibility-expression correlation. Correlate chromVAR TF activity with RNA expression along the trajectory via GetTFGeneCorrelation (tf.assay = 'chromvar', gene.assay = 'RNA'), then assemble the eGRN with GetGRN, which retains a TF-gene interaction only when the gene has an associated enhancer and the TF is bound to one of those enhancers. Finally filter edges by a correlation threshold (0.4–0.5 depending on the case study), visualize with GRNHeatmap/GRNPlot (igraph/ggraph), and save the edge table (e.g. final_grn.Rds) and recovery summaries.

## Related tools

- **scMEGA** (End-to-end eGRN inference: CoembedData, PairCells, AddTrajectory, AddMotifs, RunChromVAR, SelectTFs/SelectGenes, GetTFGeneCorrelation, GetGRN, GRNHeatmap, GRNPlot) — https://github.com/CostaLab/scMEGA
- **Seurat** (Underlying single-cell framework: CCA-based co-embedding, FindClusters, Assay5 object handling) — github.com/satijalab/seurat
- **Signac** (Chromatin/ATAC assay handling, motif object and peak-gene infrastructure within the paired Seurat object) — github.com/stuart-lab/signac
- **ArchR** (Trajectory inference approach underlying AddTrajectory pseudotime)
- **chromVAR** (Computes per-cell TF deviation scores from motif accessibility after AddMotifs)
- **Nebulosa** (Kernel-density visualization of gene/peak features to recover dropout-affected signals during annotation) — https://github.com/powellgenomicslab/Nebulosa

## Examples

```
obj <- RunChromVAR(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38); obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3))
```

## Evaluation signals

- final_grn.Rds exists and contains directed TF-to-gene edges with tf/gene/weights fields, with all retained weights above the chosen correlation threshold (0.4 or 0.5)
- GRNHeatmap resolves distinct regulatory modules matching annotated sub-clusters (e.g. module 1 for SCARA5+ progenitors with NR3C2, myofibroblast module with RUNX1 top-ranked by betweenness/page-rank centrality)
- Recovery-rate validation: >75% of TFs, 83% of genes, and 60% of TF-gene regulations recovered when using OptMatch-predicted pairs instead of true pairs (CD4 T cell case)
- All genes retained in the GRN are associated with at least one enhancer (peak) in the peak-to-gene link table and each edge's TF motif occurs in one of those enhancers
- Batch-corrected co-embedding shows cells from different patients, regions, and modalities well mixed before pairing and trajectory inference

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced)
- Edge filtering thresholds (cor.cutoff = 0.4 for TF selection; 0.4–0.5 for GRN edges) are case-study choices and may need retuning per dataset
- GRN assembly depends on the quality of peak-to-gene links and chromVAR deviation scores; sparsity of single-cell features can affect TF-activity estimation (motivating Nebulosa-style visualization)

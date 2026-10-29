---
name: chromvar-transcription-factor-activity-estimation
description: Use when inferring transcription factor activity from single-cell chromatin
  accessibility peaks (scATAC-seq or multiome), as a prerequisite for linking TF motif
  deviation scores to gene expression in enhancer-based gene regulatory network construction.
when_to_use_negative:
- Input lacks chromatin accessibility peaks (e.g. only a snRNA-seq expression matrix)
  — chromVAR deviation scores require open chromatin peaks to match motifs against.
- Genome build of the peak coordinates is unknown or mismatched with the BSgenome
  supplied to AddMotifs/RunChromVAR (e.g. hg38 peaks with a different genome build).
edam_operation: http://edamontology.org/operation_0440
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3169
tools:
- name: scMEGA
  role: wraps AddMotifs/RunChromVAR and downstream GRN assembly (GetGRN, SelectTFs,
    SelectGenes, GetTFGeneCorrelation)
  repo: github.com/CostaLab/scMEGA
- name: chromVAR
  role: computes TF deviation scores from motif matches on the peak matrix
- name: JASPAR2020
  role: source of CORE vertebrate motif PFMs
- name: TFBSTools
  role: getMatrixSet to retrieve JASPAR2020 PFMs
- name: Signac
  role: peak-matrix object infrastructure installed from the seurat5 branch
  repo: github.com/stuart-lab/signac (branch seurat5)
- name: Seurat
  role: single-cell object model used throughout the workflow (v5 branch)
  repo: github.com/satijalab/seurat (branch seurat5)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:03bc320f0c831d66b3013438e6e86faf73d8176bde23bd89365cba512463ae42
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_0440
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# chromvar-transcription-factor-activity-estimation

## Summary

Estimates per-cell transcription factor (TF) activity from scATAC-seq by matching JASPAR2020 CORE vertebrate motif PFMs against peak sequences with the hg38 genome and computing chromVAR deviation scores, a key intermediate for enhancer-based GRN inference in scMEGA.

## When to use

Use when inferring transcription factor activity from single-cell chromatin accessibility peaks (scATAC-seq or multiome), as a prerequisite for linking TF motif deviation scores to gene expression in enhancer-based gene regulatory network construction.

## When NOT to use

- Input lacks chromatin accessibility peaks (e.g. only a snRNA-seq expression matrix) — chromVAR deviation scores require open chromatin peaks to match motifs against.
- Genome build of the peak coordinates is unknown or mismatched with the BSgenome supplied to AddMotifs/RunChromVAR (e.g. hg38 peaks with a different genome build).

## Inputs

- Seurat/Signac object with scATAC-seq peak matrix (e.g. snATAC.rds from the human heart fibroblast dataset)
- BSgenome.Hsapiens.UCSC.hg38 genome
- JASPAR2020 CORE vertebrate motif PFMs via TFBSTools::getMatrixSet
- Trajectory/pseudotime assignments on paired cells (from AddTrajectory over diffusion-map dims)

## Outputs

- Per-cell chromVAR TF deviation scores added to the Seurat object
- Trajectory-correlated TF selection (SelectTFs)
- TF-gene correlation edge table (e.g. final_grn.Rds, filtered at correlation > 0.4)
- Assembled enhancer-based GRN via GetGRN with regulator centrality (betweenness, page-rank)

## How to apply

Add motif annotations to the Signac/Seurat object containing the peak matrix using AddMotifs with the appropriate BSgenome (BSgenome.Hsapiens.UCSC.hg38 for human data) and a motif set such as JASPAR2020 CORE vertebrate PFMs retrieved via TFBSTools::getMatrixSet. Then run RunChromVAR with the same genome to compute TF deviation scores per cell. Select trajectory-correlated TFs (SelectTFs) and genes (SelectGenes), then correlate chromVAR TF activity with RNA gene expression along the trajectory (GetTFGeneCorrelation) to build the TF-gene edge table; edges with correlation > 0.4 are retained in the final GRN. The resulting deviation scores, together with motif matching and peak-to-gene links, feed into GetGRN to assemble the enhancer-based regulatory network, where regulator importance is ranked by betweenness and page-rank centrality (e.g. RUNX1 in the myofibroblast case study).

## Related tools

- **scMEGA** (wraps AddMotifs/RunChromVAR and downstream GRN assembly (GetGRN, SelectTFs, SelectGenes, GetTFGeneCorrelation)) — github.com/CostaLab/scMEGA
- **chromVAR** (computes TF deviation scores from motif matches on the peak matrix)
- **JASPAR2020** (source of CORE vertebrate motif PFMs)
- **TFBSTools** (getMatrixSet to retrieve JASPAR2020 PFMs)
- **Signac** (peak-matrix object infrastructure installed from the seurat5 branch) — github.com/stuart-lab/signac (branch seurat5)
- **Seurat** (single-cell object model used throughout the workflow (v5 branch)) — github.com/satijalab/seurat (branch seurat5)

## Examples

```
obj <- AddMotifs(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38); obj <- RunChromVAR(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38)
```

## Evaluation signals

- final_grn.Rds exists and contains tf-gene-correlation edges retained at correlation > 0.4
- GRNHeatmap resolves two regulatory modules matching the SCARA5+ progenitor and POSTN/COL1A1+ myofibroblast sub-clusters
- Regulator centrality ranks (betweenness and page-rank) identify RUNX1 as the top TF and place NR3C2 in module 1
- PseudotimePlot shows coherent NR3C2 and RUNX1 binding activity, expression, and target expression along pseudotime from SCARA5+ progenitors to myofibroblasts

## Limitations

- Software/package versions are not specified in the tutorials (sessionInfo() is the only reference).
- Motif activity inference depends on genome build consistency and the accuracy of peak-to-gene links and cell pairing between modalities.

## Evidence

- [methods] TF activity estimation via motif matching and chromVAR deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [methods] AddMotifs with hg38 genome: "obj <- AddMotifs(
  object = obj,
  genome = BSgenome.Hsapiens.UCSC.hg38,"
- [methods] JASPAR2020 motif set retrieval: "pfm <- getMatrixSet(
  x = JASPAR2020,"
- [readme] scMEGA installation from GitHub: "# Install scMEGA
devtools::install_github("CostaLab/scMEGA")"
- [abstract] Myofibroblast GRN case study overview: "We provide a case study on gene regulatory networks controlling myofibroblast activation in human myocardial infarction."

---
name: peak-to-gene-linking-from-multiome-data
description: Use when you have paired scRNA-seq/scATAC-seq (e.g. 10x Genomics PBMC
  multiome) cells ordered along a trajectory and need enhancer-to-promoter links to
  constrain TF-gene edges in an eGRN.
when_to_use_negative:
- Input is unpaired snRNA-seq and snATAC-seq from separate cells without a cell-pairing
  step (PairCells/CreatePairedObject) — linking along a shared trajectory first requires
  paired or pseudo-paired modalities
- You only have scRNA-seq or only scATAC-seq data — no peak accessibility/gene expression
  pairing is possible
- The goal is bulk or non-trajectory GRN inference where enhancer-to-promoter links
  along pseudotime are not meaningful
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMEGA
  role: Provides SelectGenes/SelectTFs and GetGRN, which build the eGRN from motif
    matching, TF-gene correlations, and peak-to-gene links
  repo: github.com/CostaLab/scMEGA
- name: Seurat v5
  role: CCA-based co-embedding of RNA and ATAC modalities and clustering underlying
    the trajectory; wrapped by CoembedData
  repo: github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: Single-cell chromatin data structures (peaks, gene annotations) used for peak
    accessibility and motif matching
  repo: github.com/stuart-lab/signac (branch seurat5)
- name: ArchR
  role: Trajectory inference approach adopted by scMEGA for pseudotime along which
    accessibility–expression correlations are computed
- name: JASPAR2020
  role: Source of CORE vertebrate motif PFMs matched against hg38 for TF activity
    estimation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:223c95c820bad3ba66067bf69f8a0209c277ccb6ba124094ef6a298c7a61a736
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - JASPAR2020
  - Seurat v5
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# peak-to-gene-linking-from-multiome-data

## Summary

Correlates peak accessibility (scATAC-seq) with gene expression (scRNA-seq) along a pseudotime trajectory in single-cell multiome data to define enhancer-to-promoter (peak-to-gene) links that support enhancer-based gene regulatory network inference with scMEGA.

## When to use

Use when you have paired scRNA-seq/scATAC-seq (e.g. 10x Genomics PBMC multiome) cells ordered along a trajectory and need enhancer-to-promoter links to constrain TF-gene edges in an eGRN.

## When NOT to use

- Input is unpaired snRNA-seq and snATAC-seq from separate cells without a cell-pairing step (PairCells/CreatePairedObject) — linking along a shared trajectory first requires paired or pseudo-paired modalities
- You only have scRNA-seq or only scATAC-seq data — no peak accessibility/gene expression pairing is possible
- The goal is bulk or non-trajectory GRN inference where enhancer-to-promoter links along pseudotime are not meaningful

## Inputs

- Seurat/Signac object containing paired scRNA-seq and scATAC-seq data (e.g. 10x Genomics PBMC multiome pbmc_granulocyte_sorted_10k)
- Pseudotime trajectory added via AddTrajectory on a co-embedded reduction (e.g. CCA or MOJITOO_UMAP embedding)
- hg38 genome with JASPAR2020 CORE vertebrate motif PFMs and EnsDb.Hsapiens.v86 gene annotations

## Outputs

- Peak-to-gene (enhancer-to-promoter) links derived from trajectory-wise correlation of peak accessibility and gene expression
- Enhancer-based eGRN edge tables from GetGRN (TF-gene regulations supported by motif matching, TF activity, and peak-to-gene links)
- Recovery-rate summary (e.g. cd4_tcell_grn_recovery_rates.csv) when comparing predicted-pair vs true-pair networks

## How to apply

First co-embed the RNA and ATAC modalities into a common low-dimensional space (CoembedData wraps Seurat's CCA method) and optionally batch-correct with Harmony, then infer a pseudotime trajectory with AddTrajectory (e.g. the naive-to-memory CD4 T cell trajectory on the MOJITOO_UMAP embedding, trajectory = c(2, 1, 0, 3)). For candidate gene selection, run SelectGenes, which correlates peak accessibility with gene expression along the trajectory — the article also uses SelectTFs with cor.cutoff = 0.4 to pick candidate TFs from chromVAR deviation scores. Assemble the enhancer-based eGRN with GetGRN, which combines JASPAR2020 CORE motif matching (AddMotifs against hg38/BSgenome.Hsapiens.UCSC.hg38), TF-gene correlations from RunChromVAR deviation scores, and the peak-to-gene links. The rationale is that enhancer-to-promoter association restricts inferred regulation to TF-gene pairs whose TF motif-enriched peaks are linked to the gene, improving specificity over correlation-only GRN inference.

## Related tools

- **scMEGA** (Provides SelectGenes/SelectTFs and GetGRN, which build the eGRN from motif matching, TF-gene correlations, and peak-to-gene links) — github.com/CostaLab/scMEGA
- **Seurat v5** (CCA-based co-embedding of RNA and ATAC modalities and clustering underlying the trajectory; wrapped by CoembedData) — github.com/satijalab/seurat (branch seurat5)
- **Signac** (Single-cell chromatin data structures (peaks, gene annotations) used for peak accessibility and motif matching) — github.com/stuart-lab/signac (branch seurat5)
- **ArchR** (Trajectory inference approach adopted by scMEGA for pseudotime along which accessibility–expression correlations are computed)
- **JASPAR2020** (Source of CORE vertebrate motif PFMs matched against hg38 for TF activity estimation)

## Examples

```
obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3), reduction = 'MOJITOO_UMAP'); grn <- GetGRN(obj.pair, tf.assay = 'chromvar', gene.assay = 'RNA', trajectory.name = 'Trajectory')
```

## Evaluation signals

- Peak-to-gene links feed GetGRN edge tables where filtered edges (TF-gene correlation > 0.5) form the final network
- Candidate TFs pass SelectTFs cor.cutoff = 0.4 before entering the GRN
- Predicted-pair network recovers >75% of TFs, 83% of genes, and 60% of TF-gene regulations relative to the true-pair network (cd4_tcell_grn_recovery_rates.csv)
- Genes and peaks are restricted to cells along the inferred trajectory (e.g. naive-to-memory CD4 T cell trajectory), so cell ordering must precede correlation
- Motif matching must succeed against hg38 (BSgenome.Hsapiens.UCSC.hg38) with JASPAR2020 PFMs before chromVAR deviation scores are available

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so reproducibility of exact correlation values may vary
- Correlation-based linking depends on trajectory quality and on pairing accuracy of the modalities; scMEGA's CD4 T cell benchmark shows predicted pairs recover only ~60% of TF-gene regulations even when TF/TF-gene agreement is high

## Evidence

- [abstract] scMEGA enables end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and: "enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and"
- [methods] Trajectory inference grounds the correlations: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
- [methods] TF activity via motif matching and chromVAR deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [methods] Modality pairing prerequisite for linking: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [readme] scMEGA install path for reuse: "# Install scMEGA
devtools::install_github("CostaLab/scMEGA")"

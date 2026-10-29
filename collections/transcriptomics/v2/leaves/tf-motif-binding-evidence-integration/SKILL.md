---
name: tf-motif-binding-evidence-integration
description: Use when building a GRN from single-cell multiome (snRNA-seq + snATAC-seq)
  data where candidate TF–gene interactions must be filtered by enhancer–promoter
  linkage plus TF motif occurrence in linked peaks, as in scMEGA's eGRN assembly.
when_to_use_negative:
- Input is single-omic (e.g., scRNA-seq only) with no ATAC peaks, motif matrix, or
  peak-to-gene links — the enhancer-based criterion cannot be evaluated.
- You want a purely co-expression/correlation-based GRN without cis-regulatory evidence;
  scMEGA will drop TF–gene pairs lacking a bound, linked enhancer.
- Bulk multi-omics data without per-cell pairing/trajectory, since scMEGA's workflow
  depends on single-cell multiome objects and pseudo-time (AddTrajectory) context.
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0603
- http://edamontology.org/topic_0102
tools:
- name: scMEGA
  role: Provides GetGRN, which intersects motif matching, TF–gene correlations, and
    peak-to-gene links to assemble the enhancer-based GRN
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: Hosts the multi-omic Seurat object whose ATAC assay stores the motif occurrence
    matrix; AddMotifs builds motif matching against the genome
  repo: https://github.com/satijalab/seurat
- name: Signac
  role: Handles the single-cell chromatin data (peaks, links, annotations) within
    the Seurat v5 object used for eGRN assembly
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:498020545289e4b5f3b0937c12c388745e34b7233d330866b5a36bd3c038506a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - R
  - Seurat
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0603
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tf-motif-binding-evidence-integration

## Summary

scMEGA's enhancer-based rule for assembling a gene regulatory network: a TF–gene edge is retained only when the gene has at least one linked enhancer (peak-to-gene link) and the TF's motif is present (bound) on one of those linked enhancers, additionally supported by TF–gene correlation. This grounds inferred TF targets in cis-regulatory evidence rather than co-expression alone.

## When to use

Use when building a GRN from single-cell multiome (snRNA-seq + snATAC-seq) data where candidate TF–gene interactions must be filtered by enhancer–promoter linkage plus TF motif occurrence in linked peaks, as in scMEGA's eGRN assembly.

## When NOT to use

- Input is single-omic (e.g., scRNA-seq only) with no ATAC peaks, motif matrix, or peak-to-gene links — the enhancer-based criterion cannot be evaluated.
- You want a purely co-expression/correlation-based GRN without cis-regulatory evidence; scMEGA will drop TF–gene pairs lacking a bound, linked enhancer.
- Bulk multi-omics data without per-cell pairing/trajectory, since scMEGA's workflow depends on single-cell multiome objects and pseudo-time (AddTrajectory) context.

## Inputs

- Seurat/Signac trajectory object with an ATAC motifs assay (obj@assays$ATAC@motifs@data)
- Peak-to-gene link table (df.p2g) with enhancer-to-gene associations (>= 2 kb enhancer-to-gene links)
- TF–gene correlation data frame (df.cor) from trajectory-time-point correlation analysis
- JASPAR2020 position frequency matrices and BSgenome.Hsapiens.UCSC.hg38 for motif matching

## Outputs

- Enhancer-based GRN edge list (final_grn.Rds) with directed TF-to-gene edges carrying tf, gene, and weights (correlation) fields, all weights > 0.4
- Directed, weighted network visualization (e.g., GRNPlot via igraph/ggraph) with TF nodes colored by trajectory time point

## How to apply

First, extract the motif occurrence matrix from the trajectory Seurat object's ATAC assay (obj@assays$ATAC@motifs@data, built via AddMotifs on BSgenome.Hsapiens.UCSC.hg38 with JASPAR2020 PFMs) and subset it to the peaks present in the peak-to-gene link table and the TFs retained in the TF–gene correlation table. Then apply scMEGA's GetGRN function with the motif matching matrix, the TF–gene correlation data frame (df.cor), and the peak-to-gene links (df.p2g); the decision rule is that a gene is linked to a TF only when an associated peak is bound by that TF and the TF–gene correlation supports the regulation. Finally, filter the resulting edge list to correlations above 0.4, keeping tf, gene, and weights columns. Rationale: enhancer-based evidence (motif hit inside a linked enhancer) prevents co-expression artifacts from becoming regulatory edges.

## Related tools

- **scMEGA** (Provides GetGRN, which intersects motif matching, TF–gene correlations, and peak-to-gene links to assemble the enhancer-based GRN) — https://github.com/CostaLab/scMEGA
- **Seurat** (Hosts the multi-omic Seurat object whose ATAC assay stores the motif occurrence matrix; AddMotifs builds motif matching against the genome) — https://github.com/satijalab/seurat
- **Signac** (Handles the single-cell chromatin data (peaks, links, annotations) within the Seurat v5 object used for eGRN assembly) — https://github.com/stuart-lab/signac

## Examples

```
grn <- GetGRN(motif_matching = obj@assays$ATAC@motifs@data, df.cor = df.cor, df.p2g = df.p2g)
```

## Evaluation signals

- final_grn.Rds exists and contains a directed TF-to-gene edge list with tf/gene/weights fields
- Every retained edge weight (correlation) exceeds the 0.4 threshold
- For each retained edge, the target gene has at least one peak-to-gene link and the TF motif is present in at least one of those linked peaks (spot-checkable against the motif matrix)
- Edge count strictly decreases relative to the unfiltered TF–gene correlation table, confirming the enhancer-binding intersection was applied
- Graph loaded in igraph is directed and weighted, and node attributes include trajectory time point for TF coloring

## Limitations

- Software/package versions are not pinned in the tutorials (only sessionInfo() is suggested), so reproducibility requires recording versions manually.
- Retained edges depend on JASPAR2020 motif matching quality and the >= 2 kb peak-to-gene linkage window; weak or novel motifs may be missed.
- No changelog is provided for scMEGA, so behavior of GetGRN across versions may differ.

## Evidence

- [other] scMEGA retains a TF–gene interaction only if the gene is associated with at least one enhancer (peak) and the TF is bound to one of those associated enhancers: [evidence span withheld — anchor status: not_found]
- [methods] TF activity via motif matching and deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [abstract] scMEGA end-to-end GRN inference scope: "We here describe scMEGA (Single-cell Multiomic Enhancer-based Gene Regulatory Network Inference) that enables an end-to-end analysis of multi-omics data"
- [readme] README tutorials for building GRNs from multiomics data: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"

---
name: peak-to-gene-association-filtering
description: Use when building an enhancer-based GRN from paired snRNA-seq/snATAC-seq
  co-embedded cells and you need to restrict candidate genes to those both variable
  along pseudotime and linkable to at least one peak.
when_to_use_negative:
- Cells are unpaired or not co-embedded (no CCA/Harmony coembedding), since gene–peak
  association requires paired RNA/ATAC cells
- No pseudotime trajectory has been defined — variation-along-pseudotime filtering
  is meaningless on unordered clusters
- You need genome-wide enhancer–promoter links rather than trajectory-specific candidate
  genes for GRN seeding
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMEGA
  role: Provides SelectGenes and the enhancer-based pipeline that links pseudotime-variable
    genes to peaks and emits the candidate gene set
  repo: https://github.com/CostaLab/scMEGA
- name: ArchR
  role: Trajectory approach used to order clusters and define the pseudotime along
    which gene variation is computed
- name: Seurat
  role: Co-embedding (CCA), Harmony batch correction and sub-clustering produce the
    paired object that SelectGenes operates on
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:be73e899207d9a573aecdb5f42677032edf950d0ab6c764a70445be1d5d30910
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peak-to-gene-association-filtering

## Summary

scMEGA-style filtering that links candidate genes to at least one accessible ATAC peak and retains only genes whose expression varies along pseudotime, producing the trajectory-relevant enhancer–gene set used as GRN input.

## When to use

Use when building an enhancer-based GRN from paired snRNA-seq/snATAC-seq co-embedded cells and you need to restrict candidate genes to those both variable along pseudotime and linkable to at least one peak.

## When NOT to use

- Cells are unpaired or not co-embedded (no CCA/Harmony coembedding), since gene–peak association requires paired RNA/ATAC cells
- No pseudotime trajectory has been defined — variation-along-pseudotime filtering is meaningless on unordered clusters
- You need genome-wide enhancer–promoter links rather than trajectory-specific candidate genes for GRN seeding

## Inputs

- Seurat/ArchR paired object (coembed.sub.rds) with co-embedded snRNA-seq and snATAC-seq cells of the lineage of interest
- Pseudotime trajectory added via AddTrajectory over predefined clusters (e.g. SCARA5+ progenitor → COL1A1/POSTN/FN1-high myofibroblast)
- Peak-to-gene linkage metadata (genes associated with at least one ATAC peak)

## Outputs

- Filtered set of trajectory-relevant candidate genes (top 10% most variable along pseudotime, each linked to ≥1 peak)
- Table file of the ranked candidate gene set used as GRN inference input

## How to apply

After pairing RNA and ATAC cells with PairCells/CreatePairedObject and fitting the pseudotime with AddTrajectory (clusters ordered from the SCARA5+ progenitor state to the terminal myofibroblast state, e.g. trajectory = c(2, 1, 0, 3) in the article's fibroblast case), compute smoothed per-gene expression along pseudotime. Run SelectGenes with labelTop1 = 0 and labelTop2 = 0 and the var.cutoff.gene, cor.cutoff and fdr.cutoff parameters so that only genes varying along pseudotime and associated with at least one ATAC peak are retained; a gene is only considered a potential target when linkable to at least one peak. Rank retained genes by their variation along pseudotime and select the top 10% most variable as the trajectory-relevant candidate set (cutoff user-adjustable). This filtered gene set is then the input for downstream GRN inference, where TF activities come from motif matching (JASPAR2020/CIS-BP) and chromVAR deviation scores.

## Related tools

- **scMEGA** (Provides SelectGenes and the enhancer-based pipeline that links pseudotime-variable genes to peaks and emits the candidate gene set) — https://github.com/CostaLab/scMEGA
- **ArchR** (Trajectory approach used to order clusters and define the pseudotime along which gene variation is computed)
- **Seurat** (Co-embedding (CCA), Harmony batch correction and sub-clustering produce the paired object that SelectGenes operates on) — https://github.com/satijalab/seurat

## Examples

```
obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3)); genes <- SelectGenes(obj.pair, reduction = "pseudotime", labelTop1 = 0, labelTop2 = 0)
```

## Evaluation signals

- Output gene table is non-empty after SelectGenes filtering
- Every retained gene is associated with at least one ATAC peak (genes without peak links must be dropped)
- Retained genes show measurable expression variation along the inferred pseudotime; ranking orders by this variation
- Selected set is approximately the top 10% most variable genes (cutoff adjustable; verify against var.cutoff.gene)
- Filtered gene set feeds a downstream GRN with TF activities from motif matching and chromVAR deviation scores without further peak-link errors

## Limitations

- The 10% variability cutoff is user-adjustable, so results are parameter-sensitive (var.cutoff.gene, cor.cutoff, fdr.cutoff)
- Gene–peak links are required for a gene to be a potential target, so genes lacking linkable peaks are silently excluded
- Tutorial/session information does not pin software/package versions (article notes 'software/package versions not specified in the tutorials')

## Evidence

- [abstract] enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association: "enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association"
- [methods] we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
- [methods] obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3),: "obj.pair <- AddTrajectory(object = obj.pair, 
                          trajectory = c(2, 1, 0, 3),"
- [readme] We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"

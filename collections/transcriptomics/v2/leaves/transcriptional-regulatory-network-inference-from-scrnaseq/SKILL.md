---
name: transcriptional-regulatory-network-inference-from-scrnaseq
description: Use when you have a time-course or trajectory-ordered scRNA-seq dataset
  with cell-type labels and need to identify upstream temporal transcription factors
  (e.g., as downstream targets of LR signaling) via TRN inference plus MRA.
when_to_use_negative:
- The goal is inferring ligand-receptor communication itself (eLRs); tTF identification
  is a downstream by-product, so TRN/MRA should be run only after trajectory-variable
  gene inputs exist.
- The dataset lacks cell-type groupings or trajectory-variable genes to supply as
  the MRA phenotype.
- You already possess a curated TF/regulon table (e.g., a precomputed B_blastoid_RTN_mra_result.rds);
  re-inference is unnecessary and, per the README, the TRN build step is very time
  consuming.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TimeTalk
  role: Provides GetCellTypeSpecificTRN to build per-cell-type RTNs on the RTN framework
  repo: https://github.com/ChengLiLab/TimeTalk
- name: RTN
  role: Supplies the 'tni' object construction, tna.mra master regulator analysis,
    and tna.get extraction
- name: Seurat
  role: FindAllMarkers (p_val_adj < 0.05) defines cell-type marker genes used to filter
    retained MRA regulators
- name: monocle2
  role: differentialGeneTest (qval = 0.01) yields trajectory-variable genes and variance
    used as the MRA phenotype
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: preprocessCore
  role: normalize.quantiles performs quantile normalization of the merged expression
    matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:8ecb920f020662e231f8fb306bca090c9f2866025481921bdf0b965f4ef1c3bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - RTN
  - Seurat
  - TimeTalk
  - monocle2
  - preprocessCore
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# transcriptional-regulatory-network-inference-from-scrnaseq

## Summary

Infers cell-type-specific transcriptional regulatory networks (TRNs) from scRNA-seq with RTN and identifies temporal TFs (tTFs) by master regulator analysis (tna.mra), yielding 229 tTFs in the TimeTalk pre-implantation embryo study.

## When to use

Use when you have a time-course or trajectory-ordered scRNA-seq dataset with cell-type labels and need to identify upstream temporal transcription factors (e.g., as downstream targets of LR signaling) via TRN inference plus MRA.

## When NOT to use

- The goal is inferring ligand-receptor communication itself (eLRs); tTF identification is a downstream by-product, so TRN/MRA should be run only after trajectory-variable gene inputs exist.
- The dataset lacks cell-type groupings or trajectory-variable genes to supply as the MRA phenotype.
- You already possess a curated TF/regulon table (e.g., a precomputed B_blastoid_RTN_mra_result.rds); re-inference is unnecessary and, per the README, the TRN build step is very time consuming.

## Inputs

- Merged pre-implantation scRNA-seq expression matrix normalized to log10(RPKM + 1)
- AnimalTFDB 3.0 transcription factor list
- DEG 10 essential gene set (excluded as candidate regulators)
- HRT Atlas v1.0 housekeeping gene set (excluded as candidate regulators)
- Trajectory-variable genes from monocle2 differentialGeneTest (qval = 0.01) with their variance (MRA phenotype)
- Seurat object with cell-type identities and FindAllMarkers results (p_val_adj < 0.05)

## Outputs

- Per-cell-type RTN objects saved as res/R/*RTN_rtna_object.rds
- Merged MRA table saved as res/R/B_blastoid_RTN_mra_result.rds
- 229 temporal TFs (tTFs) passing the Pvalue < 0.05 and marker-regulon filters
- Six ward.D2 clusters C1–C6 of tTFs with distinct developmental GO enrichments (e.g., C5: Gata6, Tead4, Cdx2, Nanog, Pou5f1)
- C1–C6 dendrogram/heatmap (Fig. 2f) and tTF cluster-assignment table (Supplementary Data 1)

## How to apply

Normalize the merged scRNA-seq matrix to log10(RPKM + 1) and compile the AnimalTFDB 3.0 TF list to build the RTN 'tni' object. For each cell type, run TimeTalk's GetCellTypeSpecificTRN (built on RTN), excluding DEG 10 essential genes and HRT Atlas v1.0 housekeeping genes as candidate regulators. Supply trajectory-variable genes (monocle2 differentialGeneTest, qval = 0.01) and their variance as the phenotype to tna.mra, then extract results with tna.get(rtna, what='mra', ntop = -1). Retain regulators with Pvalue < 0.05 whose regulon targets are that cell type's Seurat FindAllMarkers genes (p_val_adj < 0.05), save per-cell-type rtna objects as res/R/*RTN_rtna_object.rds and the merged table as res/R/B_blastoid_RTN_mra_result.rds. Collect the 229 tTFs, compute average expression profiles, and hierarchically cluster them with the ward.D2 method cut into six clusters (C1–C6) for Fig. 2f and Supplementary Data 1.

## Related tools

- **TimeTalk** (Provides GetCellTypeSpecificTRN to build per-cell-type RTNs on the RTN framework) — https://github.com/ChengLiLab/TimeTalk
- **RTN** (Supplies the 'tni' object construction, tna.mra master regulator analysis, and tna.get extraction)
- **Seurat** (FindAllMarkers (p_val_adj < 0.05) defines cell-type marker genes used to filter retained MRA regulators)
- **monocle2** (differentialGeneTest (qval = 0.01) yields trajectory-variable genes and variance used as the MRA phenotype) — https://github.com/cole-trapnell-lab/monocle3
- **preprocessCore** (normalize.quantiles performs quantile normalization of the merged expression matrix)

## Examples

```
rtna <- myGetCellTypeSpecificTRN(tmp.seu = seu, tmp.ident = ii, gene.TF = gene.TF, tmp.phenotype = tmp.phenotype)
```

## Evaluation signals

- Exactly 229 tTFs are recovered after the Pvalue < 0.05 and cell-type-marker regulon filters
- The 229 tTFs partition into six ward.D2 clusters matching Fig. 2f and Supplementary Data 1
- res/R/B_blastoid_RTN_mra_result.rds and the per-cell-type res/R/*RTN_rtna_object.rds files exist
- Every retained regulator's regulon overlaps its cell type's Seurat FindAllMarkers genes (p_val_adj < 0.05)
- Cluster C5 contains Gata6, Tead4, Cdx2, Nanog, Pou5f1 with blastocyst-development GO enrichments

## Limitations

- The inferred regulation relationship between ligand-receptor interactions and TFs should be confirmed by further wet experiments, which were not performed.
- Building all cell-type-specific TRNs is very time consuming (README advises not running the demo code as-is).
- Granger causality conclusions are affected by the choice of the winsz parameter; users are advised to test various parameter combinations for dependable results.

## Evidence

- [methods] tni_build_and_mra: [evidence span withheld — anchor status: not_found]
- [discussion] mra_phenotype: [evidence span withheld — anchor status: not_found]
- [discussion] tTF_clustering: [evidence span withheld — anchor status: not_found]
- [readme] mra_extraction: "mra <- tna.get(rtna, what="mra", ntop = -1)"
- [intro] tTF_role: "we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs)"

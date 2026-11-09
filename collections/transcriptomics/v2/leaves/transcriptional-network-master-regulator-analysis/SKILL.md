---
name: transcriptional-network-master-regulator-analysis
description: Use when you need to infer which transcription factors drive temporal
  gene programs in scRNA-seq trajectories (e.g. blastocyst EPI/PE lineages) so ligand-receptor
  pairs can be causally linked to TF activity downstream.
when_to_use_negative:
- When cell identities are not yet defined (pre-blastocyst stages without definitive
  cell types) — cell-type-specific TRN construction requires per-identity annotations
- When the goal is ligand-receptor pair discovery alone without downstream TF regulation
  — use plain LR co-varying screens instead
- When only bulk RNA-seq or low-cell-count data is available — MRA requires sufficient
  cells per cell type to build regulons
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: RTN
  role: Builds the 'tni' object and cell-type-specific TRNs; tna.mra / tna.get(what='mra')
    performs the master regulator analysis producing tTFs
  repo: https://github.com/ChengLiLab/TimeTalk
- name: TimeTalk
  role: Provides myGetCellTypeSpecificTRN wrapper to build cell-type-specific TRNs
    and consumes the MRA result (tmp.mra.res) in RunTimeTalk
  repo: https://github.com/ChengLiLab/TimeTalk
- name: Seurat
  role: FindAllMarkers (only.pos = T, p_val_adj < 0.05) supplies the cell-type DE
    genes used to filter master regulator regulons
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:34fb5b342f01aefb5f487656cfb4970a794d0c82118c1b58aaf0f11e2c75779f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - RTN
  - Seurat
  - TimeTalk
  - monocle3
  - tidyverse
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# transcriptional-network-master-regulator-analysis

## Summary

Identify temporal transcription factors (tTFs) by building cell-type-specific transcriptional regulatory networks with the RTN package and running master regulator analysis (tna.mra), keeping master regulators whose regulons are differentially expressed in each cell type. In TimeTalk these tTFs are the downstream partners for Granger causality testing against ligand-receptor pairs.

## When to use

Use when you need to infer which transcription factors drive temporal gene programs in scRNA-seq trajectories (e.g. blastocyst EPI/PE lineages) so ligand-receptor pairs can be causally linked to TF activity downstream.

## When NOT to use

- When cell identities are not yet defined (pre-blastocyst stages without definitive cell types) — cell-type-specific TRN construction requires per-identity annotations
- When the goal is ligand-receptor pair discovery alone without downstream TF regulation — use plain LR co-varying screens instead
- When only bulk RNA-seq or low-cell-count data is available — MRA requires sufficient cells per cell type to build regulons

## Inputs

- log10(RPKM + 1) normalized scRNA-seq expression matrix (Seurat object, e.g. seu.rds)
- TF list from AnimalTFDB (version 3.0)
- Cell-type identity annotations (tmp.ident / phenotype, e.g. EPI, PE, blastocyst clusters)
- Differential expression phenotype used by RTN (variable gene and their variance)

## Outputs

- Cell-type-specific RTN tna objects per cell identity
- Master regulator analysis table filtered to Pvalue < 0.05 with regulons overlapping cell-type marker genes
- Saved RDS file of consolidated MRA results (e.g. B_blastoid_RTN_mra_result.rds) consumed as tmp.mra.res by RunTimeTalk

## How to apply

Build a 'tni' object from log10(RPKM+1) normalized scRNA-seq expression and a TF list from AnimalTFDB (version 3.0), then construct cell-type-specific TRNs per cell identity (e.g. EPI, PE, blastocyst) via myGetCellTypeSpecificTRN with a differential-expression phenotype. Run master regulator analysis with tna.get(rtna, what='mra', ntop=-1) and retain regulators with Pvalue < 0.05 whose Regulon overlaps genes that are significant markers for that cell type (FindAllMarkers, p_val_adj < 0.05, only.pos=TRUE), tagging each row with group = tmp.ident. Bind the per-cell-type results with Reduce(rbind) and save as an .rds (e.g. B_blastoid_RTN_mra_result.rds) that becomes the tmp.mra.res input to RunTimeTalk. The rationale is that only TFs acting as master regulators of lineage-specific gene programs are plausible causal targets of developmental signaling, restricting the Granger LR-to-TF tests to a biologically grounded tTF set.

## Related tools

- **RTN** (Builds the 'tni' object and cell-type-specific TRNs; tna.mra / tna.get(what='mra') performs the master regulator analysis producing tTFs) — https://github.com/ChengLiLab/TimeTalk
- **TimeTalk** (Provides myGetCellTypeSpecificTRN wrapper to build cell-type-specific TRNs and consumes the MRA result (tmp.mra.res) in RunTimeTalk) — https://github.com/ChengLiLab/TimeTalk
- **Seurat** (FindAllMarkers (only.pos = T, p_val_adj < 0.05) supplies the cell-type DE genes used to filter master regulator regulons)

## Examples

```
mra <- tna.get(rtna, what="mra", ntop = -1); mra.res <- mra %>% filter(Pvalue < 0.05, Regulon %in% tmp.gene.use) %>% mutate(group = tmp.ident)
```

## Evaluation signals

- Each retained master regulator has Pvalue < 0.05 in tna.get(what='mra') output
- Every retained Regulon is a member of the cell-type's significant marker set (p_val_adj < 0.05 from FindAllMarkers, only.pos = T)
- Consolidated MRA table carries a group column equal to the cell identity (tmp.ident) for every row
- The saved .rds (B_blastoid_RTN_mra_result.rds) loads and is accepted as tmp.mra.res by RunTimeTalk without schema errors
- Downstream RunTimeTalk with this MRA input recovers the PASS-filtered EPI-PE LR set (~311 pairs) and Fgf4-Fgfr2/Gata6 p-values (p_LR_to_TF = 0.0184, p_TF_to_LR = 4.74e-5)

## Limitations

- Building all cell-type-specific TRNs is very time consuming — the README explicitly warns users not to run the TRN-building demo code on their own demo data
- MRA is purely in silico; inferred LR-to-TF regulatory relationships require wet-lab experimental confirmation which was not performed in the study
- Downstream Granger causality conclusions are sensitive to the interpolation parameter winsz; users are advised to test various parameter combinations

## Evidence

- [discussion] tTF identification via RTN transcriptional network and master regulator analysis: [evidence span withheld — anchor status: not_found]
- [discussion] the variable gene and their variance were input as a phenotype to perform master regulator analysis by function 'tna.mra': "the variable gene and their variance were input as a phenotype to perform master regulator analysis by function “tna.mra”"
- [discussion] the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0) to build the 'tni' object: "the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0)71 to build the “tni” object"
- [intro] we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs): "we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs)"
- [readme] README: rtna <- myGetCellTypeSpecificTRN(tmp.seu = seu, tmp.ident = ii, gene.TF = gene.TF, tmp.phenotype = tmp.phenotype): "rtna <- myGetCellTypeSpecificTRN(tmp.seu = seu,
                                   tmp.ident = ii,
                                   gene.TF = gene.TF,"

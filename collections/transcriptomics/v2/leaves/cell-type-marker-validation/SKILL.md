---
name: cell-type-marker-validation
description: Use when a single-cell TE matrix (scTE .h5ad output from scRNA-seq or
  scTEATAC output from scATAC-seq) has been clustered and you must confirm cell-type
  assignments or marker-TE specificity.
when_to_use_negative:
- Input is already a gene-level expression matrix without TE quantification — scTE/scTEATAC
  must first produce the TE-by-cell matrix
- Bulk RNA-seq or bulk ATAC-seq data — scTE is designed for single-cell BAM/SAM input
  with cell barcode and UMI tags (CB/UB or CR/UR)
- The goal is peak-level enhancer calling from scATAC-seq rather than TE-aggregated
  accessibility — merging TE data to infer enhancers was described as speculative
  and not demonstrated
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTE
  role: 'Quantifies TE expression per cell from single-cell BAM/SAM (exclusive mode:
    reads allocated first to gene exons, then to TEs); supplies the TE-by-cell .h5ad
    matrix in which marker-TE specificity is assessed'
  repo: github.com/JiekaiLab/scTE
- name: scTEATAC
  role: Processes scATAC-seq BAM files against the mm10.te.atac.idx index to obtain
    per-cell TE accessibility counts used for cell-type discrimination and marker-TE
    openness checks
  repo: github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Builds the TE accessibility index from the filtered mm10 RepeatMasker BED
    (LINE/SINE/LTR/Retroposon classes, columns 6-8 and 11)
  repo: github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the gene+TE genome index from a GTF and RepeatMasker BED (e.g. mm10.exclusive.idx)
    required before per-cell TE counting
  repo: github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:97baf4ef22cd6dc9a35d96e5f6de590590913294e0452676c83766ff4f7dccb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTEATAC
  - scTEATAC_build
  - scTE_build
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-marker-validation

## Summary

Validates that computationally recovered cell identities are supported by cell-type-specific transposable element signals, using scTE/scTEATAC quantifications of TE expression or chromatin accessibility as marker readouts.

## When to use

Use when a single-cell TE matrix (scTE .h5ad output from scRNA-seq or scTEATAC output from scATAC-seq) has been clustered and you must confirm cell-type assignments or marker-TE specificity.

## When NOT to use

- Input is already a gene-level expression matrix without TE quantification — scTE/scTEATAC must first produce the TE-by-cell matrix
- Bulk RNA-seq or bulk ATAC-seq data — scTE is designed for single-cell BAM/SAM input with cell barcode and UMI tags (CB/UB or CR/UR)
- The goal is peak-level enhancer calling from scATAC-seq rather than TE-aggregated accessibility — merging TE data to infer enhancers was described as speculative and not demonstrated

## Inputs

- TE-by-cell count/accessibility matrix from scTE (out.h5ad, --hdf5 True) or from scTEATAC on BAM input
- Per-cell metadata/annotations (e.g. FACS-purified identities, gastrulation cell-type labels, UMAP cluster assignments)
- RepeatMasker-derived marker TE list (families such as IAP/RLTR45, MER46C, RLTR20B3, LTRIS2, L1ME3D, ETnERV3-int, RLTR13A, RLTR4_Mm, RLTR13G, RMER19B/C)

## Outputs

- Per-cell-type marker-TE specificity assessment (e.g. accessibility/expression of RLTR13A, RLTR4_Mm, RLTR13G, RMER19B-C restricted to CPCs, CD4+ T cells, ESCs, skin fibroblasts)
- UMAP/clustering built on TE accessibility alone showing cells grouped by known identity
- Validated lineage/fate marker calls such as ETnERV3-int marking early cardiac-fate cells and L1ME3D marking Tnnt2+ cardiomyocytes

## How to apply

After quantifying TEs with scTE (exclusive mode allocates reads first to gene exons, then to TEs, producing a TE-by-cell matrix exported as .h5ad) or with scTEATAC for accessibility, aggregate counts of candidate marker TE families (e.g. IAP/RLTR45, MER46C, RLTR20B3, LTRIS2, L1ME3D, ETnERV3-int, RLTR13A, RLTR4_Mm, RLTR13G, RMER19B/C) by the cell-type annotation. Perform dimensionality reduction/UMAP on TE values alone and check that cells separate by known FACS-purified or annotated identities. Then confirm each expected cell-type/TE association: elevated openness or expression of the marker TE in its assigned cell type relative to all others (e.g. RLTR13A in CPCs, RLTR4_Mm in CD4+ T cells, RLTR13G in ESCs, RMER19B/C in skin fibroblasts). Marker specificity can also be validated dynamically, as with ETnERV3-int marking transient cardiac fate from mesoderm and being reduced in Tnnt2+ cardiomyocytes where L1ME3D is instead expressed.

## Related tools

- **scTE** (Quantifies TE expression per cell from single-cell BAM/SAM (exclusive mode: reads allocated first to gene exons, then to TEs); supplies the TE-by-cell .h5ad matrix in which marker-TE specificity is assessed) — github.com/JiekaiLab/scTE
- **scTEATAC** (Processes scATAC-seq BAM files against the mm10.te.atac.idx index to obtain per-cell TE accessibility counts used for cell-type discrimination and marker-TE openness checks) — github.com/JiekaiLab/scTE
- **scTEATAC_build** (Builds the TE accessibility index from the filtered mm10 RepeatMasker BED (LINE/SINE/LTR/Retroposon classes, columns 6-8 and 11)) — github.com/JiekaiLab/scTE
- **scTE_build** (Builds the gene+TE genome index from a GTF and RepeatMasker BED (e.g. mm10.exclusive.idx) required before per-cell TE counting) — github.com/JiekaiLab/scTE

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- Clustering on TE accessibility alone must recover the expected FACS-purified cell-type groupings in the UMAP
- Each marker TE must show elevated signal in its corresponding cell type relative to all others (RLTR13A→CPCs, RLTR4_Mm→CD4+ T cells, RLTR13G→ESCs, RMER19B/C→skin fibroblasts)
- Output file exists and is valid h5ad format with per-cell TE counts
- Each of the five gastrulation cell-type/TE-family associations (extraembryonic ectoderm→IAP/RLTR45; Apoa2+ endoderm→MER46C/RLTR20B3/LTRIS2; cardiomyocytes→L1ME3D; cardiac-fate cells→ETnERV3-int) is detected as cell-type-specific
- Previously reported positive controls reproduce, e.g. MERVL in mouse ESCs and LTR7/HERVH in human ESCs

## Limitations

- TE information content is lower than that of genes; TEs serve mainly as an addendum to gene information, so TE-only clustering may be weaker than gene-based clustering
- Marker assignments depend on the annotation mode: in exclusive mode TEs inside exon/UTR regions contribute only to genes, so marker-TE calls differ under --mode inclusive or nointron
- Inferring TEs as enhancer information in individual scATAC-seq cells remains speculative because single-cell peak data are sparse

## Evidence

- [abstract] scTE can discriminate cell type using chromatin accessibility of TEs alone when applied to single-cell ATAC-seq data: [evidence span withheld — anchor status: not_found]
- [methods] scTE command with tags: "$ scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR"
- [methods] exclusive allocation rationale: "The scTE algorithm will allocate 
reads first to gene exons, and then to TEs by default."
- [methods] RepeatMasker filter: "zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 >mm10.te.bed"
- [discussion] positive-control marker TEs: "we could observe previously identified phenomena such as MERVL and LTR7/HERVH expression in mouse and human ESCs, respectively."
- [readme] unfiltered BAM recommendation: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."

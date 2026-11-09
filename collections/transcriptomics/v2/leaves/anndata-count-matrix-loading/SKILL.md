---
name: anndata-count-matrix-loading
description: Use when starting an XClone run and you need the RDR read-depth matrix,
  the BAF AD/DP allelic matrices, and genome-mode feature annotation assembled into
  AnnData objects before run_RDR/run_BAF.
when_to_use_negative:
- Input is raw SAM/BAM/CRAM alignments rather than already-extracted count matrices
  — run the xcltk preprocessing pipeline (xcltk RDR and BAF modules) first.
- Data are single-cell DNA-seq (e.g. ACT/CHISEL-style inputs) rather than droplet-
  or well-based scRNA-seq counts, which is the modality XClone models.
- The task is downstream CNA-state calling or clonal clustering (XClustering, ARI
  vs Clone_ID) — that consumes the AnnData produced here, not produce it.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: XClone
  role: provides the data loaders (xclone.data.tnbc1_rdr/tnbc1_baf) and AnnData preprocessing
    helpers (xclone.pp.xclonedata, xclone.pp.load_anno) that assemble the count matrices
  repo: github.com/single-cell-genetics/XClone
- name: xcltk
  role: preprocessing toolkit that generates the AD, DP, and total-read-depth matrices
    from SAM/BAM/CRAM files before AnnData loading
  repo: hxj5/xcltk
- name: scanpy
  role: AnnData ecosystem used to handle the gene-by-cell count matrices loaded for
    the XClone workflow
provenance:
  source_task_ids:
  - task_008
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:60da909ba6ca8b3d44cd0bf0e3745f8dd5644d1b996d1cf7804bd4f0ae247872
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT
  - XClone
  - scanpy
  - statsmodels
  - xcltk
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# anndata-count-matrix-loading

## Summary

Loads XClone's three scRNA-seq input matrices — allele-specific AD and DP (BAF signals) and total read depth (RDR signals) — into AnnData objects with hg19/hg38 gene annotation, either from the bundled demo datasets (e.g. xclone.data.tnbc1_rdr()) or from xcltk-preprocessed mtx files.

## When to use

Use when starting an XClone run and you need the RDR read-depth matrix, the BAF AD/DP allelic matrices, and genome-mode feature annotation assembled into AnnData objects before run_RDR/run_BAF.

## When NOT to use

- Input is raw SAM/BAM/CRAM alignments rather than already-extracted count matrices — run the xcltk preprocessing pipeline (xcltk RDR and BAF modules) first.
- Data are single-cell DNA-seq (e.g. ACT/CHISEL-style inputs) rather than droplet- or well-based scRNA-seq counts, which is the modality XClone models.
- The task is downstream CNA-state calling or clonal clustering (XClustering, ARI vs Clone_ID) — that consumes the AnnData produced here, not produce it.

## Inputs

- AD.mtx and DP.mtx integer allelic count matrices (from xcltk preprocessing, e.g. GEO GSE148673 for TNBC1)
- total read depth matrix (RDR signals)
- cell barcode list / annotation (e.g. cell_type with reference cell type 'N')
- feature annotation via genome_mode ('hg19_genes', 'hg38_genes', 'hg38_blocks')
- demo AnnData objects xclone.data.tnbc1_rdr() and xclone.data.tnbc1_baf()

## Outputs

- RDR_adata AnnData with gene-by-cell read counts
- BAF_adata AnnData with integer 'AD' and 'DP' allelic layers
- annotated gene table (chr/start/stop/arm) suitable for run_RDR and run_BAF

## How to apply

Build the inputs either from the demo data loaders (RDR_adata = xclone.data.tnbc1_rdr(), BAF_adata = xclone.pp/tnbc1_baf() for the TNBC1 10x Genomics sample) or by running xcltk preprocessing on SAM/BAM/CRAM to produce AD.mtx, DP.mtx, and the total-read-depth matrix, then loading them with xclone.pp.xclonedata together with barcodes and genome_mode ('hg19_genes' or 'hg38_genes'). Attach feature annotation (chr/start/stop/arm) via xclone.pp.load_anno(genome_mode='hg38_genes'), and optionally hg38_blocks for genome-block information. The AD and DP layers must be integer allelic counts so the BAF module's three-step phasing (SNP-to-gene, EM gene bins of 100 genes, chromosome-arm dynamic programming) and beta-binomial mixture can operate. Correctly loaded objects are passed directly to xclone.XCloneConfig and xclone.model.run_RDR / run_BAF; save the module outputs as .h5ad (e.g. BAF_merge_Xdata.h5ad).

## Related tools

- **XClone** (provides the data loaders (xclone.data.tnbc1_rdr/tnbc1_baf) and AnnData preprocessing helpers (xclone.pp.xclonedata, xclone.pp.load_anno) that assemble the count matrices) — github.com/single-cell-genetics/XClone
- **xcltk** (preprocessing toolkit that generates the AD, DP, and total-read-depth matrices from SAM/BAM/CRAM files before AnnData loading) — hxj5/xcltk
- **scanpy** (AnnData ecosystem used to handle the gene-by-cell count matrices loaded for the XClone workflow)

## Examples

```
RDR_adata = xclone.data.tnbc1_rdr(); BAF_adata = xclone.data.tnbc1_baf(); hg38_genes = xclone.pp.load_anno(genome_mode='hg38_genes')
```

## Evaluation signals

- BAF_adata carries integer 'AD' and 'DP' layers over the same gene/barcode axes expected by run_BAF (5 configured allele-bias states downstream).
- Feature annotation is populated with chr/start/stop/arm columns after xclone.pp.load_anno with the chosen genome_mode ('hg19_genes'/'hg38_genes').
- Loaded objects match demo dataset dimensions (e.g. TNBC1 droplet-based 10x Genomics sample; BCH869 smart-seq2 glioma with 489 malignant cells and 3 non-tumour cells).
- Downstream calls xclone.model.run_RDR(RDR_adata, config_file=xconfig) and run_BAF(BAF_adata, config_file=xconfig) execute without axis- or layer-mismatch errors.

## Limitations

- AD/DP layers must be integer allelic counts; non-integer or bulk-aggregated inputs break the beta-binomial BAF modeling.
- Demo data availability is limited to the processed folders in Rongtingting/xclone-data (e.g. BCH869_scRNA, TNBC1_scRNA, ATC2_scRNA, GBM_10XsnRNA); other datasets require generating matrices with xcltk.
- XClone is under frequent update; README advises checking release history and keeping software current, so loader behavior may change between versions.

## Evidence

- [readme] It takes three matrices as input: the allele-specific AD and DP matrices (BAF signals) and the total read depth matrix (RDR signals).: [evidence span withheld — anchor status: external_doc]
- [methods] hg38_genes annotation loading: "hg38_genes = xclone.pp.load_anno(genome_mode = "hg38_genes")"
- [methods]  genome_mode options for default feature annotation: "use default feature annotation after you specify the genome_mode (include: "hg19_genes","

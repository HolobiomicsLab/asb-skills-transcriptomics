---
name: scrna-seq-pseudoalignment-quantification
description: Use when raw scRNA-seq FASTQ files from a 3'-end tag chemistry (e.g.
  10xv2/10xv3) must be converted to gene- and UTR isoform-level count matrices without
  whole-transcript alignment.
when_to_use_negative:
- The input is already a gene-by-cell count matrix (e.g. Cell Ranger filtered HDF5)
  and no 3'UTR isoform resolution is needed
- The library is not a 3'-end tag-based assay (e.g. full-length SMART-seq), since
  kallisto-bustools compatibility is stated for 3'-end tag scRNA-seq libraries
- De novo mapping of mRNA 3' cleavage sites from 10x data is the goal, since most
  10x reads do not span cleavage sites
edam_operation: http://edamontology.org/operation_3580
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scUTRquant
  role: Snakemake pipeline orchestrating pseudoalignment-based quantification of gene
    and 3'UTR isoform expression
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: 'Underlying workflow: kallisto bus pseudoalignment against UTRome targets
    plus bustools barcode correction'
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow engine executing the pipeline environments via Conda
- name: Conda (Miniforge)
  role: Installs the needed software for each pipeline step; libmamba solver recommended
    (>= v23.11)
  repo: https://github.com/conda-forge/miniforge#miniforge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:d5acf413f57a3c60b2a2af505b3a08d84c5f06d399fac8ebfb7bd8e106c09ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Conda (Miniforge)
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3580
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scrna-seq-pseudoalignment-quantification

## Summary

Rapid pseudoalignment-based quantification of gene and 3'UTR isoform expression from 3'-end tag scRNA-seq data using the kallisto-bustools workflow inside the scUTRquant Snakemake pipeline, avoiding full alignment while remaining compatible with 10x Genomics libraries.

## When to use

Use when raw scRNA-seq FASTQ files from a 3'-end tag chemistry (e.g. 10xv2/10xv3) must be converted to gene- and UTR isoform-level count matrices without whole-transcript alignment.

## When NOT to use

- The input is already a gene-by-cell count matrix (e.g. Cell Ranger filtered HDF5) and no 3'UTR isoform resolution is needed
- The library is not a 3'-end tag-based assay (e.g. full-length SMART-seq), since kallisto-bustools compatibility is stated for 3'-end tag scRNA-seq libraries
- De novo mapping of mRNA 3' cleavage sites from 10x data is the goal, since most 10x reads do not span cleavage sites

## Inputs

- Raw paired-end scRNA-seq FASTQ files (R1 barcode/read1, R2 cDNA/read2) from 10x Genomics 3' v2 or v3 libraries
- sample_sheet.csv with sample_id, file_type, and semicolon-separated R1/R2 FASTQ file lists
- config.yaml specifying dataset_name, sample_file, output_type, target UTRome (e.g. utrome_mm10_v2), tech, strand, bx_whitelist, min_umis
- Barcode whitelist file for bustools correct

## Outputs

- Gene-level and 3'UTR isoform-level SingleCellExperiment objects (data/sce/<target>/)
- Retained BUS/MTX count intermediates
- HTML QC reports per sample
- (Optionally) scverse AnnData objects as experimental output

## How to apply

Prepare a sample_sheet.csv (sample_id, file_type, semicolon-separated R1/R2 FASTQ lists per sample) and a dataset config.yaml naming dataset_name, output_type ['txs','genes'], target UTRome (e.g. utrome_mm10_v2), tech ('10xv2' or '10xv3'), strand '--fr-stranded' (required for all 10x 3'-end libraries), a barcode whitelist for bustools correct, and a min_umis cell-filtering threshold. Run `snakemake --use-conda --configfile examples/<dataset>/config.yaml` (Snakemake >= 6.0, Conda >= v23.11 with libmamba solver) so kallisto bus pseudoaligns reads against truncated UTRome targets and bustools corrects cell barcodes. Retrieve gene-level and isoform-level SingleCellExperiment objects from data/sce/<target>/ along with retained BUS/MTX intermediates and HTML QC reports. Correctness is judged by comparing gene counts to a reference pipeline: per-cell Spearman rho should exceed 0.99 versus Cell Ranger UMI counts, and per-gene rho should be around 0.92 on the 10x demo datasets.

## Related tools

- **scUTRquant** (Snakemake pipeline orchestrating pseudoalignment-based quantification of gene and 3'UTR isoform expression) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying workflow: kallisto bus pseudoalignment against UTRome targets plus bustools barcode correction) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow engine executing the pipeline environments via Conda)
- **Conda (Miniforge)** (Installs the needed software for each pipeline step; libmamba solver recommended (>= v23.11)) — https://github.com/conda-forge/miniforge#miniforge

## Examples

```
snakemake --use-conda --configfile examples/<dataset>/config.yaml
```

## Evaluation signals

- Per-cell Spearman rho > 0.99 between scUTRquant and Cell Ranger 3.0.0 gene-level UMI counts on the six mouse 10x 3' demo datasets
- Per-gene Spearman rho of approximately 0.92 versus Cell Ranger gene-level counts
- Output objects are valid Bioconductor SingleCellExperiment files present under data/sce/utrome_mm10_v2/
- Cells below the configured min_umis threshold are excluded from the final matrix
- HTML QC reports are generated for each processed sample

## Limitations

- Most 10x Genomics reads do not span mRNA 3' cleavage sites, so exact de novo CS mapping is not possible; quantification relies on a pre-built UTRome target set
- The strand flag '--fr-stranded' must be set for all 10X 3'-end libraries; omitting it gives wrong results
- Gene-level counts agree strongly with Cell Ranger (rho ~0.92 per gene) but are not identical, so downstream comparisons should expect small discrepancies

## Evidence

- [abstract] simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA: "We incorporate the CS annotations into a computational pipeline, called scUTRquant, for rapid, accurate, and simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA"
- [methods] pseudoalignment via kallisto bus: "`tech`: argument to `kallisto bus` indicating the scRNA-seq technology"
- [methods] barcode whitelist for bustools correct: "`bx_whitelist`: file of valid barcodes used in `bustools correct`"
- [discussion] built on kallisto-bustools for 3'-end tag libraries: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries"
- [readme] pipeline description: "A bioinformatics pipeline for single-cell 3' UTR isoform quantification."

---
name: scrna-seq-quality-control
description: Use when reprocessing 10x Genomics 3'-end scRNA-seq FASTQs with scUTRquant
  and you must verify that gene counts from a truncated UTRome reference remain concordant
  with Cell Ranger before trusting downstream UTR-isoform quantification.
when_to_use_negative:
- 'Data from full-length or non-3''-end scRNA-seq protocols: most reads from 10x Genomics
  data do not span mRNA 3'' end cleavage sites, so this truncated-UTRome quantification
  route (and its Cell Ranger concordance check) does not apply'
- The goal is de novo mapping of mRNA 3' ends from 10x data — scUTRquant quantifies
  against a pre-computed UTRome (e.g. utrome_mm10_v2), not cleavage sites discovered
  per dataset
- Libraries lacking a valid barcode whitelist, since bustools correct requires a file
  of valid barcodes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scUTRquant
  role: Snakemake pipeline that quantifies gene and 3'UTR isoform expression from
    10x 3' FASTQs against the utrome_mm10_v2 target; the gene-level output is the
    object under QC
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying pseudoalignment (kallisto bus) and barcode correction (bustools
    correct) engine producing the BUS counts that scUTRquant summarizes
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow engine executing the pipeline with Conda environments (>= 6.0, libmamba
    solver) via the dataset config.yaml
  repo: https://github.com/Mayrlab/scUTRquant
- name: Miniforge/Conda
  role: Environment manager recommended by the authors (>= v23.11 with libmamba solver)
    to install the scUTRquant software stack
  repo: https://github.com/conda-forge/miniforge#miniforge
- name: scUTRquant-demo
  role: Reference demonstration repository whose GitHub Actions workflows show the
    full end-to-end run used for this Cell Ranger comparison
  repo: https://github.com/mfansler/scUTRquant-demo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:c393f70b9781c55f5cf5f35a5d102d73845996ed7cab6b93d7775bb836d40e9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Miniforge/Conda
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  - scUTRquant-demo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# scrna-seq-quality-control

## Summary

A QC/benchmarking skill for validating that scUTRquant gene-level UMI counts agree with standard Cell Ranger quantification on 10x Genomics 3' scRNA-seq data, using per-cell and per-gene Spearman correlations as concordance metrics.

## When to use

Use when reprocessing 10x Genomics 3'-end scRNA-seq FASTQs with scUTRquant and you must verify that gene counts from a truncated UTRome reference remain concordant with Cell Ranger before trusting downstream UTR-isoform quantification.

## When NOT to use

- Data from full-length or non-3'-end scRNA-seq protocols: most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so this truncated-UTRome quantification route (and its Cell Ranger concordance check) does not apply
- The goal is de novo mapping of mRNA 3' ends from 10x data — scUTRquant quantifies against a pre-computed UTRome (e.g. utrome_mm10_v2), not cleavage sites discovered per dataset
- Libraries lacking a valid barcode whitelist, since bustools correct requires a file of valid barcodes

## Inputs

- 10x Genomics 3' demonstration FASTQ files (paired R1/R2, mouse)
- sample_sheet.csv with sample_id, file_type, semicolon-separated R1/R2 FASTQ lists per sample
- dataset config.yaml (dataset_name, sample_file, output_type, target 'utrome_mm10_v2', tech, strand, bx_whitelist, min_umis)
- Cell Ranger 3.0.0 filtered HDF5 (filtered_feature_bc_matrix.h5) for the same six datasets
- utrome_mm10_v2 UTRome target reference (from Mayrlab/scUTRquant-inputs)

## Outputs

- Gene-level SingleCellExperiment RDS objects in data/sce/utrome_mm10_v2/
- spearman_correlations.csv with per-sample per-cell and per-gene rho values
- Retained BUS/MTX intermediates
- HTML QC reports per sample

## How to apply

Download the six mouse 10x 3' demonstration FASTQ datasets and write a sample_sheet.csv with sample_id, file_type, and semicolon-separated R1/R2 FASTQ lists; then write a config.yaml specifying dataset_name, sample_file, output_type ['txs','genes'], target 'utrome_mm10_v2', tech ('10xv2' or '10xv3'), strand '--fr-stranded' (required for all 10X 3'-end libraries), a barcode whitelist for bustools correct, and a min_umis cell-filtering threshold. Execute the pipeline for all six samples with Snakemake >= 6.0 under Conda with the libmamba solver, then collect the gene-level SingleCellExperiment objects (readRDS) from data/sce/utrome_mm10_v2/. Extract raw UMI matrices from the Cell Ranger 3.0.0 filtered HDF5 files for the same datasets and compute per-cell and per-gene Spearman correlations, writing them to spearman_correlations.csv. The rationale is that scUTRquant pseudoaligns reads to truncated 3'UTR transcripts via kallisto-bustools rather than a standard transcriptome, so gene-level concordance against Cell Ranger is the primary QC gate for correctness of the UTRome-based quantification.

## Related tools

- **scUTRquant** (Snakemake pipeline that quantifies gene and 3'UTR isoform expression from 10x 3' FASTQs against the utrome_mm10_v2 target; the gene-level output is the object under QC) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying pseudoalignment (kallisto bus) and barcode correction (bustools correct) engine producing the BUS counts that scUTRquant summarizes) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow engine executing the pipeline with Conda environments (>= 6.0, libmamba solver) via the dataset config.yaml) — https://github.com/Mayrlab/scUTRquant
- **Miniforge/Conda** (Environment manager recommended by the authors (>= v23.11 with libmamba solver) to install the scUTRquant software stack) — https://github.com/conda-forge/miniforge#miniforge
- **scUTRquant-demo** (Reference demonstration repository whose GitHub Actions workflows show the full end-to-end run used for this Cell Ranger comparison) — https://github.com/mfansler/scUTRquant-demo

## Examples

```
snakemake --use-conda --configfile examples/<dataset>/config.yaml
```

## Evaluation signals

- Per-cell Spearman rho between scUTRquant and Cell Ranger 3.0.0 gene-level UMI counts exceeds 0.99 for every sample
- Per-gene Spearman rho reaches approximately 0.92 across the tested 10x Genomics datasets
- Gene-level SingleCellExperiment objects are present under data/sce/utrome_mm10_v2/ and readable via readRDS
- spearman_correlations.csv is produced with per-sample rho entries and no missing samples
- HTML QC reports and BUS/MTX intermediates exist for all six demo samples, confirming the Snakemake DAG completed

## Limitations

- Per-gene correlation (~0.92) is lower than per-cell (>0.99), so gene-level ranks are concordant but not identical to Cell Ranger
- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, which prevents exact de novo mapping of mRNA 3' ends and constrains the comparison to quantification concordance
- The strand flag '--fr-stranded' is required for all 10X 3'-end libraries; omitting it or using the wrong tech ('10xv2' vs '10xv3') chemistry will degrade counts
- The repo appears as workflow_step 'snakemake --use-conda --configfile examples/<dataset>/config.yaml' with no sections — treat any claim about demo results with caution

## Evidence

- [methods] min_umis filter parameter: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
- [methods] barcode whitelist for bustools correct: "`bx_whitelist`: file of valid barcodes used in `bustools correct`"
- [discussion] kallisto-bustools foundation: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification"
- [intro] 10x data limitation: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS)"
- [readme] demo repository end-to-end validation: [evidence span withheld — anchor status: external_doc]

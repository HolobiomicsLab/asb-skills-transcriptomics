---
name: perturb-seq-bam-processing-with-snakemake
description: Use when starting from raw 3'-end tag-based scRNA-seq or Perturb-seq
  BAM files (10x-style chemistries) and you need 3'UTR isoform-resolved counts rather
  than gene counts, as the first step before pseudobulk aggregation and zdWUI testing.
when_to_use_negative:
- When the analysis goal only needs gene-level counts (no 3'UTR isoform resolution),
  a standard gene-count workflow is sufficient.
- When the data are not 3'-end tag-based scRNA-seq — the kallisto-bustools approach
  is built for 3'-end tag libraries, not full-length protocols.
- When exact de novo mapping of mRNA 3' ends from 10x reads is required — most 10x
  reads do not span cleavage sites, so this pipeline quantifies against a fixed CS
  catalog instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scUTRquant
  role: Snakemake pipeline that runs the whole BAM-to-3'UTR-isoform quantification
    workflow
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Pseudoaligns reads with kallisto bus and corrects barcodes with bustools correct
    during quantification
- name: Snakemake
  role: Workflow engine orchestrating the pipeline, using Conda to install needed
    software
- name: Conda/Miniforge
  role: Environment management for reproducible pipeline execution (Conda >=v23.11
    recommended for libmamba solver)
  repo: https://github.com/conda-forge/miniforge#miniforge
- name: scUTRquant-demo
  role: Reference implementation demonstrating end-to-end runs from raw data to SingleCellExperiment/AnnData
    outputs
  repo: https://github.com/mfansler/scUTRquant-demo
- name: scUTRquant-inputs
  role: Provides input sample sheets and raw-data download scripts for pipeline runs
  repo: https://github.com/Mayrlab/scUTRquant-inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:ce73227907ba074583480b4a86a55189670d4ebc7b69525405ccd1053687729c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Conda/Miniforge
  - Snakemake
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  - scUTRquant-demo
  - scUTRquant-inputs
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# perturb-seq-bam-processing-with-snakemake

## Summary

Reprocess raw 3'-end scRNA-seq/Perturb-seq BAM inputs (e.g., the 48 K562 Perturb-seq BAMs) through the scUTRquant Snakemake pipeline to obtain gene-level and 3'UTR isoform SingleCellExperiment objects, using kallisto-bustools pseudoalignment and min_umis cell filtering.

## When to use

Use when starting from raw 3'-end tag-based scRNA-seq or Perturb-seq BAM files (10x-style chemistries) and you need 3'UTR isoform-resolved counts rather than gene counts, as the first step before pseudobulk aggregation and zdWUI testing.

## When NOT to use

- When the analysis goal only needs gene-level counts (no 3'UTR isoform resolution), a standard gene-count workflow is sufficient.
- When the data are not 3'-end tag-based scRNA-seq — the kallisto-bustools approach is built for 3'-end tag libraries, not full-length protocols.
- When exact de novo mapping of mRNA 3' ends from 10x reads is required — most 10x reads do not span cleavage sites, so this pipeline quantifies against a fixed CS catalog instead.

## Inputs

- 48 K562 Perturb-seq BAM files (10x 3'-end tag-based scRNA-seq)
- Sample sheet / input configuration for scUTRquant
- UTRome target annotation (utrome_hg38_v1) built from HCL-derived cleavage sites plus GENCODE
- Barcode whitelist (bx_whitelist) for bustools correct

## Outputs

- Gene-level SingleCellExperiment object
- 3'UTR isoform SingleCellExperiment object
- AnnData (scverse) output (experimental, per scUTRquant-demo)

## How to apply

Clone and run the scUTRquant Snakemake pipeline in a Conda/Mamba (or Singularity) environment; Snakemake uses Conda to install dependencies, and Conda >=v23.11 is recommended for its faster libmamba solver. Configure the target (e.g., 'utrome_hg38_v1', which uses the Human Cell Landscape-derived UTRome annotations) and supply the sample sheet with the 48 K562 Perturb-seq BAM inputs; reads are pseudoaligned with kallisto bus using the correct 'tech' argument for the library chemistry, and cell barcodes are corrected against the 'bx_whitelist' via bustools correct. Apply the 'min_umis' filter so that cells below the minimum UMI count are excluded, producing gene-level and 3'UTR isoform SingleCellExperiment objects (AnnData output is experimental). The rationale is that quantification must be against truncated-transcript CS annotations because most 10x Genomics reads do not span 3' end cleavage sites, preventing de novo mapping.

## Related tools

- **scUTRquant** (Snakemake pipeline that runs the whole BAM-to-3'UTR-isoform quantification workflow) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Pseudoaligns reads with kallisto bus and corrects barcodes with bustools correct during quantification)
- **Snakemake** (Workflow engine orchestrating the pipeline, using Conda to install needed software)
- **Conda/Miniforge** (Environment management for reproducible pipeline execution (Conda >=v23.11 recommended for libmamba solver)) — https://github.com/conda-forge/miniforge#miniforge
- **scUTRquant-demo** (Reference implementation demonstrating end-to-end runs from raw data to SingleCellExperiment/AnnData outputs) — https://github.com/mfansler/scUTRquant-demo
- **scUTRquant-inputs** (Provides input sample sheets and raw-data download scripts for pipeline runs) — https://github.com/Mayrlab/scUTRquant-inputs

## Examples

```
snakemake --use-conda --cores 8 utrome_hg38_v1
```

## Evaluation signals

- The run completes the specified target (e.g., utrome_hg38_v1) without rule failures in the Snakemake Conda/Singularity environment.
- Outputs are Bioconductor SingleCellExperiment objects containing both gene-level and 3'UTR isoform count matrices for all 48 K562 Perturb-seq samples.
- Cells below the min_umis threshold are absent from the final objects (verify per-cell UMI counts >= min_umis).
- Barcode correction was applied against the provided bx_whitelist (bustools correct ran in the workflow).
- A demo run via scUTRquant-demo reproduces the expected output structure, starting from raw scRNA-seq data.

## Limitations

- Most 10x Genomics reads do not span mRNA 3' end cleavage sites, so exact de novo CS mapping is impossible; quantification is limited to the pre-defined UTRome CS set.
- The pipeline is designed for 3'-end tag scRNA-seq libraries and may not suit full-length or custom 3' end protocols.
- The provided scUTRquant README is truncated, so exact CLI flags and config schema beyond the documented keys (tech, bx_whitelist, min_umis, exclude_unannotated_cells) are not fully documented here.

## Evidence

- [methods] kallisto tech argument: "`tech`: argument to `kallisto bus` indicating the scRNA-seq technology"
- [methods] barcode whitelist: "`bx_whitelist`: file of valid barcodes used in `bustools correct`"
- [methods] min_umis filter: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
- [methods] conda dependency install: "Snakemake can use Conda to install the needed software"
- [readme] demo outputs: [evidence span withheld — anchor status: external_doc]
- [readme] gwps-sq purpose: [evidence span withheld — anchor status: external_doc]

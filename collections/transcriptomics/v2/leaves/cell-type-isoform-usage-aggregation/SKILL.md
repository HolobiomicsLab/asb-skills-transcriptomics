---
name: cell-type-isoform-usage-aggregation
description: Use when SingleCellExperiment 3'UTR isoform count matrices (*.txs.Rds,
  *.genes.Rds) from scUTRquant must be summarized per cell type and genes labeled
  multi-UTR vs IPA under a ≥10% isoform-fraction cutoff.
when_to_use_negative:
- Input is a standard per-cell gene-count matrix without 3'UTR isoform resolution
  — tandem isoform usage and IPA cannot be tabulated
- Fewer than 200 cells per cell type are available, so the cell-type retention threshold
  cannot be met
- Goal is de novo cleavage site discovery from Microwell-seq data (hcl-utrome / mca-utrome
  pipelines) rather than summarizing isoform usage of already-quantified cells
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scUTRquant
  role: Snakemake pipeline that quantifies gene and 3'UTR isoform expression into
    sparse SingleCellExperiment count matrices aggregated by cell type
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Backend pseudoalignment (kallisto bus) against the UTRome index and barcode
    correction (bustools correct) feeding the isoform count matrices
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow engine executing the scUTRquant pipeline in Conda or Singularity
    mode from config.yaml and sample_sheet.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:8c095322fae76e3fc8b75e5cdb5a78c3789df59cfbbd242e9d8e333fd30eb23b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-isoform-usage-aggregation

## Summary

Aggregates scUTRquant 3'UTR isoform counts into per-cell-type usage profiles (retaining cell types with ≥200 cells) and classifies expressed protein-coding genes as multi-UTR or intronic polyadenylation (IPA) genes using a ≥10% isoform-fraction criterion across Tabula Sapiens / Tabula Muris data.

## When to use

Use when SingleCellExperiment 3'UTR isoform count matrices (*.txs.Rds, *.genes.Rds) from scUTRquant must be summarized per cell type and genes labeled multi-UTR vs IPA under a ≥10% isoform-fraction cutoff.

## When NOT to use

- Input is a standard per-cell gene-count matrix without 3'UTR isoform resolution — tandem isoform usage and IPA cannot be tabulated
- Fewer than 200 cells per cell type are available, so the cell-type retention threshold cannot be met
- Goal is de novo cleavage site discovery from Microwell-seq data (hcl-utrome / mca-utrome pipelines) rather than summarizing isoform usage of already-quantified cells

## Inputs

- FASTQ/BAM scRNA-seq inputs listed in sample_sheet.csv (e.g., Tabula Sapiens / Tabula Muris 10x 3'-end libraries)
- Prebuilt UTRome kallisto index (utrome_hg38_v1 or utrome_mm10_v1) and barcode whitelist
- Snakemake config.yaml with target, tech, strand, and min_umis settings
- Per-cell 3'UTR isoform count SingleCellExperiment objects (*.txs.Rds and *.genes.Rds) from scUTRquant
- Cell-type annotations per cell

## Outputs

- Per-cell-type 3'UTR isoform usage profiles (cell types with ≥200 cells only)
- gene_utr_classification.csv with per-gene multi-UTR / IPA labels and category counts
- Expected category counts: mouse 6766/16195 multi-UTR and 1869 IPA; human 8129/16185 multi-UTR and 4113 IPA (Supplementary Data 3-4)

## How to apply

First run the scUTRquant Snakemake pipeline (Conda or Singularity mode) with a configfile (config.yaml) and sample_sheet.csv, setting `target` to utrome_hg38_v1 or utrome_mm10_v1, `tech` to the matching 10x chemistry, and `strand` to --fr-stranded for 10X 3'-end libraries, so kallisto bus pseudoaligns reads against the UTRome index and bustools corrects barcodes against the whitelist. Apply cell-level filtering with the `min_umis` threshold to produce sparse 3'UTR isoform count SingleCellExperiment objects. Aggregate isoform usage across cells belonging to each cell type, retaining only cell types with ≥200 cells. Classify each expressed gene by isoform fraction with the ≥10% cutoff — the UTRome atlas logic tabulates how many distinct cell types use a particular tandem (non-intronic) isoform at 5% or 10% frequency, and genes with more than two isoforms of this type are multi-UTR genes, while IPA genes are called separately from intronic isoforms. Tabulate per-gene categories in gene_utr_classification.csv and validate against Supplementary Data 3–4 counts.

## Related tools

- **scUTRquant** (Snakemake pipeline that quantifies gene and 3'UTR isoform expression into sparse SingleCellExperiment count matrices aggregated by cell type) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Backend pseudoalignment (kallisto bus) against the UTRome index and barcode correction (bustools correct) feeding the isoform count matrices) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow engine executing the scUTRquant pipeline in Conda or Singularity mode from config.yaml and sample_sheet.csv)

## Examples

```
snakemake --use-conda --cores 8 --configfile config.yaml
```

## Evaluation signals

- gene_utr_classification.csv mouse counts must equal 6766 multi-UTR of 16195 expressed protein-coding genes and 1869 IPA genes
- Human validation: 8129/16185 multi-UTR genes and 4113 IPA genes (Supplementary Data 3-4)
- Every retained cell type in the aggregated usage table has ≥200 cells
- Multi-UTR labels correspond to genes with more than two tandem (non-intronic) isoforms used at the 5% or 10% frequency threshold across distinct cell types
- Per-target outputs (*.txs.Rds, *.genes.Rds) exist for the specified target (utrome_hg38_v1 or utrome_mm10_v1) and chemistry

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so cleavage sites come from the curated UTRome rather than de novo mapping of the quantified library
- Cell types with fewer than 200 cells are dropped from the aggregated usage analysis
- Multi-UTR classification is based on tandem (non-intronic) isoforms only; intronic polyadenylation isoforms are categorized separately as IPA
- MWS-derived cleavage sites required quality control and filtering for internal priming before use

## Evidence

- [readme] At its core is a tabulation of how many distinct cell types use a particular tandem (non-intronic) isoform at either 5% or 10% frequency.: [evidence span withheld — anchor status: external_doc]
- [methods] `min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
- [discussion] our large-scale analysis on cell type-specific CS usage allowed us to categorize CS into major and minor sites: "our large-scale analysis on cell type-specific CS usage allowed us to categorize CS into major and minor sites"
- [methods] Snakemake can use Conda to install the needed software: "Snakemake can use Conda to install the needed software"

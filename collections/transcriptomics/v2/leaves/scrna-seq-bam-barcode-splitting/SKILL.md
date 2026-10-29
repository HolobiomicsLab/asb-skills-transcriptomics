---
name: scrna-seq-bam-barcode-splitting
description: Use when input is one 10x Genomics barcoded BAM file (e.g., pbmc_8k)
  plus barcodes.tsv.gz and per-cell BAM files are required for downstream per-cell
  analyses such as geneBodyCoverage.pl.
when_to_use_negative:
- The input BAM files are already split by cell — README states 'If BAM files are
  split by cells already, you can skip this step.'
- The input is not a 10x Genomics barcoded BAM (e.g., non-10x datasets like E-MTAB-2600
  SMARTer data) — this script is specific to the 10x barcode format.
- The barcode list does not match the barcodes embedded in the BAM — a mismatch produces
  empty gene body coverage plots downstream.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: Provides 0_split10XbyBarcode.sh and split10XbyBarcode.pl, the scripts that
    split the barcoded 10x BAM into per-cell BAM files
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:6215301ca4648128ea2bc8246785458092c6d31edbc9653769e65880fe1b04fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - docker
  - singularity
  - udocker
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scrna-seq-bam-barcode-splitting

## Summary

Splits a single 10x Genomics barcoded BAM file into per-cell BAM files, one per cell barcode, as the prerequisite step before per-cell gene body coverage analysis with SkewC. Uses the barcodes.tsv.gz from the 10x outs/filtered_feature_bc_matrix/ directory as the barcode reference.

## When to use

Use when input is one 10x Genomics barcoded BAM file (e.g., pbmc_8k) plus barcodes.tsv.gz and per-cell BAM files are required for downstream per-cell analyses such as geneBodyCoverage.pl.

## When NOT to use

- The input BAM files are already split by cell — README states 'If BAM files are split by cells already, you can skip this step.'
- The input is not a 10x Genomics barcoded BAM (e.g., non-10x datasets like E-MTAB-2600 SMARTer data) — this script is specific to the 10x barcode format.
- The barcode list does not match the barcodes embedded in the BAM — a mismatch produces empty gene body coverage plots downstream.

## Inputs

- 10x Genomics barcoded BAM file (e.g., pbmc_8k bam from outs/)
- barcodes.tsv.gz from outs/filtered_feature_bc_matrix/
- output directory path for split BAM files (default 'input')

## Outputs

- Multiple per-cell BAM files, one per cell barcode, written to the output directory (e.g., input/example.TTTGTCATCTAACGGT-1.bam)

## How to apply

Run bash 0_split10XbyBarcode.sh with three arguments: the 10x BAM file, the barcodes.tsv.gz located under the 10x outs/filtered_feature_bc_matrix/ directory, and an output directory (default 'input'). The script uses split10XbyBarcode.pl to emit multiple BAM files, one per cell barcode, named like example.TTTGTCATCTAACGGT-1.bam. This step is needed because 10x Genomics outputs a single BAM file while per-cell analyses such as geneBodyCoverage.pl require separate per-cell BAMs. Note that geneBodyCoverage.pl does not require BAM index files (unlike RSeQC geneBody_coverage.py), so indexing of the split BAMs can be skipped. If the BAM is already split by cell, this step can be omitted.

## Related tools

- **SkewC** (Provides 0_split10XbyBarcode.sh and split10XbyBarcode.pl, the scripts that split the barcoded 10x BAM into per-cell BAM files) — https://github.com/LSBDT/SkewC.git

## Examples

```
bash 0_split10XbyBarcode.sh example/example.bam example/barcodes.tsv.gz input
```

## Evaluation signals

- Output directory contains one BAM file per barcode listed in barcodes.tsv.gz
- Split BAM file names follow the pattern <original>.<BARCODE>.bam (e.g., example.TTTGTCATCTAACGGT-1.bam)
- Each split BAM contains reads whose 10x barcode tag matches the corresponding barcode in barcodes.tsv.gz
- No empty per-cell BAM files, which would indicate a mismatch between barcode.tsv names and barcodes in the BAM

## Limitations

- Barcode mismatch between barcode.tsv and the barcoded BAM file yields empty downstream gene body coverage PDFs — verify barcode consistency before splitting.
- The script is specific to 10x Genomics output structure (outs/filtered_feature_bc_matrix/barcodes.tsv.gz); non-10x data requires a different splitting approach.
- This step cannot be run inside the geneBodyCoverage.sh workflow blindly — geneBodyCoverage.pl must not be run in parallel while creating its reference index file.

## Evidence

- [abstract] SkewC will split the barcoded BAM into multiple BAM files (one BAM file per cell barcode).: "SkewC will split the barcoded BAM into multiple BAM files (one BAM file per cell barcode)."
- [methods] bash 0_split10XbyBarcode.sh $bam $barcode $outdir: "bash 0_split10XbyBarcode.sh $bam $barcode $outdir"
- [readme] Before doing gene body coverage analysis, there is a need to split BAM files by cells.: "Before doing gene body coverage analysis, there is a need to split BAM files by cells."
- [readme] $barcode - barcodes.tsv.gz under 10XGenomics outs/filtered_feature_bc_matrix/: "$barcode - barcodes.tsv.gz under 10XGenomics outs/filtered_feature_bc_matrix/"
- [readme] RSeQC 'geneBody_coverage.py' required bam files to be indexed, but geneBodyCoverage.pl doesn't need index files.: "RSeQC 'geneBody_coverage.py' required bam files to be indexed, but geneBodyCoverage.pl doesn't need index files."

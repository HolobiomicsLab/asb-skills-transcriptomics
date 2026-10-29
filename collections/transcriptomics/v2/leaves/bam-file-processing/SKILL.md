---
name: bam-file-processing
description: Use when input is a single 10x Genomics barcoded BAM plus a barcodes.tsv.gz
  from outs/filtered_feature_bc_matrix/ and per-cell BAM files are needed for gene
  body coverage or per-cell QC.
when_to_use_negative:
- BAM files are already split by cells — the README explicitly says you can skip this
  step
- Input is a non-10x dataset with per-cell BAMs already (e.g. E-MTAB-2600 style smart-seq
  data)
- No barcodes.tsv.gz is available or it mismatches the barcodes in the BAM (produces
  empty gene body coverage plots)
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: Provides 0_split10XbyBarcode.sh and bin/split10XbyBarcode.pl to split the
    barcoded BAM into per-cell BAM files
  repo: https://github.com/LSBDT/SkewC.git
- name: Perl
  role: Language in which split10XbyBarcode.pl (and the rest of SkewC) is implemented
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:80e8b5902fbf90cae16f23a5629723aa48812de691a699c05793695afc6877c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - Perl
  - R
  - SkewC
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bam-file-processing

## Summary

Splits a single 10x Genomics barcoded BAM file into one BAM file per cell barcode using SkewC's 0_split10XbyBarcode.sh (bin/split10XbyBarcode.pl), a prerequisite step before SkewC's gene body coverage computation.

## When to use

Use when input is a single 10x Genomics barcoded BAM plus a barcodes.tsv.gz from outs/filtered_feature_bc_matrix/ and per-cell BAM files are needed for gene body coverage or per-cell QC.

## When NOT to use

- BAM files are already split by cells — the README explicitly says you can skip this step
- Input is a non-10x dataset with per-cell BAMs already (e.g. E-MTAB-2600 style smart-seq data)
- No barcodes.tsv.gz is available or it mismatches the barcodes in the BAM (produces empty gene body coverage plots)

## Inputs

- 10x Genomics barcoded BAM file (e.g. TestData/10xGenomics/barcoded.bam)
- barcodes.tsv.gz from 10x Genomics outs/filtered_feature_bc_matrix/
- output directory path (default 'input')

## Outputs

- One per-cell BAM file per barcode, written under the specified output directory

## How to apply

Run 'bash 0_split10XbyBarcode.sh $bam $barcode $outdir' from the SkewC root directory, passing the 10x barcoded BAM, the compressed barcode list (barcodes.tsv.gz), and an output directory (default 'input'). The script is implemented by the Perl program bin/split10XbyBarcode.pl, which iterates over each barcode in barcodes.tsv.gz and extracts reads carrying that cell barcode into a separate BAM file under the output directory. If BAMs are already split per cell, skip this step entirely. After the run, count the produced BAM files and verify the count equals the number of entries in barcodes.tsv; a barcode mismatch between barcodes.tsv and the BAM yields empty downstream gene body coverage plots.

## Related tools

- **SkewC** (Provides 0_split10XbyBarcode.sh and bin/split10XbyBarcode.pl to split the barcoded BAM into per-cell BAM files) — https://github.com/LSBDT/SkewC.git
- **Perl** (Language in which split10XbyBarcode.pl (and the rest of SkewC) is implemented)

## Examples

```
bash 0_split10XbyBarcode.sh TestData/10xGenomics/barcoded.bam barcodes.tsv.gz input
```

## Evaluation signals

- The split script runs without error and produces BAM files under the chosen output directory
- The number of generated per-cell BAM files equals the number of entries in barcodes.tsv
- Each output BAM filename corresponds to a cell barcode from barcodes.tsv.gz
- Downstream 1_geneBodyCoverage.sh receives one BAM per cell (barcoded BAM mismatch would yield empty PDF coverage plots)

## Limitations

- A mismatch between barcode names in barcode.tsv and the barcodes in the barcoded BAM results in empty gene body coverage PDF files
- Multiple BAMs (separated by cell) go into a default 'input' directory if $outdir is omitted — may collide with other uses of that directory name
- Indexing/coverage steps downstream should not be run in parallel while geneBodyCoverage.pl is creating its reference index (5–10 minutes)

## Evidence

- [methods] Split step command and args: "bash 0_split10XbyBarcode.sh $bam $barcode $outdir"
- [readme] Split purpose: "Since 10x Genomics output one BAM file, split step is needed. If BAM files are split by cells already, you can skip this step."
- [abstract] Per-cell BAM creation: "SkewC will split the barcoded BAM into multiple BAM files (one BAM file per cell barcode)."
- [discussion] Barcode mismatch failure mode: "It occurs when there is a mis-match between the barcode names in the barcode.tsv file and the barcodes in the barcoded BAM file."
- [readme] Barcode list provenance: "$barcode - barcodes.tsv.gz under 10XGenomics outs/filtered_feature_bc_matrix/"

---
name: 10x-genomics-output-format-handling
description: Use when input data comes from a 10x Genomics run consisting of one barcoded
  BAM plus a barcodes.tsv.gz from outs/filtered_feature_bc_matrix/, and per-cell BAM
  files are needed before downstream per-cell analysis.
when_to_use_negative:
- The BAM files are already split by cells (skip 0_split10XbyBarcode.sh as documented).
- Input is a non-10x scRNA-seq dataset (e.g. E-MTAB-2600-style SMARTer data) that
  already arrives as one BAM per cell.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: SkewC
  role: Provides the 0_split10XbyBarcode.sh / bin/split10XbyBarcode.pl split step
    that converts the single 10x barcoded BAM into per-cell BAM files
  repo: https://github.com/LSBDT/SkewC.git
- name: Perl
  role: Implementation language of bin/split10XbyBarcode.pl, which extracts reads
    per cell barcode from the 10x BAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:f22e972a96727c7dd2711def0b0fcdc558d20e8b2f6536e767f930ad45ff66dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - Perl
  - SkewC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# 10x-genomics-output-format-handling

## Summary

Handles the standard 10x Genomics output bundle — the single barcoded BAM file and the barcodes.tsv.gz list under outs/filtered_feature_bc_matrix/ — by splitting it into one BAM per cell barcode using SkewC's split step, a prerequisite for per-cell gene body coverage analysis.

## When to use

Use when input data comes from a 10x Genomics run consisting of one barcoded BAM plus a barcodes.tsv.gz from outs/filtered_feature_bc_matrix/, and per-cell BAM files are needed before downstream per-cell analysis.

## When NOT to use

- The BAM files are already split by cells (skip 0_split10XbyBarcode.sh as documented).
- Input is a non-10x scRNA-seq dataset (e.g. E-MTAB-2600-style SMARTer data) that already arrives as one BAM per cell.

## Inputs

- 10x Genomics barcoded BAM file (single BAM containing all cells)
- barcodes.tsv.gz file from 10x Genomics outs/filtered_feature_bc_matrix/ directory
- Output directory path (default='input')

## Outputs

- One BAM file per cell barcode, named e.g. example.TTTGTCATCTAACGGT-1.bam, written under the specified output directory (default 'input')

## How to apply

Load the 10x Genomics barcoded BAM (e.g. TestData/10xGenomics/barcoded.bam) and the compressed barcode list barcodes.tsv.gz from the outs/filtered_feature_bc_matrix/ directory. Run SkewC's split step 'bash 0_split10XbyBarcode.sh $bam $barcode $outdir', which is implemented by the Perl script bin/split10XbyBarcode.pl; the $outdir argument can be omitted to use the default 'input' directory. The script iterates over each barcode in barcodes.tsv.gz, extracts the reads carrying that cell barcode from the single 10x BAM, and writes one per-cell BAM file into the output directory. If the BAM files are already split by cell, this step can be skipped entirely. Note that geneBodyCoverage.pl does not require indexed BAMs, so no samtools indexing is needed before the gene body coverage step.

## Related tools

- **SkewC** (Provides the 0_split10XbyBarcode.sh / bin/split10XbyBarcode.pl split step that converts the single 10x barcoded BAM into per-cell BAM files) — https://github.com/LSBDT/SkewC.git
- **Perl** (Implementation language of bin/split10XbyBarcode.pl, which extracts reads per cell barcode from the 10x BAM)

## Examples

```
bash 0_split10XbyBarcode.sh example/example.bam example/barcodes.tsv.gz
```

## Evaluation signals

- The split script runs without error on the barcoded BAM and barcodes.tsv.gz.
- The number of generated per-cell BAM files equals the number of entries in barcodes.tsv.
- Each output BAM file is named after exactly one cell barcode from the barcode list (e.g. example.TTTGTCATCTAACGGT-1.bam).
- All per-cell BAM files appear under the specified output directory (default 'input').

## Limitations

- A mismatch between barcode names in barcodes.tsv and barcodes in the barcoded BAM produces empty downstream gene body coverage plots (empty PDF files).
- The script is specific to 10x Genomics output; the barcodes.tsv.gz must come from outs/filtered_feature_bc_matrix/.
- Barcode format issues in either file require re-formatting with a text editor and re-running step 1 (split barcoded BAM file) before continuing the SkewC workflow.

## Evidence

- [readme] Split script inputs are the barcoded BAM and barcode list: "bash 0_split10XbyBarcode.sh $bam $barcode $outdir"
- [readme] 10x outputs a single BAM so splitting is required: "Since 10x Genomics output one BAM file, split step is needed. * If BAM files are split by cells already, you can skip this step."
- [methods] Barcode list location in the 10x output tree: "$barcode - barcodes.tsv.gz under 10XGenomics outs/filtered_feature_bc_matrix/"
- [readme] Default output directory for split BAMs: "$outdir  - directory to store split BAM files (default='input')"
- [abstract] SkewC splits the barcoded BAM into per-cell BAMs: "SkewC will split the barcoded BAM into multiple BAM files (one BAM file per cell barcode)."

---
name: bam-file-splitting-by-cell-barcode
description: Use when the input is a single 10x Genomics barcoded BAM (e.g. possorted_genome_bam.bam)
  plus a barcodes.tsv.gz from outs/filtered_feature_bc_matrix/, and per-cell BAM files
  are needed before gene body coverage analysis.
when_to_use_negative:
- BAM files are already split by cells — the README states 'If BAM files are split
  by cells already, you can skip this step.'
- Input is not a 10x Genomics barcoded BAM (e.g. non-10x datasets like E-MTAB-2600
  that already come as per-cell files)
- You do not intend to use SkewC's geneBodyCoverage.pl pipeline at all (the split
  is only needed to feed per-cell gene body coverage computation)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: SkewC
  role: Provides the 0_split10XbyBarcode.sh batch command that wraps the split step
  repo: https://github.com/LSBDT/SkewC.git
- name: Perl
  role: Implementation language of bin/split10XbyBarcode.pl, which extracts per-barcode
    reads from the 10x BAM
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:d5d6993a6037811e5ba73b272acace9deb9a5229d6d34032474d4109cdf19770
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - Perl
  - R
  - SkewC
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bam-file-splitting-by-cell-barcode

## Summary

Splits a single 10x Genomics barcoded BAM file into one BAM file per cell barcode using SkewC's 0_split10XbyBarcode.sh (implemented by Perl script bin/split10XbyBarcode.pl), producing per-cell BAMs required for downstream per-cell gene body coverage computation.

## When to use

Use when the input is a single 10x Genomics barcoded BAM (e.g. possorted_genome_bam.bam) plus a barcodes.tsv.gz from outs/filtered_feature_bc_matrix/, and per-cell BAM files are needed before gene body coverage analysis.

## When NOT to use

- BAM files are already split by cells — the README states 'If BAM files are split by cells already, you can skip this step.'
- Input is not a 10x Genomics barcoded BAM (e.g. non-10x datasets like E-MTAB-2600 that already come as per-cell files)
- You do not intend to use SkewC's geneBodyCoverage.pl pipeline at all (the split is only needed to feed per-cell gene body coverage computation)

## Inputs

- Single 10x Genomics barcoded BAM file (e.g. possorted_genome_bam.bam from outs/)
- Compressed barcode list barcodes.tsv.gz from 10x outs/filtered_feature_bc_matrix/

## Outputs

- One BAM file per cell barcode, stored in the specified output directory (default 'input')
- Count of per-cell BAM files equal to the number of barcodes in barcodes.tsv

## How to apply

Run 'bash 0_split10XbyBarcode.sh $bam $barcode $outdir' from the SkewC root directory, supplying the 10x barcoded BAM, the compressed barcode list (barcodes.tsv.gz), and an output directory (default='input'; the argument can be omitted if 'input' is acceptable). For each barcode entry in barcodes.tsv.gz, the Perl script bin/split10XbyBarcode.pl extracts the reads carrying that cell barcode and writes one per-cell BAM file into the output directory. If the BAM files are already split by cells, this step can be skipped entirely. The rationale is that SkewC's gene body coverage step (1_geneBodyCoverage.sh, e.g. with species 'mm10') operates on individual cell BAM files, so a single 10x BAM must be decomposed first; note that geneBodyCoverage.pl does not require BAM index files, unlike RSeQC's geneBody_coverage.py.

## Related tools

- **SkewC** (Provides the 0_split10XbyBarcode.sh batch command that wraps the split step) — https://github.com/LSBDT/SkewC.git
- **Perl** (Implementation language of bin/split10XbyBarcode.pl, which extracts per-barcode reads from the 10x BAM) — https://github.com/LSBDT/SkewC.git

## Examples

```
bash 0_split10XbyBarcode.sh example/example.bam example/barcodes.tsv.gz input
```

## Evaluation signals

- Script 0_split10XbyBarcode.sh completes without error
- Number of generated per-cell BAM files under the output directory equals the number of entries in barcodes.tsv
- Each produced BAM file corresponds to exactly one barcode from barcodes.tsv.gz
- Downstream 1_geneBodyCoverage.sh runs on the split BAMs without barcode-mismatch errors (no empty gene body coverage PDFs)
- Output directory defaults to 'input' when the $outdir argument is omitted

## Limitations

- Barcode mismatch between barcodes.tsv and the barcodes in the barcoded BAM results in empty gene body coverage output (empty PDF files) downstream
- The script is specific to 10x Genomics output structure (barcodes.tsv.gz under outs/filtered_feature_bc_matrix/)
- SkewC setup requires git and at least one of docker, udocker, or singularity installed before any batch command (including the split step) can run

## Evidence

- [abstract] split barcoded BAM file into per-cell BAM files: "SkewC will split the barcoded BAM into multiple BAM files (one BAM file per cell barcode)."
- [methods] split command and arguments: "bash 0_split10XbyBarcode.sh $bam $barcode $outdir"
- [readme] readme split rationale: "Before doing gene body coverage analysis, there is a need to split BAM files by cells."
- [readme] readme barcode source file: "$barcode - barcodes.tsv.gz under 10XGenomics outs/filtered_feature_bc_matrix/"
- [results] task_001 workflow split step: [evidence span withheld — anchor status: not_found]

---
name: single-cell-rnaseq-quality-assessment
description: Use when a 10X Genomics barcoded BAM file (with barcodes.tsv.gz from
  outs/filtered_feature_bc_matrix/) needs per-cell quality assessment based on gene
  body coverage skewness before downstream scRNA-seq analysis.
when_to_use_negative:
- BAM files are already split by cell — the 0_split10XbyBarcode.sh step can be skipped,
  but the rest of the workflow still applies
- The scRNA-seq data is non-10X with per-cell files already available — SkewC remains
  technology-independent but the 10X-specific barcode splitting is unnecessary
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: 'Main pipeline: splits the barcoded BAM, computes gene body coverage per cell,
    and classifies typical vs skewed cells via trim clustering'
  repo: https://github.com/LSBDT/SkewC.git
- name: tclust
  role: R trim clustering function applied over gene body coverage profiles in SkewC_TrimClustering.Rmd
    to separate typical from skewed cells
  repo: https://github.com/LSBDT/SkewC.git
- name: geneBodyCoverage.pl
  role: Perl script imitating RSeQC geneBody_coverage.py to compute per-cell gene
    body coverage ~10x faster
  repo: https://github.com/LSBDT/SkewC.git
- name: docker
  role: Container runtime option for executing the SkewC workflow
- name: singularity
  role: Recommended container runtime; build SIF via singularity build skewc.sif docker://moirai2/skewc:latest
- name: udocker
  role: Docker-like container execution in user space without admin privileges, an
    alternative to docker/singularity for running SkewC
  repo: https://github.com/indigo-dc/udocker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:a85a8a78218e93786c195c2245c680cffc368beb8adc8018f599f051f3fbc7c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - docker
  - geneBodyCoverage.pl
  - singularity
  - tclust
  - udocker
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-rnaseq-quality-assessment

## Summary

Apply SkewC to evaluate single-cell RNA-seq data quality by computing per-cell gene body coverage, then using tclust trim clustering (k=1, user-set alpha) to separate typical cells with prototypical coverage profiles from skewed low-quality cells that should be excluded from downstream analysis.

## When to use

Use when a 10X Genomics barcoded BAM file (with barcodes.tsv.gz from outs/filtered_feature_bc_matrix/) needs per-cell quality assessment based on gene body coverage skewness before downstream scRNA-seq analysis.

## When NOT to use

- BAM files are already split by cell — the 0_split10XbyBarcode.sh step can be skipped, but the rest of the workflow still applies
- The scRNA-seq data is non-10X with per-cell files already available — SkewC remains technology-independent but the 10X-specific barcode splitting is unnecessary

## Inputs

- 10X Genomics barcoded BAM file (e.g., possorted_genome_bam.bam)
- barcodes.tsv.gz from 10X Genomics outs/filtered_feature_bc_matrix/
- Reference BED file (hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed) bundled in the SkewC reference/ directory

## Outputs

- Per-cell .geneBodyCoverage, .geneBodyCoverage.txt, and .log files under the coverage directory
- TypicalCellsID.tsv (cell barcodes with prototypical gene body coverage)
- SkewedCellsID.tsv (cell barcodes with skewed gene body coverage recommended for exclusion)
- Index HTML report under the skewc/ output directory, including gene body coverage plots for typical and skewed cells

## How to apply

Clone the SkewC repository (git clone https://github.com/LSBDT/SkewC.git) and run the batch scripts in order from the SkewC root directory. First, split the barcoded BAM into one BAM per cell with 0_split10XbyBarcode.sh, supplying the BAM, the barcodes.tsv.gz, and an output directory (default='input'). Second, compute per-cell gene body coverage with 1_geneBodyCoverage.sh (species 'hg38' or 'mm10'), which uses geneBodyCoverage.pl — a Perl reimplementation of RSeQC geneBody_coverage.py roughly 10x faster, computing ~9 min per 1GB BAM. Third, run 2_SkewC.sh with a project name, coverage input directory, output directory 'skewc', and an alpha value (e.g., alpha=0.04); this applies trim clustering via the R tclust function (SkewC_TrimClustering.Rmd) over the gene body coverage profiles. Validation: check that TypicalCellsID.tsv and SkewedCellsID.tsv plus the index HTML appear in the skewc/ directory and that typical + skewed counts equal the number of input cells (e.g., 897 typical + 33 skewed = 930 for the 1k Brain Cells from an E18 Mouse dataset). Optionally pre-filter cells with 3_filter.sh before running SkewC.

## Related tools

- **SkewC** (Main pipeline: splits the barcoded BAM, computes gene body coverage per cell, and classifies typical vs skewed cells via trim clustering) — https://github.com/LSBDT/SkewC.git
- **tclust** (R trim clustering function applied over gene body coverage profiles in SkewC_TrimClustering.Rmd to separate typical from skewed cells) — https://github.com/LSBDT/SkewC.git
- **geneBodyCoverage.pl** (Perl script imitating RSeQC geneBody_coverage.py to compute per-cell gene body coverage ~10x faster) — https://github.com/LSBDT/SkewC.git
- **docker** (Container runtime option for executing the SkewC workflow)
- **singularity** (Recommended container runtime; build SIF via singularity build skewc.sif docker://moirai2/skewc:latest)
- **udocker** (Docker-like container execution in user space without admin privileges, an alternative to docker/singularity for running SkewC) — https://github.com/indigo-dc/udocker

## Examples

```
bash 0_split10XbyBarcode.sh example/example.bam example/barcodes.tsv.gz; bash 1_geneBodyCoverage.sh mm10 input coverage; bash 2_SkewC.sh prjname coverage skewc 0.04
```

## Evaluation signals

- TypicalCellsID.tsv and SkewedCellsID.tsv plus an index HTML are written to the skewc/ output directory
- The number of typical cells plus skewed cells equals the number of input cells (e.g., 897 + 33 = 930 for the 1k Brain Cells from an E18 Mouse dataset)
- Per-cell .geneBodyCoverage, .geneBodyCoverage.txt, and .log files exist under the coverage directory for every cell barcode
- Gene body coverage plots in the HTML show prototypical profiles for typical cells and skewed profiles for skewed cells
- The trim clustering used the specified alpha (e.g., alpha=0.04) — an empty PDF plot signals a barcode mismatch between barcode.tsv and the BAM, not a successful run

## Limitations

- Gene body coverage computation is time-consuming: ~9 min per 1GB BAM, and a 500-cell BAM took on average 3 h on 10 cores; performance depends on BAM and reference genome size
- Barcode mismatch between barcode.tsv and the barcoded BAM produces empty gene body coverage PDF plots
- An R package error (stringi/libicui18n.so.57) terminates SkewC and is not fixable from the user's side; SkewC requires git and at least one of docker, udocker, or singularity installed
- geneBodyCoverage.pl output differs slightly from RSeQC geneBody_coverage.py, though the difference is negligible
- Do not run geneBodyCoverage.pl in parallel while it is creating the reference index (5-10 min indexing step; mm10 index 60MB, hg38 index 82MB)

## Evidence

- [abstract] SkewC distinguishes between two types of single cells: typical cells with prototypical gene body coverage profiles and skewed cells with skewed gene body coverage profiles.: "SkewC distinguishes between two types of single cells: typical cells with prototypical gene body coverage profiles and skewed cells with skewed gene body coverage profiles."
- [abstract] The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function.: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
- [methods] geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py': "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [readme] Computation time of geneBodyCoverage.pl is around 9min per 1GB.: "Computation time of geneBodyCoverage.pl is around 9min per 1GB."
- [discussion] running SkewC on a BAM file containing 500 cells took on average 3 h to compute the gene body coverage.: "running SkewC on a BAM file containing 500 cells took on average 3 h to compute the gene body coverage."
- [discussion] It occurs when there is a mis-match between the barcode names in the barcode.tsv file and the barcodes in the barcoded BAM file.: "It occurs when there is a mis-match between the barcode names in the barcode.tsv file and the barcodes in the barcoded BAM file."

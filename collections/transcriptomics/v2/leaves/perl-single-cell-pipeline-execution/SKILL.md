---
name: perl-single-cell-pipeline-execution
description: Use when a split barcoded 10X BAM set (one BAM per cell barcode) must
  be converted into per-cell gene body coverage profiles under SkewC, using either
  hg38 (Gencode V28) or mm10 (Gencode VM18) references.
when_to_use_negative:
- BAM files are not yet split by cell barcode — run 0_split10XbyBarcode.sh first (or
  skip only if BAMs are already per-cell)
- Using RSeQC geneBody_coverage.py instead, which requires indexed BAM files and is
  about 10x slower
- The 1_geneBodyCoverage.sh output is meant for gene body coverage computation, not
  quality filtering (that is 3_filter.sh) or trim clustering (2_SkewC.sh)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: Pipeline whose gene body coverage computation step is implemented by bin/geneBodyCoverage.pl
  repo: https://github.com/LSBDT/SkewC.git
- name: Perl
  role: Language of geneBodyCoverage.pl, which imitates RSeQC geneBody_coverage.py
    at roughly 10x speed
- name: R
  role: Reads the per-cell 100-value coverage vectors from coverage.r into an R data
    frame for downstream analysis
- name: RSeQC
  role: geneBody_coverage.py reference algorithm that geneBodyCoverage.pl imitates
- name: docker
  role: Container runtime option for executing the SkewC workflow pipeline
- name: singularity
  role: Recommended container option; SIF built via 'singularity build skewc.sif docker://moirai2/skewc:latest'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:696e7a7f8869e073d897976bb003af95920ba1809a8fe7bb55840954f3621d24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - Perl
  - R
  - RSeQC
  - SkewC
  - docker
  - singularity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# perl-single-cell-pipeline-execution

## Summary

Executes the SkewC per-cell gene body coverage computation step, in which bin/geneBodyCoverage.pl — a Perl reimplementation of RSeQC geneBody_coverage.py — computes a 100-value normalized coverage vector for each single-cell BAM file, aggregated into a coverage.r file for downstream SkewC analysis.

## When to use

Use when a split barcoded 10X BAM set (one BAM per cell barcode) must be converted into per-cell gene body coverage profiles under SkewC, using either hg38 (Gencode V28) or mm10 (Gencode VM18) references.

## When NOT to use

- BAM files are not yet split by cell barcode — run 0_split10XbyBarcode.sh first (or skip only if BAMs are already per-cell)
- Using RSeQC geneBody_coverage.py instead, which requires indexed BAM files and is about 10x slower
- The 1_geneBodyCoverage.sh output is meant for gene body coverage computation, not quality filtering (that is 3_filter.sh) or trim clustering (2_SkewC.sh)

## Inputs

- Per-cell split BAM files (one BAM per cell barcode, e.g. input/example.TTTGTCATCTAACGGT-1.bam)
- Species reference BED file: reference/hg38_Gencode_V28.norRNAtRNA.bed or reference/mm10_Gencode_VM18.norRNAtRNA.bed
- Species argument: 'hg38' (human) or 'mm10' (mouse)

## Outputs

- Per-cell files XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log written to the coverage directory
- A vector of 100 numerical normalized coverage values per cell (identified by cell id/barcode), aggregated into coverage.r for downstream SkewC R Markdown analysis
- Reference index file under the reference directory (mm10: 60MB; hg38: 82MB) built on first iteration

## How to apply

First ensure the barcoded BAM has been split into per-cell BAM files (e.g. by 0_split10XbyBarcode.sh into the default 'input' directory). Then run 'bash 1_geneBodyCoverage.sh $species $indir $outdir' with species='hg38' or 'mm10' (default hg38), indir='input', outdir='coverage', which invokes 'perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.bam > coverage/example.log'. On the first iteration the script builds a reference index under the reference directory (about 5-10 minutes; hg38 13MB bed -> 82MB index, mm10 9MB bed -> 60MB index) and auto-detects whether the 10X BAM reference column lacks the 'chr' prefix, so it must not be run in parallel during indexing; subsequent iterations reuse the index. The Perl script runs about 10x faster than RSeQC geneBody_coverage.py at roughly 9 minutes per 1GB of BAM input, and writes XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log per cell into the coverage directory.

## Related tools

- **SkewC** (Pipeline whose gene body coverage computation step is implemented by bin/geneBodyCoverage.pl) — https://github.com/LSBDT/SkewC.git
- **Perl** (Language of geneBodyCoverage.pl, which imitates RSeQC geneBody_coverage.py at roughly 10x speed)
- **R** (Reads the per-cell 100-value coverage vectors from coverage.r into an R data frame for downstream analysis)
- **RSeQC** (geneBody_coverage.py reference algorithm that geneBodyCoverage.pl imitates)
- **docker** (Container runtime option for executing the SkewC workflow pipeline)
- **singularity** (Recommended container option; SIF built via 'singularity build skewc.sif docker://moirai2/skewc:latest')

## Examples

```
perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam > coverage/example.TTTGTCATCTAACGGT-1.log
```

## Evaluation signals

- Each cell's coverage vector contains exactly 100 values
- All coverage values fall within the normalized range [0,1]
- Coverage directory contains XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log for every input per-cell BAM
- A reference index file of the expected size (hg38: 82MB from a 13MB bed; mm10: 60MB from a 9MB bed) was created under the reference directory on first iteration
- Runtime is consistent with ~9 minutes per 1GB of BAM input and output differs from RSeQC geneBody_coverage.py only negligibly

## Limitations

- Do not run geneBodyCoverage.pl in parallel while it is creating the index file (first iteration takes about 5-10 minutes)
- geneBodyCoverage.pl output differs a bit from RSeQC geneBody_coverage.py, although the algorithm is the same (differences negligible)
- A barcode mismatch between barcode.tsv and the barcoded BAM file produces empty gene body coverage output/plots
- An R package error (stringi/libicui18n.so.57) can terminate SkewC execution and is not fixable from the user's side
- Computation time scales with BAM file and reference genome size; a 500-cell BAM took on average 3 h (10 cores)

## Evidence

- [methods] geneBodyCoverage.pl imitates RSeQC geneBody_coverage.py: "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [readme] 10x speed and ~9 min per 1GB runtime: "geneBodyCoverage.pl was written to reduce running time of geneBodyCoverage step by about 10 folds."
- [readme] Sequential indexing requirement: "Don't run geneBodyCoverage.pl in parallel (at the same time) when it's creating an index file."
- [abstract] Critical pipeline step: "A critical step in SkewC is the gene body coverage computation."
- [other] 100-value normalized coverage vectors in coverage.r: [evidence span withheld — anchor status: not_found]

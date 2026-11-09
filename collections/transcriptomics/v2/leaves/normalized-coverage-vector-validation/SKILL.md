---
name: normalized-coverage-vector-validation
description: Use when per-cell gene body coverage files (XXXXXXXXX.geneBodyCoverage,
  .geneBodyCoverage.txt) have been generated for split 10X BAM files and must be validated
  and aggregated into coverage.r before running 2_SkewC.sh.
when_to_use_negative:
- Input is a single barcoded 10X BAM file that has not yet been split by cell barcode
  with 0_split10XbyBarcode.sh — no per-cell coverage vectors can be produced
- The goal is to run RSeQC's original geneBody_coverage.py, which requires indexed
  BAM files and is not what geneBodyCoverage.pl replaces (outputs differ a bit, though
  negligibly)
- Barcode names in barcode.tsv.gz mismatch the barcodes in the barcoded BAM, which
  produces empty gene body coverage output rather than valid 100-value vectors
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: Workflow whose gene body coverage computation (bin/geneBodyCoverage.pl, invoked
    via 1_geneBodyCoverage.sh) generates the per-cell normalized coverage vectors
    validated here
  repo: https://github.com/LSBDT/SkewC.git
- name: Perl
  role: Language in which geneBodyCoverage.pl is implemented to imitate RSeQC geneBody_coverage.py
    at ~10x speed
- name: R
  role: Language in which the aggregated 100-value coverage vectors are read from
    coverage.r into an R data frame for downstream SkewC analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:f2f5134c0438cfb77e2793e4ddfb1a909148f752761b8051fb286a43f8952e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - Perl
  - R
  - SkewC
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# normalized-coverage-vector-validation

## Summary

Validates the per-cell gene body coverage vectors produced by SkewC's geneBodyCoverage.pl step, confirming each cell (barcode) yields exactly 100 normalized coverage values aggregated into the coverage.r data frame for downstream trim clustering.

## When to use

Use when per-cell gene body coverage files (XXXXXXXXX.geneBodyCoverage, .geneBodyCoverage.txt) have been generated for split 10X BAM files and must be validated and aggregated into coverage.r before running 2_SkewC.sh.

## When NOT to use

- Input is a single barcoded 10X BAM file that has not yet been split by cell barcode with 0_split10XbyBarcode.sh — no per-cell coverage vectors can be produced
- The goal is to run RSeQC's original geneBody_coverage.py, which requires indexed BAM files and is not what geneBodyCoverage.pl replaces (outputs differ a bit, though negligibly)
- Barcode names in barcode.tsv.gz mismatch the barcodes in the barcoded BAM, which produces empty gene body coverage output rather than valid 100-value vectors

## Inputs

- split per-cell BAM files (one BAM per cell barcode) in the input directory (default='input')
- species-specific reference BED (reference/hg38_Gencode_V28.norRNAtRNA.bed or reference/mm10_Gencode_VM18.norRNAtRNA.bed)
- species argument ('hg38' or 'mm10')

## Outputs

- per-cell files XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log in the coverage directory (default='coverage')
- per-cell 100-value normalized coverage vector in [0,1]
- aggregated coverage.r data frame file for downstream SkewC analysis
- reference index file created under the reference directory on first iteration

## How to apply

After running 'bash 1_geneBodyCoverage.sh $species $indir $outdir' (species='hg38' or 'mm10'), inspect the coverage output directory for the per-cell files XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log. For each cell identified by its barcode, confirm the coverage vector contains exactly 100 values and that every value lies within the normalized range [0,1], since geneBodyCoverage.pl imitates RSeQC geneBody_coverage.py's normalized gene body coverage with about 10x speedup (~9 min per 1GB of BAM input). Aggregate the per-cell 100-value vectors into the coverage.r file, which is read into an R data frame by the SkewC R Markdowns (e.g., TestData/coverage.r). Note that geneBodyCoverage.pl builds its reference index on the first iteration (5-10 minutes; hg38 13MB bed -> 82MB index, mm10 9MB bed -> 60MB index) and must not be run in parallel during indexing.

## Related tools

- **SkewC** (Workflow whose gene body coverage computation (bin/geneBodyCoverage.pl, invoked via 1_geneBodyCoverage.sh) generates the per-cell normalized coverage vectors validated here) — https://github.com/LSBDT/SkewC.git
- **Perl** (Language in which geneBodyCoverage.pl is implemented to imitate RSeQC geneBody_coverage.py at ~10x speed)
- **R** (Language in which the aggregated 100-value coverage vectors are read from coverage.r into an R data frame for downstream SkewC analysis)

## Examples

```
bash 1_geneBodyCoverage.sh hg38 input coverage
```

## Evaluation signals

- Each cell's coverage vector contains exactly 100 values
- All coverage values fall within the normalized range [0,1]
- Each processed cell produces the expected triplet of files: XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log in the coverage directory
- The aggregated coverage.r file loads into an R data frame with one 100-value row per cell barcode
- Runtime is consistent with the documented ~9 min per 1GB of BAM input (E-MTAB-2600 test case mapped to mm10)

## Limitations

- geneBodyCoverage.pl must not be run in parallel while it is creating the reference index on the first iteration (indexing takes about 5-10 minutes)
- Output from geneBodyCoverage.pl differs a bit from RSeQC geneBody_coverage.py, although the same algorithm is used and the difference is negligible
- A mismatch between barcode names in barcode.tsv and barcodes in the barcoded BAM yields empty gene body coverage output (empty PDF plot)
- Performance depends on both the BAM file size and reference genome size in GB

## Evidence

- [methods] geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py': "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [readme] Per-cell output files and runtime: "Computation time of geneBodyCoverage.pl is around 9min per 1GB."
- [readme] Indexing caveat: "Don't run geneBodyCoverage.pl in parallel (at the same time) when it's creating an index file."
- [readme] Output equivalence with RSeQC: "Although same algorithm is used, output from geneBodyCoverage.pl and geneBody_coverage.py differs a bit, but it's negligible."
- [readme] Aggregated coverage.r used downstream: "coverage.r - sample data for SkewC Rmarkdown"

---
name: per-cell-bam-processing
description: Use when starting from per-cell (barcode-split) BAM files from a 10x
  or non-10x scRNA-seq experiment and you need a per-cell normalized gene body coverage
  vector for SkewC quality/skewness analysis.
when_to_use_negative:
- BAM files are not yet split per cell — run 0_split10XbyBarcode.sh first (10x Genomics
  outputs a single barcoded BAM that must be split by cell barcode)
- You plan to run RSeQC's geneBody_coverage.py instead — then BAM index files are
  required and the geneBodyCoverage.pl speed/behavior described here does not apply
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: SkewC
  role: pipeline whose 1_geneBodyCoverage.sh step drives per-cell gene body coverage
    computation
  repo: https://github.com/LSBDT/SkewC.git
- name: geneBodyCoverage.pl
  role: Perl script that imitates RSeQC geneBody_coverage.py to compute per-cell gene
    body coverage about 10x faster, without needing BAM index files
  repo: https://github.com/LSBDT/SkewC.git
- name: R
  role: coverage.r output is read into an R data frame for downstream SkewC analysis
- name: docker
  role: containerization option for running the SkewC workflow
- name: udocker
  role: user-space container runtime for running SkewC where docker privileges are
    unavailable
  repo: https://github.com/indigo-dc/udocker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:78d739c54dc26b3806fe284bd9962cf27dfcf876c2420ddb912670643e745362
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - docker
  - geneBodyCoverage.pl
  - udocker
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-cell-bam-processing

## Summary

Computes normalized gene body coverage for each single cell in the SkewC scRNA-seq workflow by running geneBodyCoverage.pl (a ~10x faster Perl imitation of RSeQC's geneBody_coverage.py) over per-cell BAM files, producing per-cell 100-value coverage vectors aggregated into coverage.r.

## When to use

Use when starting from per-cell (barcode-split) BAM files from a 10x or non-10x scRNA-seq experiment and you need a per-cell normalized gene body coverage vector for SkewC quality/skewness analysis.

## When NOT to use

- BAM files are not yet split per cell — run 0_split10XbyBarcode.sh first (10x Genomics outputs a single barcoded BAM that must be split by cell barcode)
- You plan to run RSeQC's geneBody_coverage.py instead — then BAM index files are required and the geneBodyCoverage.pl speed/behavior described here does not apply

## Inputs

- per-cell BAM files in the input directory (default='input') produced by the barcode-splitting step (0_split10XbyBarcode.sh)
- reference BED file: reference/hg38_Gencode_V28.norRNAtRNA.bed (human) or reference/mm10_Gencode_VM18.norRNAtRNA.bed (mouse)
- species argument: 'hg38' or 'mm10' (default='hg38')

## Outputs

- XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log per cell in the coverage directory (default='coverage')
- reference index file created under the reference directory (mm10: ~60MB; hg38: ~82MB)
- coverage.r file aggregating one vector of 100 numerical normalized coverage values (range [0,1]) per cell for downstream SkewC R Markdown analysis

## How to apply

Collect the per-cell BAM files in the input directory (default='input') produced by 0_split10XbyBarcode.sh, then run 'bash 1_geneBodyCoverage.sh $species $indir $outdir' with species='hg38' or 'mm10', indir='input', outdir='coverage'. The script invokes bin/geneBodyCoverage.pl against the matching norRNAtRNA BED (reference/hg38_Gencode_V28.norRNAtRNA.bed or reference/mm10_Gencode_VM18.norRNAtRNA.bed). On the first iteration geneBodyCoverage.pl builds a reference index (about 5-10 minutes; hg38 13MB bed -> 82MB index, mm10 9MB bed -> 60MB index) and auto-detects whether the 10X BAM reference column lacks the 'chr' prefix, so run cells sequentially, not in parallel, during indexing. The script imitates RSeQC geneBody_coverage.py at roughly 10x speed, about 9 min per 1GB of BAM input, writing XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log per cell into the coverage directory. Finally, aggregate each cell's 100-value normalized coverage vector into the coverage.r file, which is read into an R data frame for downstream SkewC trim-clustering analysis.

## Related tools

- **SkewC** (pipeline whose 1_geneBodyCoverage.sh step drives per-cell gene body coverage computation) — https://github.com/LSBDT/SkewC.git
- **geneBodyCoverage.pl** (Perl script that imitates RSeQC geneBody_coverage.py to compute per-cell gene body coverage about 10x faster, without needing BAM index files) — https://github.com/LSBDT/SkewC.git
- **R** (coverage.r output is read into an R data frame for downstream SkewC analysis)
- **docker** (containerization option for running the SkewC workflow)
- **udocker** (user-space container runtime for running SkewC where docker privileges are unavailable) — https://github.com/indigo-dc/udocker

## Examples

```
bash 1_geneBodyCoverage.sh $species $indir $outdir
```

## Evaluation signals

- Each cell's coverage vector contains exactly 100 values
- All coverage values fall within the normalized range [0,1]
- Per-cell output files XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log exist for every cell barcode in the coverage directory
- Reference index file was created on first iteration and reused (no re-indexing) on subsequent iterations
- Runtime is consistent with the documented ~9 min per 1GB of BAM input

## Limitations

- Do not run geneBodyCoverage.pl in parallel while it is creating the reference index (first iteration only, ~5-10 minutes)
- Runtime scales with the number of cells and depends on BAM file and reference genome size; 500 cells took on average 3 h (10 cores)
- Barcode mismatch between barcode.tsv and the barcoded BAM file produces empty gene body coverage plots (empty PDF)
- An R package error (stringi/libicui18n.so.57) terminates SkewC and is not fixable from the user's side
- Output differs slightly from RSeQC geneBody_coverage.py, though the difference is negligible

## Evidence

- [methods] geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py': "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [readme] indexing timing and sizes: "Indexing reference files take about 5-10 minutes."
- [readme] no parallel indexing: "Don't run geneBodyCoverage.pl in parallel (at the same time) when it's creating an index file."
- [full_text] per-cell 100-value output: [evidence span withheld — anchor status: not_found]
- [readme] runtime: "Computation time of geneBodyCoverage.pl is around 9min per 1GB."

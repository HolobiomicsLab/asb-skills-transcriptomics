---
name: gene-body-coverage-computation
description: Use when you have per-cell BAM files split from a 10x Genomics barcoded
  BAM and need normalized gene body coverage profiles (100 values per cell) as input
  to SkewC quality assessment.
when_to_use_negative:
- If you plan to use RSeQC's geneBody_coverage.py instead, per-cell BAM index files
  are required and SkewC's 0_split10XbyBarcode.sh splitting/indexing shortcuts do
  not apply
- If BAM files are already split by cells and indexed for another tool, note that
  geneBodyCoverage.pl itself does not need index files — do not add unnecessary indexing
  work
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SkewC
  role: Provides 1_geneBodyCoverage.sh and bin/geneBodyCoverage.pl that compute per-cell
    gene body coverage with reference indexing
  repo: https://github.com/LSBDT/SkewC.git
- name: RSeQC
  role: Reference implementation (geneBody_coverage.py) whose process geneBodyCoverage.pl
    imitates; used for equivalence/runtime comparison
- name: Perl
  role: Language in which geneBodyCoverage.pl is implemented to achieve ~10x speed-up
    over RSeQC
- name: R
  role: Reads the coverage.r file into an R data frame for downstream SkewC trim clustering
    analysis
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:20dd3d9de430c96d45a651fd1e6dcc893c8de1931e330136db4b4d9c8391aeba
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
  - udocker
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# gene-body-coverage-computation

## Summary

Computes per-cell gene body coverage for 10x Genomics scRNA-seq BAM files using SkewC's geneBodyCoverage.pl, a Perl re-implementation of RSeQC's geneBody_coverage.py that runs roughly 10x faster and emits a 100-bin normalized coverage vector per cell for downstream SkewC trim clustering.

## When to use

Use when you have per-cell BAM files split from a 10x Genomics barcoded BAM and need normalized gene body coverage profiles (100 values per cell) as input to SkewC quality assessment.

## When NOT to use

- If you plan to use RSeQC's geneBody_coverage.py instead, per-cell BAM index files are required and SkewC's 0_split10XbyBarcode.sh splitting/indexing shortcuts do not apply
- If BAM files are already split by cells and indexed for another tool, note that geneBodyCoverage.pl itself does not need index files — do not add unnecessary indexing work

## Inputs

- Per-cell BAM files produced by 0_split10XbyBarcode.sh (default directory 'input')
- Species selection: 'hg38' or 'mm10'
- Reference BED: reference/hg38_Gencode_V28.norRNAtRNA.bed or reference/mm10_Gencode_VM18.norRNAtRNA.bed

## Outputs

- Per-cell files XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, and XXXXXXXXX.log in the coverage directory (default='coverage')
- Aggregated coverage.r file containing a 100-value normalized coverage vector per cell barcode
- Reference index file created under the reference directory (hg38: ~82MB; mm10: ~60MB)

## How to apply

Run 'bash 1_geneBodyCoverage.sh $species $indir $outdir' with species 'hg38' (reference/hg38_Gencode_V28.norRNAtRNA.bed) or 'mm10' (reference/mm10_Gencode_VM18.norRNAtRNA.bed), indir defaulting to 'input' (the split per-cell BAMs) and outdir to 'coverage'. On the first iteration geneBodyCoverage.pl builds a reference index (5–10 minutes; 13MB hg38 bed -> 82MB index, 9MB mm10 bed -> 60MB index), so run instances sequentially during indexing; afterwards the index is reused to speed up calculation and parallel runs are supported. The script auto-detects whether the 10x BAM reference column lacks the 'chr' prefix and does not require BAM index files (unlike RSeQC geneBody_coverage.py). Computation imitates geneBody_coverage.py at roughly 10x speed and about 9 minutes per 1GB of BAM input, writing per-cell .geneBodyCoverage, .geneBodyCoverage.txt, and .log files into the coverage directory. The per-cell 100-value normalized coverage vectors are aggregated into a coverage.r R data frame for the downstream trim clustering (2_SkewC.sh, k=1, alpha=0.04 in the E18 mouse example).

## Related tools

- **SkewC** (Provides 1_geneBodyCoverage.sh and bin/geneBodyCoverage.pl that compute per-cell gene body coverage with reference indexing) — https://github.com/LSBDT/SkewC.git
- **RSeQC** (Reference implementation (geneBody_coverage.py) whose process geneBodyCoverage.pl imitates; used for equivalence/runtime comparison)
- **Perl** (Language in which geneBodyCoverage.pl is implemented to achieve ~10x speed-up over RSeQC)
- **R** (Reads the coverage.r file into an R data frame for downstream SkewC trim clustering analysis)

## Examples

```
perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam > coverage/example.TTTGTCATCTAACGGT-1.log
```

## Evaluation signals

- Each cell's coverage vector contains exactly 100 values
- All coverage values fall within the normalized range [0,1]
- A per-cell log file (XXXXXXXXX.log) is written alongside each .geneBodyCoverage and .geneBodyCoverage.txt output in the coverage directory
- The reference index file is created once (hg38 ~82MB, mm10 ~60MB) and reused on subsequent iterations, confirming the indexing step completed
- Coverage profiles from geneBodyCoverage.pl differ negligibly from RSeQC geneBody_coverage.py on the same BAM, with recorded runtime ~10x faster and ~9 min per 1GB

## Limitations

- Do not run geneBodyCoverage.pl in parallel while it is creating the reference index (first iteration, ~5–10 minutes)
- Computation time is ~9 min per 1GB of BAM input and scales linearly with cell number; a 500-cell pbmc_8k subset averaged ~3 h on 10 cores
- Performance depends on both the BAM file and the reference genome size in GB
- Although the same algorithm is used, output from geneBodyCoverage.pl and RSeQC geneBody_coverage.py differs slightly, though the difference is negligible
- Barcode mismatches between barcodes.tsv.gz and the barcoded BAM produce empty outputs downstream, so verify barcode consistency before computing coverage

## Evidence

- [methods] geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py': "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [readme] index reuse and no parallel indexing: "geneBodyCoverage.pl will create an index file under a reference directory (default='reference') at the beginning of first iteration."
- [readme] runtime rate: "Computation time of geneBodyCoverage.pl is around 9min per 1GB."
- [methods] 100-value normalized coverage vector: [evidence span withheld — anchor status: not_found]
- [discussion] linear runtime scaling: "Performance evaluation analysis of SkewC shows a linear association between the number of cells and SkewC runtime."

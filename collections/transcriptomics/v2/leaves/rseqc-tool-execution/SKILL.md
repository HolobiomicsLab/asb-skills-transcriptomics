---
name: rseqc-tool-execution
description: Use when computing gene body coverage across many split per-cell scRNA-seq
  BAM files with RSeQC-equivalent output at ~10-fold lower cost, or when benchmarking
  geneBodyCoverage.pl vs geneBody_coverage.py on the same BAM/BED.
when_to_use_negative:
- BAM files are not yet split by cell barcode — run 0_split10XbyBarcode.sh first,
  otherwise the coverage is not per-cell
- Species has no bundled norRNAtRNA reference (SkewC provides only hg38_Gencode_V28
  and mm10_Gencode_VM18 BED files)
- The task demands bit-identical results to RSeQC — the outputs 'differ a bit' (negligible,
  but not identical)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: geneBodyCoverage.pl (SkewC)
  role: Perl reimplementation of RSeQC geneBody_coverage.py that builds a reference
    index on the first iteration to achieve ~10-fold faster per-cell gene body coverage
  repo: https://github.com/LSBDT/SkewC.git
- name: RSeQC geneBody_coverage.py
  role: Reference implementation run on the identical BAM/BED for the runtime and
    output equivalence comparison
  repo: http://rseqc.sourceforge.net/#genebody-coverage-py
- name: R
  role: Parses the .geneBodyCoverage outputs of both tools and computes per-bin coverage
    differences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:4dd3d6d930d73b50fc3175ed69918100eed08b4b104bde768615c678bbf72e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - RSeQC
  - RSeQC geneBody_coverage.py
  - SkewC
  - geneBodyCoverage.pl (SkewC)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# rseqc-tool-execution

## Summary

Executes SkewC's geneBodyCoverage.pl — a Perl script written to imitate RSeQC's geneBody_coverage.py — on split per-cell scRNA-seq BAM files, using reference indexing to cut gene body coverage runtime by about 10 folds (~9 min per 1 GB BAM). It also covers benchmarking the Perl reimplementation against RSeQC for runtime and output equivalence.

## When to use

Use when computing gene body coverage across many split per-cell scRNA-seq BAM files with RSeQC-equivalent output at ~10-fold lower cost, or when benchmarking geneBodyCoverage.pl vs geneBody_coverage.py on the same BAM/BED.

## When NOT to use

- BAM files are not yet split by cell barcode — run 0_split10XbyBarcode.sh first, otherwise the coverage is not per-cell
- Species has no bundled norRNAtRNA reference (SkewC provides only hg38_Gencode_V28 and mm10_Gencode_VM18 BED files)
- The task demands bit-identical results to RSeQC — the outputs 'differ a bit' (negligible, but not identical)

## Inputs

- Split per-cell BAM files (e.g. input/example.TTTGTCATCTAACGGT-1.bam) from a barcoded scRNA-seq dataset
- Species-appropriate reference BED: reference/hg38_Gencode_V28.norRNAtRNA.bed (human) or reference/mm10_Gencode_VM18.norRNAtRNA.bed (mouse)
- For the RSeQC comparison arm: the identical BAM plus its .bai index (required by geneBody_coverage.py)

## Outputs

- Per-cell geneBodyCoverage output files (.geneBodyCoverage and .geneBodyCoverage.txt) from each tool
- Reference index file created under the reference/ directory (60MB for mm10, 82MB for hg38 BED)
- Per-run logs (e.g. coverage/example.TTTGTCATCTAACGGT-1.log)
- Benchmark records: wall-clock runtimes, computed fold speed-up (~10x), and negligible per-bin coverage diff

## How to apply

Run the coverage step on already-split per-cell BAMs either via the wrapper 'bash 1_geneBodyCoverage.sh $species $indir $outdir' (species 'hg38' or 'mm10') or directly, e.g. 'perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam' (use mm10_Gencode_VM18.norRNAtRNA.bed for mouse data such as E-MTAB-2600 mapped to mm10). The first iteration creates a reference index under the default 'reference' directory (5-10 minutes; 60MB index for the 9MB mm10 BED, 82MB for the 13MB hg38 BED) — do not launch parallel 1_geneBodyCoverage.sh runs while indexing; from the second iteration on the indexed reference speeds up calculation. Unlike geneBody_coverage.py, geneBodyCoverage.pl does not need indexed (.bai) BAM files, and computation time is around 9min per 1GB. For the equivalence benchmark (source task), time both tools on the identical BAM and BED with the Unix 'time' utility, compute the wall-clock fold speed-up, then parse the resulting .geneBodyCoverage and .geneBodyCoverage.txt files from both tools and diff per-bin values with R or Python.

## Related tools

- **geneBodyCoverage.pl (SkewC)** (Perl reimplementation of RSeQC geneBody_coverage.py that builds a reference index on the first iteration to achieve ~10-fold faster per-cell gene body coverage) — https://github.com/LSBDT/SkewC.git
- **RSeQC geneBody_coverage.py** (Reference implementation run on the identical BAM/BED for the runtime and output equivalence comparison) — http://rseqc.sourceforge.net/#genebody-coverage-py
- **R** (Parses the .geneBodyCoverage outputs of both tools and computes per-bin coverage differences)

## Examples

```
perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam > coverage/example.TTTGTCATCTAACGGT-1.log
```

## Evaluation signals

- Unix 'time' wall-clock of geneBodyCoverage.pl vs geneBody_coverage.py on the identical BAM shows approximately an order-of-magnitude (~10-fold) speed-up
- Runtime scales at around 9min per 1GB of BAM, as documented for the E-MTAB-2600 test case mapped to mm10
- First run creates the reference index (60MB for the 9MB mm10 BED, 82MB for the 13MB hg38 BED) in 5-10 minutes; subsequent runs complete without re-indexing
- geneBodyCoverage.pl succeeds on BAMs without .bai index files, while geneBody_coverage.py requires indexed BAM
- Per-bin coverage values parsed from the two tools' .geneBodyCoverage files show a negligible diff

## Limitations

- Do not run geneBodyCoverage.pl in parallel (at the same time) while it is creating the reference index
- Index creation costs 5-10 minutes and 60-82MB per reference BED on the first iteration
- Output from geneBodyCoverage.pl and geneBody_coverage.py differs a bit, though this is described as negligible
- Performance depends both on the BAM file and reference genome size; running SkewC on a 500-cell BAM took on average 3 h for gene body coverage

## Evidence

- [methods] geneBodyCoverage.pl imitates RSeQC geneBody_coverage.py: "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [readme] 10-fold runtime reduction: "geneBodyCoverage.pl was written to reduce running time of geneBodyCoverage step by about 10 folds."
- [readme] Reference index creation on first iteration: "geneBodyCoverage.pl will create an index file under a reference directory (default='reference') at the beginning of first iteration."
- [readme] No BAM index required: "RSeQC 'geneBody_coverage.py' required bam files to be indexed, but geneBodyCoverage.pl doesn't need index files."
- [readme] Output near-equivalence: "Although same algorithm is used, output from geneBodyCoverage.pl and geneBody_coverage.py differs a bit, but it's negligible."
- [discussion] Runtime scale example: "running SkewC on a BAM file containing 500 cells took on average 3 h to compute the gene body coverage."

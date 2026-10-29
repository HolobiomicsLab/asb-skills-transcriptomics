---
name: reference-transcript-annotation-handling
description: Use when setting up SkewC gene body coverage computation and you must
  select the species-appropriate norRNAtRNA BED reference, build its index, or reconcile
  chromosome naming between the reference and 10X BAM files.
when_to_use_negative:
- You plan to run RSeQC's geneBody_coverage.py instead of geneBodyCoverage.pl, in
  which case BAM index files (not the SkewC reference index) are required
- The organism is neither human hg38 nor mouse mm10 — no pre-packaged norRNAtRNA BED
  reference is provided
- Per-cell BAM files are not yet available (run 0_split10XbyBarcode.sh first, or skip
  if BAMs are already split by cell)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SkewC
  role: Workflow whose geneBodyCoverage.pl consumes the species-specific norRNAtRNA
    BED reference and builds its index
  repo: https://github.com/LSBDT/SkewC.git
- name: Perl
  role: Language of geneBodyCoverage.pl and splitByGeneLength.pl, which build the
    index and prepare the reference BED
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:74aa327e4e59aa48ee2ecb7eeacdd7e25d4a02280c893ec4d2b8b990e3abcc8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - Perl
  - R
  - SkewC
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# reference-transcript-annotation-handling

## Summary

Prepares and manages the SkewC reference transcript annotations (hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed) used by geneBodyCoverage.pl, including first-run index building and BAM 'chr' prefix auto-detection.

## When to use

Use when setting up SkewC gene body coverage computation and you must select the species-appropriate norRNAtRNA BED reference, build its index, or reconcile chromosome naming between the reference and 10X BAM files.

## When NOT to use

- You plan to run RSeQC's geneBody_coverage.py instead of geneBodyCoverage.pl, in which case BAM index files (not the SkewC reference index) are required
- The organism is neither human hg38 nor mouse mm10 — no pre-packaged norRNAtRNA BED reference is provided
- Per-cell BAM files are not yet available (run 0_split10XbyBarcode.sh first, or skip if BAMs are already split by cell)

## Inputs

- Species identifier: 'hg38' (human) or 'mm10' (mouse)
- Reference transcript BED file: reference/hg38_Gencode_V28.norRNAtRNA.bed or reference/mm10_Gencode_VM18.norRNAtRNA.bed
- Split per-cell BAM files from the input directory (default='input')

## Outputs

- Reference index file under the reference directory (e.g. 82MB for hg38, 60MB for mm10)
- Per-cell coverage outputs (XXXXXXXXX.geneBodyCoverage, XXXXXXXXX.geneBodyCoverage.txt, XXXXXXXXX.log) consistent with the chosen reference annotation

## How to apply

Select the reference BED matching the organism: reference/hg38_Gencode_V28.norRNAtRNA.bed for human ('hg38') or reference/mm10_Gencode_VM18.norRNAtRNA.bed for mouse ('mm10'), optionally split by gene length with bin/splitByGeneLength.pl. On the first run, geneBodyCoverage.pl builds a reference index under the reference directory (about 5-10 minutes; the 13MB hg38 BED yields an 82MB index, the 9MB mm10 BED a 60MB index), which is reused from the second iteration onward to speed up coverage calculation. The script auto-detects whether the 10X BAM reference column lacks the 'chr' prefix and aligns reference annotations accordingly. During indexing, run geneBodyCoverage.pl sequentially — never in parallel — to avoid corrupt or conflicting index files.

## Related tools

- **SkewC** (Workflow whose geneBodyCoverage.pl consumes the species-specific norRNAtRNA BED reference and builds its index) — https://github.com/LSBDT/SkewC.git
- **Perl** (Language of geneBodyCoverage.pl and splitByGeneLength.pl, which build the index and prepare the reference BED)

## Examples

```
perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam > coverage/example.TTTGTCATCTAACGGT-1.log
```

## Evaluation signals

- The correct species BED (hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed) was selected to match the BAM's genome build
- An index file of the expected size (about 82MB for hg38, 60MB for mm10) appears under the reference directory after the first run
- Chromosome naming is consistent: the 'chr'-prefix status of the 10X BAM reference column was detected and handled
- Each cell's coverage vector derived from the reference contains exactly 100 values in the normalized range [0,1]

## Limitations

- Indexing takes about 5-10 minutes and geneBodyCoverage.pl must not be run in parallel during index creation
- Only hg38 (Gencode V28) and mm10 (Gencode VM18) norRNAtRNA references are shipped; other genomes require custom annotation
- Performance depends on both the BAM file size and the reference genome size in GB (about 9 min per 1GB of BAM input)

## Evidence

- [methods] Reference index building and chr-prefix auto-detection: [evidence span withheld — anchor status: not_found]
- [methods] Reference BED files shipped with SkewC: "reference/hg38_Gencode_V28.norRNAtRNA.bed"
- [readme] Index creation behavior from the README: "geneBodyCoverage.pl will create an index file under a reference directory (default='reference') at the beginning of first iteration. From second iteration on, indexed reference file will be used to"
- [methods] Splitting reference by gene length: "perl bin/splitByGeneLength.pl hg38_Gencode_V28.norRNAtRNA.bed"

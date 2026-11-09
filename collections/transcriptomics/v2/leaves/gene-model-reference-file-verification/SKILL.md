---
name: gene-model-reference-file-verification
description: Reference for reproducing or validating the SkewC gene-model filtering
  step when the default hg38/mm10 .norRNAtRNA.bed references cannot be used, e.g.
  a new assembly or GENCODE release.
when_to_use_negative:
- When the default SkewC references (hg38_Gencode_V28 or mm10_Gencode_VM18) already
  match the genome and annotation used for alignment — use them directly
- When running RSeQC geneBody_coverage.py instead of geneBodyCoverage.pl, since the
  custom .norRNAtRNA.bed index-building step is specific to the SkewC Perl script
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: intersectBed
  role: Removes rRNA and tRNA entries from the gene model BED with -split -v -s -wa
    filtering
- name: SkewC
  role: Consumes the verified .norRNAtRNA.bed reference during gene body coverage
    computation
  repo: https://github.com/LSBDT/SkewC.git
- name: UCSC Table Browser
  role: Source of the GENCODE basic gene BED and rRNA/tRNA repeat BED inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:24437963adc3ae4070b733bb89902e3704586bca50a42665a69000a9649820d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - SkewC
  - UCSC Table Browser
  - intersectBed
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-model-reference-file-verification

## Summary

Verify that a user-prepared gene model BED file (with rRNA and tRNA entries removed via intersectBed) matches the validated reference files shipped with SkewC (e.g. reference/hg38_Gencode_V28.norRNAtRNA.bed) before running gene body coverage.

## When to use

Reference for reproducing or validating the SkewC gene-model filtering step when the default hg38/mm10 .norRNAtRNA.bed references cannot be used, e.g. a new assembly or GENCODE release.

## When NOT to use

- When the default SkewC references (hg38_Gencode_V28 or mm10_Gencode_VM18) already match the genome and annotation used for alignment — use them directly
- When running RSeQC geneBody_coverage.py instead of geneBodyCoverage.pl, since the custom .norRNAtRNA.bed index-building step is specific to the SkewC Perl script

## Inputs

- GENCODE basic gene annotation BED (e.g. hg38_Gencode_V34.bed from UCSC Table Browser)
- rRNA/tRNA repeat BED (UCSC 'Variation and Repeats' group, repClass rRNA and tRNA)
- Reference .norRNAtRNA.bed from SkewC repo (reference/hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed)

## Outputs

- Filtered gene model BED (e.g. hg38_Gencode_V34.norRNAtRNA.bed) matching the SkewC reference file
- Validation comparison (diff/content match) against reference/hg38_Gencode_V28.norRNAtRNA.bed

## How to apply

Retrieve the GENCODE basic gene BED from the UCSC Table Browser (e.g. Human hg38, track ALL GENCODE, table wgEncodeGencodeBasicV34, BED output) and the rRNA/tRNA repeat BED from the 'Variation and Repeats' group, filtering on repClass for 'rRNA' and 'tRNA' separately. Remove rRNA/tRNA genes from the gene model with 'intersectBed -split -v -s -wa -a <genes.bed> -b <rRNA_tRNA.bed>'. Then confirm the output .norRNAtRNA.bed exists and its contents match the versioned reference shipped in the SkewC repository's reference/ directory (hg38_Gencode_V28 or mm10_Gencode_VM18), since geneBodyCoverage.pl indexes this exact file to accelerate coverage computation. The -s flag enforces strand specificity so only sense-strand overlaps are removed.

## Related tools

- **intersectBed** (Removes rRNA and tRNA entries from the gene model BED with -split -v -s -wa filtering)
- **SkewC** (Consumes the verified .norRNAtRNA.bed reference during gene body coverage computation) — https://github.com/LSBDT/SkewC.git
- **UCSC Table Browser** (Source of the GENCODE basic gene BED and rRNA/tRNA repeat BED inputs)

## Examples

```
intersectBed -split -v -s -wa -a hg38_Gencode_V34.bed -b hg38_rRNA_tRNA.bed > hg38_Gencode_V34.norRNAtRNA.bed
```

## Evaluation signals

- Output .norRNAtRNA.bed exists and is non-empty after the intersectBed command completes
- diff or content comparison against reference/hg38_Gencode_V28.norRNAtRNA.bed (or the equivalent versioned file) shows a match
- No BED entries overlapping rRNA/tRNA repeats on the sense strand remain (due to -split -v -s -wa flags)
- geneBodyCoverage.pl successfully builds its index over the file within the expected 5–10 minutes and file-size range (hg38 ~13MB BED → ~82MB index)
- Downstream gene body coverage step (bash 1_geneBodyCoverage.sh) runs without reference-file errors

## Limitations

- The article's example uses GENCODE V34 BED but the validated reference shipped is hg38_Gencode_V28; a new annotation version will not match the shipped reference byte-for-byte and needs its own validation
- geneBodyCoverage.pl index files must not be created in parallel; indexing the reference takes 5–10 minutes
- Computation depends on reference genome size (~9 min per 1GB BAM), so an incorrect or oversized gene model directly inflates runtime

## Evidence

- [methods] intersectBed filtering command: "intersectBed -split -v -s -wa -a hg38_Gencode_V34.bed -b hg38_rRNA_tRNA.bed > hg38_Gencode_V34.norRNAtRNA.bed"
- [methods] rRNA/tRNA removal documented: [evidence span withheld — anchor status: not_found]
- [readme] reference file used by SkewC pipeline: "perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam"
- [readme] index construction over reference BED: "geneBodyCoverage.pl will create an index file under a reference directory (default='reference') at the beginning of first iteration."
- [methods] UCSC retrieval of rRNA/tRNA repeats: [evidence span withheld — anchor status: not_found]

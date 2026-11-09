---
name: bed-genomic-interval-intersection
description: Use when preparing a custom gene model BED for SkewC gene body coverage
  and rRNA/tRNA sequences must be removed from a reference gene annotation.
when_to_use_negative:
- The target organism's reference gene model already shipped with SkewC (reference/hg38_Gencode_V28.norRNAtRNA.bed
  or mm10_Gencode_VM18.norRNAtRNA.bed) matches the assembly and annotation version
  being used
- The goal is general interval intersection or annotation enrichment rather than producing
  a rRNA/tRNA-depleted gene model for SkewC gene body coverage
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3307
tools:
- name: intersectBed
  role: Performs the inverse (-v), strand-aware (-s), split-mode intersection that
    removes rRNA/tRNA-overlapping genes from the GENCODE BED
- name: SkewC
  role: Consumer of the filtered .norRNAtRNA.bed reference; its reference/ directory
    ships the expected output files
  repo: https://github.com/LSBDT/SkewC.git
- name: UCSC Table Browser
  role: Source for both the GENCODE basic gene BED export and the repClass-filtered
    rRNA/tRNA repeat BEDs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:78542499106015cf15ea1d766bcb189484d419ccaa1bf56449d975173adf4605
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - SkewC
  - UCSC Table Browser
  - intersectBed
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# bed-genomic-interval-intersection

## Summary

Reconstructs a SkewC-compatible gene model BED by removing rRNA and tRNA intervals from a GENCODE reference annotation using an inverse, strand-aware, split-mode intersectBed operation, producing a filtered .norRNAtRNA.bed reference file.

## When to use

Use when preparing a custom gene model BED for SkewC gene body coverage and rRNA/tRNA sequences must be removed from a reference gene annotation.

## When NOT to use

- The target organism's reference gene model already shipped with SkewC (reference/hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed) matches the assembly and annotation version being used
- The goal is general interval intersection or annotation enrichment rather than producing a rRNA/tRNA-depleted gene model for SkewC gene body coverage

## Inputs

- GENCODE basic gene annotation in BED format (e.g. hg38_Gencode_V34.bed from UCSC Table Browser, track ALL GENCODE, table wgEncodeGencodeBasicV34)
- rRNA repeat intervals in BED format (UCSC 'Variation and Repeats' group, repClass filtered to 'rRNA')
- tRNA repeat intervals in BED format (UCSC 'Variation and Repeats' group, repClass filtered to 'tRNA')
- intersectBed (BEDTools)

## Outputs

- Filtered gene model BED file, e.g. hg38_Gencode_V34.norRNAtRNA.bed (or hg38_Gencode_V28.norRNAtRNA.bed), with rRNA/tRNA-overlapping genes removed

## How to apply

Retrieve the GENCODE basic gene BED (e.g. hg38 with wgEncodeGencodeBasicV34) from the UCSC Table Browser, then retrieve the rRNA and tRNA repeat BEDs from the 'Variation and Repeats' group by filtering repClass on 'rRNA' and 'tRNA' separately. Merge the repeat BEDs and run an inverse intersection 'intersectBed -split -v -s -wa -a <genes.bed> -b <rRNA_tRNA.bed>' so that only genes lacking rRNA/tRNA overlap are kept; -split handles block-level overlap on spliced BED12 entries and -s enforces strand awareness. Write the result as e.g. hg38_Gencode_V34.norRNAtRNA.bed, mirroring the reference/hg38_Gencode_V28.norRNAtRNA.bed and mm10_Gencode_VM18.norRNAtRNA.bed files shipped in the SkewC repository. The rationale is that SkewC's gene body coverage computation (geneBodyCoverage.pl, which imitates RSeQC's geneBody_coverage.py) indexes the gene model reference, so rRNA/tRNA entries would skew coverage profiles. Validate by confirming the output .norRNAtRNA.bed exists and matches the reference file shipped in the SkewC repo, then optionally run 'perl bin/splitByGeneLength.pl hg38_Gencode_V28.norRNAtRNA.bed' to exercise it in the SkewC pipeline.

## Related tools

- **intersectBed** (Performs the inverse (-v), strand-aware (-s), split-mode intersection that removes rRNA/tRNA-overlapping genes from the GENCODE BED)
- **SkewC** (Consumer of the filtered .norRNAtRNA.bed reference; its reference/ directory ships the expected output files) — https://github.com/LSBDT/SkewC.git
- **UCSC Table Browser** (Source for both the GENCODE basic gene BED export and the repClass-filtered rRNA/tRNA repeat BEDs)

## Examples

```
intersectBed -split -v -s -wa -a hg38_Gencode_V34.bed -b hg38_rRNA_tRNA.bed > hg38_Gencode_V34.norRNAtRNA.bed
```

## Evaluation signals

- Output file hg38_Gencode_V34.norRNAtRNA.bed (or matching version) exists and is non-empty
- Contents of the filtered .norRNAtRNA.bed match the shipped reference file reference/hg38_Gencode_V28.norRNAtRNA.bed (or the corresponding mm10 file) when built from the same inputs
- No retained gene interval overlaps an rRNA/tRNA repeat interval (inverse-intersection invariant of the -v flag)
- Downstream indexing works: geneBodyCoverage.pl creates its index from the filtered BED in about 5-10 minutes, and splitByGeneLength.pl accepts the file
- Index file size falls in the expected range for the filtered reference (e.g. hg38 ~13 MB BED => ~82 MB index; mm10 ~9 MB BED => ~60 MB index)

## Limitations

- Command variants in the article mix annotation versions (hg38_Gencode_V34 inputs vs. hg38_Gencode_V28.norRNAtRNA.bed reference), so the user must keep assembly/annotation versions consistent
- geneBodyCoverage.pl builds its index at the first iteration and must not be run in parallel during indexing; reference indexing takes about 5-10 minutes
- The article's own example output name (hg38_Gencode_V28.norRNAtRNA.bed in the workflow) differs from the input annotation (V34), requiring careful naming by the user

## Evidence

- [methods] intersectBed inverse intersection: [evidence span withheld — anchor status: not_found]
- [readme] shipped reference BED files: "reference/
│   ├── hg38_Gencode_V28.norRNAtRNA.bed
│   └── mm10_Gencode_VM18.norRNAtRNA.bed"
- [readme] why the filtered reference is used: "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [methods] UCSC retrieval steps: [evidence span withheld — anchor status: not_found]

---
name: ucsc-table-browser-gene-model-retrieval
description: Use when SkewC's shipped reference files (hg38_Gencode_V28.norRNAtRNA.bed,
  mm10_Gencode_VM18.norRNAtRNA.bed) do not match your genome assembly and a custom
  gene model without rRNA/tRNA entries must be built.
when_to_use_negative:
- A matching reference file (hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed)
  is already shipped in the SkewC repository and matches your assembly
- The goal is gene body coverage computation or cell filtering within SkewC itself
  rather than building the gene-model reference
- A different gene-model filtering strategy than strand-aware rRNA/tRNA exclusion
  is required
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: UCSC Table Browser
  role: Source of the hg38 GENCODE basic gene BED and rRNA/tRNA repeat BEDs (Variation
    and Repeats track filtered on repClass)
- name: intersectBed
  role: Removes rRNA/tRNA intervals from the gene model via 'intersectBed -split -v
    -s -wa -a <genes.bed> -b <rRNA_tRNA.bed>'
- name: SkewC
  role: Consumer of the resulting norRNAtRNA BED; its geneBodyCoverage.pl step uses
    the reference file for gene body coverage computation
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:f51c7dd617c23ddf4ad604112b4d1f5a6a16ecfb3e162973401a2fa2d784bbb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - SkewC
  - UCSC Table Browser
  - intersectBed
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# ucsc-table-browser-gene-model-retrieval

## Summary

Retrieves a GENCODE basic gene model BED (e.g. hg38 wgEncodeGencodeBasicV34) and rRNA/tRNA repeat BEDs from the UCSC Table Browser, then removes rRNA/tRNA intervals with intersectBed to build the norRNAtRNA gene-model reference required by SkewC.

## When to use

Use when SkewC's shipped reference files (hg38_Gencode_V28.norRNAtRNA.bed, mm10_Gencode_VM18.norRNAtRNA.bed) do not match your genome assembly and a custom gene model without rRNA/tRNA entries must be built.

## When NOT to use

- A matching reference file (hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed) is already shipped in the SkewC repository and matches your assembly
- The goal is gene body coverage computation or cell filtering within SkewC itself rather than building the gene-model reference
- A different gene-model filtering strategy than strand-aware rRNA/tRNA exclusion is required

## Inputs

- UCSC Table Browser query parameters for GENCODE basic gene track (wgEncodeGencodeBasicV34, BED output)
- UCSC Variation and Repeats track filtered on repClass='rRNA' and repClass='tRNA' (BED output)
- Reference gene annotation BED file (e.g. hg38_Gencode_V34.bed)
- BEDTools intersectBed (bedtools2) installation

## Outputs

- hg38_Gencode_V34.norRNAtRNA.bed — gene model BED with rRNA/tRNA sequences removed
- Validated norRNAtRNA reference file usable by SkewC's geneBodyCoverage.pl and splitByGeneLength.pl

## How to apply

First download the gene model from the UCSC Table Browser using genome: Human, assembly Dec. 2013 GRCh38/hg38, group: Genes and Gene Predictions, track: ALL GENCODE, table: Basic/wgEncodeGencodeBasicV34, output format BED. Then retrieve the rRNA and tRNA repeat BEDs from the same Table Browser by selecting the 'Variation and Repeats' group and filtering repClass on 'rRNA' and 'tRNA' separately. Finally run intersectBed with -split -v -s -wa -a <genes.bed> -b <rRNA_tRNA.bed> to emit the gene model with rRNA/tRNA entries removed; the -v flag keeps genes not overlapping repeats, -s enforces strand specificity, and -wa preserves the original gene lines. Validate the resulting .norRNAtRNA.bed against the reference file shipped in the SkewC repository (reference/hg38_Gencode_V28.norRNAtRNA.bed) before using it with geneBodyCoverage.pl.

## Related tools

- **UCSC Table Browser** (Source of the hg38 GENCODE basic gene BED and rRNA/tRNA repeat BEDs (Variation and Repeats track filtered on repClass))
- **intersectBed** (Removes rRNA/tRNA intervals from the gene model via 'intersectBed -split -v -s -wa -a <genes.bed> -b <rRNA_tRNA.bed>')
- **SkewC** (Consumer of the resulting norRNAtRNA BED; its geneBodyCoverage.pl step uses the reference file for gene body coverage computation) — https://github.com/LSBDT/SkewC.git

## Examples

```
intersectBed -split -v -s -wa -a hg38_Gencode_V34.bed -b hg38_rRNA_tRNA.bed > hg38_Gencode_V34.norRNAtRNA.bed
```

## Evaluation signals

- The output file hg38_Gencode_V34.norRNAtRNA.bed exists and is non-empty after intersectBed
- The output content matches the repository-shipped reference file reference/hg38_Gencode_V28.norRNAtRNA.bed for the same assembly
- No gene intervals in the output overlap rRNA/tRNA repeat regions on the same strand (spot-check with intersectBed without -v)
- The output BED format is accepted downstream by 'perl bin/splitByGeneLength.pl hg38_Gencode_V28.norRNAtRNA.bed' and geneBodyCoverage.pl
- Reference index file size is consistent with expectations (hg38 13MB bed file => 82MB index file)

## Limitations

- The Table Browser workflow is assembly-specific: parameters and output filenames shown for hg38/GENCODE V34 must be adapted for other assemblies (e.g. mm10/GENCODE VM18)
- Filenames in the article workflow (V34 retrieval written to hg38_Gencode_V28.norRNAtRNA.bed) reflect the SkewC-shipped naming; the validation diff is only exact when reproducing the V28 reference
- geneBodyCoverage.pl creates an index under the reference directory on first run and must not be run in parallel during indexing (5-10 minutes indexing time)

## Evidence

- [methods] intersectBed command: "intersectBed -split -v -s -wa -a hg38_Gencode_V34.bed -b hg38_rRNA_tRNA.bed > hg38_Gencode_V34.norRNAtRNA.bed"
- [methods] UCSC Table Browser parameters: [evidence span withheld — anchor status: not_found]
- [methods] repeat track retrieval: [evidence span withheld — anchor status: not_found]
- [readme] downstream consumer of the reference: "perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam > coverage/example.TTTGTCATCTAACGGT-1.log"

---
name: repeatmasker-repeat-class-filtering
description: Use when a user needs a custom gene-model BED for SkewC and must remove
  rRNA/tRNA RepeatMasker entries from a GENCODE annotation for a genome/assembly (e.g.
  hg38, mm10) not shipped in the repository's reference/ folder.
when_to_use_negative:
- The SkewC repository already ships a matching reference (reference/hg38_Gencode_V28.norRNAtRNA.bed
  or mm10_Gencode_VM18.norRNAtRNA.bed) for your genome and annotation version
- You need to filter whole-transcript or unblocked interval BEDs — the -split flag
  assumes blocked (BED12-style) gene models
- You want to remove repeat classes other than rRNA/tRNA from the gene model
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: intersectBed
  role: Performs the negative, split, strand-aware interval subtraction that removes
    rRNA/tRNA overlapping genes from the reference BED
- name: UCSC Table Browser
  role: Source of both the GENCODE basic gene BED and the rRNA/tRNA repeat BEDs (repClass
    filtering)
- name: SkewC
  role: 'Downstream consumer: the cleaned .norRNAtRNA.bed is used as the gene-model
    reference for gene body coverage via bin/geneBodyCoverage.pl'
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:805b6c281e75c1aa3e2c66a4efc29a611c00a03041481144091476fb5b609ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - SkewC
  - UCSC Table Browser
  - intersectBed
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# repeatmasker-repeat-class-filtering

## Summary

Prepares a SkewC gene-model BED by excluding rRNA and tRNA repeat entries from a GENCODE reference annotation using a negative, strand-aware, split-mode intersectBed operation, producing a cleaned '.norRNAtRNA.bed' reference for gene body coverage computation.

## When to use

Use when a user needs a custom gene-model BED for SkewC and must remove rRNA/tRNA RepeatMasker entries from a GENCODE annotation for a genome/assembly (e.g. hg38, mm10) not shipped in the repository's reference/ folder.

## When NOT to use

- The SkewC repository already ships a matching reference (reference/hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed) for your genome and annotation version
- You need to filter whole-transcript or unblocked interval BEDs — the -split flag assumes blocked (BED12-style) gene models
- You want to remove repeat classes other than rRNA/tRNA from the gene model

## Inputs

- GENCODE basic gene annotation in BED format from the UCSC Table Browser (e.g. wgEncodeGencodeBasicV34 for hg38)
- RepeatMasker repeat BED entries with repClass 'rRNA' and 'tRNA' exported separately from the UCSC Table Browser 'Variation and Repeats' group

## Outputs

- Cleaned gene-model BED without rRNA/tRNA entries, e.g. hg38_Gencode_V34.norRNAtRNA.bed (analogous to reference/hg38_Gencode_V28.norRNAtRNA.bed and mm10_Gencode_VM18.norRNAtRNA.bed)

## How to apply

First export the gene annotation from the UCSC Table Browser (e.g. genome Human, assembly Dec. 2013 GRCh38/hg38, group 'Genes and Gene Predictions', track 'ALL GENCODE', table 'Basic'/wgEncodeGencodeBasicV34, output format BED). Then export the RepeatMasker-derived repeats from the 'Variation and Repeats' group, filtering repClass on 'rRNA' and 'tRNA' separately, and concatenate them into one rRNA/tRNA BED. Run the negative intersection 'intersectBed -split -v -s -wa -a <genes.bed> -b <rRNA_tRNA.bed>' to keep genes (-v) with no overlap (-wa), using -split so only the blocked (exonic) portions of genes are tested and -s so overlaps must be on the same strand. Write the result as e.g. hg38_Gencode_V34.norRNAtRNA.bed, mirroring the reference/hg38_Gencode_V28.norRNAtRNA.bed file that ships with the SkewC repository.

## Related tools

- **intersectBed** (Performs the negative, split, strand-aware interval subtraction that removes rRNA/tRNA overlapping genes from the reference BED)
- **UCSC Table Browser** (Source of both the GENCODE basic gene BED and the rRNA/tRNA repeat BEDs (repClass filtering))
- **SkewC** (Downstream consumer: the cleaned .norRNAtRNA.bed is used as the gene-model reference for gene body coverage via bin/geneBodyCoverage.pl) — https://github.com/LSBDT/SkewC.git

## Examples

```
intersectBed -split -v -s -wa -a hg38_Gencode_V34.bed -b hg38_rRNA_tRNA.bed > hg38_Gencode_V34.norRNAtRNA.bed
```

## Evaluation signals

- Output file hg38_Gencode_V34.norRNAtRNA.bed (or equivalent) exists and is valid BED
- Contents of the output match the structure of the shipped reference file reference/hg38_Gencode_V28.norRNAtRNA.bed (same column layout and blocked gene format)
- No genes remain whose exonic blocks overlap rRNA/tRNA repeats on the same strand (re-running intersectBed -split -s -a output -b rRNA_tRNA.bed returns no records)
- Output line count is smaller than the input genes.bed line count, reflecting removed rRNA/tRNA-overlapping entries
- SkewC's geneBodyCoverage.pl can index the cleaned reference without errors (indexing takes about 5-10 minutes; hg38 13MB bed => 82MB index)

## Limitations

- The exact command shown in SkewC documentation references hg38 Gencode V34/V28 file names; for other assemblies (e.g. mm10) the file names must be adapted (mm10_Gencode_VM18.norRNAtRNA.bed)
- rRNA and tRNA repeats must be exported and filtered by repClass separately in the UCSC Table Browser before combining them into one -b file
- The same-strand constraint (-s) means antisense-overlapping rRNA/tRNA hits are retained in the gene model

## Evidence

- [methods] SkewC documents that rRNA and tRNA sequences are removed from the gene model reference using the intersectBed command: [evidence span withheld — anchor status: not_found]
- [methods] UCSC Table Browser retrieval of rRNA/tRNA repeats: [evidence span withheld — anchor status: not_found]
- [readme] Reference BED files shipped in SkewC repository: "│   ├── reference/
│   │   ├── hg38_Gencode_V28.norRNAtRNA.bed
│   │   └── mm10_Gencode_VM18.norRNAtRNA.bed"
- [readme] Cleaned reference used as gene-model input for coverage computation: "perl bin/geneBodyCoverage.pl -o coverage reference/hg38_Gencode_V28.norRNAtRNA.bed input/example.TTTGTCATCTAACGGT-1.bam > coverage/example.TTTGTCATCTAACGGT-1.log"

---
name: cleavage-site-peak-calling-per-cell-type
description: Use when the goal is to map 3' end cleavage sites de novo from MWS reads
  spanning the poly(A) tail per cell type, e.g. building a CS atlas from the Mouse
  Cell Atlas or Human Cell Landscape.
when_to_use_negative:
- 'Input is 10x Genomics scRNA-seq data: most reads do not span mRNA 3'' end cleavage
  sites, preventing exact de novo CS mapping — use scUTRquant quantification against
  a fixed CS set instead'
- The CS annotation set already exists and the task is quantification of gene and
  3'UTR isoform expression rather than de novo peak discovery
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PEAR
  role: Merges overlapping paired-end Microwell-seq reads to reconstruct full insert
    sequences carrying 3'-end information
- name: HISAT
  role: Aligns the poly(A)-trimmed reads to hg38 and mm10 reference genomes prior
    to 5' end aggregation
- name: Snakemake
  role: Workflow engine hosting the de novo CS calling pipelines (Mayrlab/mca-utrome,
    Mayrlab/hcl-utrome)
  repo: https://github.com/Mayrlab/mca-utrome
- name: scUTRquant
  role: Downstream pipeline that consumes the resulting CS/UTRome annotations for
    3'UTR isoform quantification
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:b4ea68e94f753dfe7f28241c9e9a0cf7b1c79e2f82c32db149bd079752bc4d24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/mca-utrome
  tools:
  - HISAT
  - PEAR
  - Snakemake
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cleavage-site-peak-calling-per-cell-type

## Summary

De novo mapping of mRNA 3' end cleavage sites (CS) from Microwell-seq (MWS) data across 206 human and mouse primary cell types, producing a coordinate-annotated CS atlas that extends GENCODE annotations by 40%.

## When to use

Use when the goal is to map 3' end cleavage sites de novo from MWS reads spanning the poly(A) tail per cell type, e.g. building a CS atlas from the Mouse Cell Atlas or Human Cell Landscape.

## When NOT to use

- Input is 10x Genomics scRNA-seq data: most reads do not span mRNA 3' end cleavage sites, preventing exact de novo CS mapping — use scUTRquant quantification against a fixed CS set instead
- The CS annotation set already exists and the task is quantification of gene and 3'UTR isoform expression rather than de novo peak discovery

## Inputs

- Microwell-seq paired-end FASTQ files from the Mouse Cell Atlas (GEO:GSE108097)
- Microwell-seq paired-end FASTQ files from the Human Cell Landscape (GEO:GSE134355)
- MCA/HCL cell type annotations
- hg38 and mm10 reference genomes
- GENCODE v39 (human) and vM21 (mouse) annotations

## Outputs

- Coordinate-annotated GTF records of cleavage site candidates per cell type (mouse N=170,617; human N=150,191)
- UTRome CS annotations consumable by the scUTRquant pipeline (Mayrlab/mca-utrome, Mayrlab/hcl-utrome outputs)

## How to apply

Download the Microwell-seq FASTQ files (Mouse Cell Atlas GEO:GSE108097; Human Cell Landscape GEO:GSE134355) and merge overlapping paired-end reads with PEAR to reconstruct full inserts carrying 3'-end information. Extract cell barcodes/UMIs, assign reads to cell types using MCA/HCL annotations, and align reads to hg38/mm10 with HISAT. Aggregate read 5' ends per cell type, call coverage peaks as CS candidates, then merge candidate peaks within 30 nt. Filter out low-abundance reads and peaks consistent with internal priming at genomic adenosine stretches, and restrict to protein-coding transcripts with verified 3' ends in GENCODE v39/vM21, excluding mRNA_end_NF-tagged transcripts. Validate by confirming the final GTF records contain the expected counts (mouse N=170,617; human N=150,191).

## Related tools

- **PEAR** (Merges overlapping paired-end Microwell-seq reads to reconstruct full insert sequences carrying 3'-end information)
- **HISAT** (Aligns the poly(A)-trimmed reads to hg38 and mm10 reference genomes prior to 5' end aggregation)
- **Snakemake** (Workflow engine hosting the de novo CS calling pipelines (Mayrlab/mca-utrome, Mayrlab/hcl-utrome)) — https://github.com/Mayrlab/mca-utrome
- **scUTRquant** (Downstream pipeline that consumes the resulting CS/UTRome annotations for 3'UTR isoform quantification) — https://github.com/Mayrlab/scUTRquant

## Examples

```
git clone git@github.com:Mayrlab/mca-utrome.git && snakemake --use-conda --cores 8
```

## Evaluation signals

- Final mouse CS candidate set contains N=170,617 coordinate-annotated GTF records
- Final human CS candidate set contains N=150,191 coordinate-annotated GTF records
- Merged candidate peaks are within the 30 nt merge distance and restricted to protein-coding GENCODE v39/vM21 transcripts with verified 3' ends
- mRNA_end_NF-tagged transcripts are excluded from the final CS sets
- Rigorous internal-priming QC is applied: peaks at genomic adenosine stretches and low-abundance reads are filtered out

## Limitations

- Peaks from internal priming at genomic adenosine stretches can mimic true cleavage sites and require explicit filtering
- De novo mapping requires reads that span the 3' end; tag-based protocols like 10x Genomics whose reads do not span CS cannot be used for this skill
- Low-abundance reads must be filtered, so very rare cell-type-specific CS may be missed

## Evidence

- [methods] CS candidates were produced by removing poly(A) tails from reads, mapping the remaining portions to the genome, and filtering out low-abundance reads and peaks from priming at genomic adenosine stretches.: [evidence span withheld — anchor status: not_found]
- [abstract] We map CS in more than 200 primary human and mouse cell types and increase CS annotations relative to the GENCODE database by 40%.: "We map CS in more than 200 primary human and mouse cell types and increase CS annotations relative to the GENCODE database by 40%."
- [readme] This reposistory hosts a Snakemake pipeline used to call 3' UTR cleavage sites *de novo* from [the Mouse Cell Atlas dataset](http://bis.zju.edu.cn/MCA/).: [evidence span withheld — anchor status: external_doc]
- [discussion] We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality: "We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality"
- [intro] most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS): "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS)"

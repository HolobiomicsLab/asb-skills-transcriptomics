---
name: paired-end-read-overlap-assembly
description: Use when processing paired-end, 3'-end tag scRNA-seq data (e.g. Microwell-seq
  FASTQ from Mouse Cell Atlas / Human Cell Landscape) where reads must overlap and
  be merged to recover the full insert containing the 3' cleavage site.
when_to_use_negative:
- 'Input is 10x Genomics 3''-end tag data: most reads do not span mRNA 3'' end cleavage
  sites, so overlap assembly cannot support exact de novo CS mapping (quantification
  against a known CS annotation, e.g. scUTRquant, is the correct approach instead).'
- Read pairs from short-insert protocols with no overlap cannot be merged, so overlap
  assembly would fail or produce partial inserts lacking the cleavage site.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PEAR
  role: Merges overlapping paired-end Microwell-seq reads to reconstruct full insert
    sequences carrying the 3'-end information
- name: HISAT
  role: Aligns merged reads to the hg38 and mm10 reference genomes prior to cleavage-site
    peak calling
- name: scUTRquant
  role: Downstream pipeline (built on kallisto-bustools) that consumes CS annotations
    for gene and 3'UTR isoform quantification; contrasted with de novo mapping from
    merged MWS reads
  repo: https://github.com/Mayrlab/scUTRquant
- name: Mayrlab/mca-utrome
  role: Snakemake pipeline that calls 3' UTR cleavage sites de novo from the Mouse
    Cell Atlas, producing the Mouse UTRome; contains the PEAR-based overlap assembly
    stage
  repo: https://github.com/Mayrlab/mca-utrome
- name: Mayrlab/hcl-utrome
  role: Pipeline to reprocess Human Cell Landscape data for cleavage-site identification
    using the same overlap-assembly approach
  repo: https://github.com/Mayrlab/hcl-utrome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:e2bc4a33b2dc401fd13c4aab1789cb17ad5b846466ee06a80d9816ad8a2ac8f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - HISAT
  - Mayrlab/hcl-utrome
  - Mayrlab/mca-utrome
  - PEAR
  - scUTRquant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# paired-end-read-overlap-assembly

## Summary

Reconstruct full insert sequences carrying mRNA 3'-end information by merging overlapping paired-end Microwell-seq reads (e.g. with PEAR) before downstream cleavage-site mapping. This is essential for MWS-style data where only full-span reads preserve the position of the cleavage site.

## When to use

Use when processing paired-end, 3'-end tag scRNA-seq data (e.g. Microwell-seq FASTQ from Mouse Cell Atlas / Human Cell Landscape) where reads must overlap and be merged to recover the full insert containing the 3' cleavage site.

## When NOT to use

- Input is 10x Genomics 3'-end tag data: most reads do not span mRNA 3' end cleavage sites, so overlap assembly cannot support exact de novo CS mapping (quantification against a known CS annotation, e.g. scUTRquant, is the correct approach instead).
- Read pairs from short-insert protocols with no overlap cannot be merged, so overlap assembly would fail or produce partial inserts lacking the cleavage site.

## Inputs

- Paired-end Microwell-seq FASTQ files (e.g. GEO:GSE108097 Mouse Cell Atlas, GEO:GSE134355 Human Cell Landscape)
- Cell annotations from MCA/HCL for downstream cell-type assignment
- Reference genomes hg38 and mm10 (HISAT indices)

## Outputs

- Merged (assembled) insert FASTQ sequences spanning the mRNA 3' cleavage site
- Extracted cell barcode/UMI assignments per read
- Genome-aligned reads (hg38/mm10) suitable for 5'-end peak calling and CS candidate discovery

## How to apply

Download the paired-end Microwell-seq FASTQ files (Mouse Cell Atlas, GEO:GSE108097; Human Cell Landscape, GEO:GSE134355) and merge overlapping read pairs with PEAR to reconstruct full insert sequences that carry the 3'-end information. After merging, extract cell barcodes/UMIs from the merged reads and assign reads to cell types using MCA/HCL cell annotations. Align the merged reads to hg38 and mm10 with HISAT, then aggregate read 5' ends per cell type to call coverage peaks as cleavage-site candidates. Rationale: unlike 10x Genomics data, where most reads do not span the cleavage site and prevent exact de novo mapping, MWS paired-end reads overlap at the 3' end, making overlap assembly a prerequisite for single-nucleotide-resolution CS mapping.

## Related tools

- **PEAR** (Merges overlapping paired-end Microwell-seq reads to reconstruct full insert sequences carrying the 3'-end information)
- **HISAT** (Aligns merged reads to the hg38 and mm10 reference genomes prior to cleavage-site peak calling)
- **scUTRquant** (Downstream pipeline (built on kallisto-bustools) that consumes CS annotations for gene and 3'UTR isoform quantification; contrasted with de novo mapping from merged MWS reads) — https://github.com/Mayrlab/scUTRquant
- **Mayrlab/mca-utrome** (Snakemake pipeline that calls 3' UTR cleavage sites de novo from the Mouse Cell Atlas, producing the Mouse UTRome; contains the PEAR-based overlap assembly stage) — https://github.com/Mayrlab/mca-utrome
- **Mayrlab/hcl-utrome** (Pipeline to reprocess Human Cell Landscape data for cleavage-site identification using the same overlap-assembly approach) — https://github.com/Mayrlab/hcl-utrome

## Evaluation signals

- Merged reads should retain cell barcodes/UMIs and their full insert including the 3' poly(A)-proximal region; after poly(A) tail removal, remaining portions should map uniquely to hg38 or mm10 with HISAT.
- Downstream CS candidate counts should match the expected validation numbers: mouse N=170,617 and human N=150,191 coordinate-annotated GTF records.
- Candidate peaks merged within 30 nt should be restricted to protein-coding transcripts with verified 3' ends in GENCODE v39/vM21, excluding mRNA_end_NF-tagged transcripts.
- Low-abundance reads and peaks caused by internal priming at genomic adenosine stretches should be filtered out before the CS set is finalized.

## Limitations

- Most 10x Genomics reads do not span mRNA 3' end cleavage sites, so the overlap-assembly approach is specific to Microwell-seq-style libraries with full-span reads.
- Internal priming at genomic adenosine stretches creates spurious peaks that must be filtered; unfiltered sets overestimate the CS catalog.
- Only protein-coding transcripts with verified 3' ends (excluding mRNA_end_NF-tagged transcripts) are retained, so 3' ends of noncoding or unverified transcripts are out of scope.

## Evidence

- [methods] PEAR merges overlapping paired-end MWS reads: [evidence span withheld — anchor status: not_found]
- [intro] 10x reads do not span cleavage sites: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS)"
- [methods] CS candidates from poly(A) removal and filtering: [evidence span withheld — anchor status: not_found]
- [readme] UTRome pipelines from MWS data: [evidence span withheld — anchor status: external_doc]

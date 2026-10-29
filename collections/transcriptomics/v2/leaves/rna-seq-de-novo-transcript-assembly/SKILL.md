---
name: rna-seq-de-novo-transcript-assembly
description: Use when the goal is to reconstruct full IGHV-D-J Ig transcripts from
  an RNA-seq BAM (e.g., for IGHV hypermutation status in CLL) rather than inferring
  the B-cell receptor repertoire from only a portion of the IGHV gene segment.
when_to_use_negative:
- The input BAM is not aligned to the hg38 (GRCh38) genome build or is not coordinate-sorted
  and indexed — CRIS explicitly requires this precondition
- The analysis can tolerate inferring the B-cell receptor repertoire from only a portion
  of the IGHV gene segment; CRIS's de novo assembly is aimed at complete IGHV-D-J
  reconstruction
- RNA-seq data have far fewer than 25M reads per sample, below the README's stated
  minimum for ribodepleted or polyA-selected RNA-seq
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CRIS
  role: Pipeline that extracts Ig-locus reads, runs Trinity assembly, and produces
    Ig transcript outputs; distributed as Docker image or bash script
  repo: https://github.com/Rashedul/CRIS
- name: sambamba
  role: Extracts RNA-seq reads aligned to putative immunoglobulin gene loci from the
    sorted BAM (v0.7.0)
- name: picard
  role: Picard SamToFastq (v2.20.3) converts extracted alignments to paired-end FASTQ
    input for Trinity
- name: trinity
  role: De novo assembles the extracted FASTQ reads into Ig transcript contigs (v2.1.1,
    threads=4, memory=16G)
- name: blast
  role: Downstream alignment of assembled contigs to reference Ig sequences (v2.9.0)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:234eca3f3e7f17772112fa58f94f5e705fed810be34756317f4c7adaec62af96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - blast
  - picard
  - sambamba
  - trinity
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rna-seq-de-novo-transcript-assembly

## Summary

De novo assembly of immunoglobulin transcripts from RNA-seq reads extracted at putative Ig loci, as performed in the CRIS pipeline using Trinity (v2.1.1), yielding Ig transcript contigs that carry IGHV-D-J sequence for downstream SHM analysis.

## When to use

Use when the goal is to reconstruct full IGHV-D-J Ig transcripts from an RNA-seq BAM (e.g., for IGHV hypermutation status in CLL) rather than inferring the B-cell receptor repertoire from only a portion of the IGHV gene segment.

## When NOT to use

- The input BAM is not aligned to the hg38 (GRCh38) genome build or is not coordinate-sorted and indexed — CRIS explicitly requires this precondition
- The analysis can tolerate inferring the B-cell receptor repertoire from only a portion of the IGHV gene segment; CRIS's de novo assembly is aimed at complete IGHV-D-J reconstruction
- RNA-seq data have far fewer than 25M reads per sample, below the README's stated minimum for ribodepleted or polyA-selected RNA-seq

## Inputs

- Coordinate-sorted, indexed BAM aligned to hg38 (GRCh38) with BWA (e.g., SRR1814049_test.bam)
- Paired-end FASTQ reads extracted from putative Ig loci (produced from the BAM via sambamba + Picard SamToFastq)

## Outputs

- De novo-assembled Ig transcript contigs (Trinity FASTA) carrying IGHV-D-J sequence
- Ig-transcript files sorted by expression, e.g. ig-transcripts.sortedbyTPM.txt (IDs + TPM) and ig-transcripts.sortedbyTPM.fasta (sequences)
- IgBLAST output (e.g., .IgBLAST_out.txt) with percent identity and alignment between Ig-transcript and top germline IGHV gene hits

## How to apply

First ensure the input BAM is aligned to the hg38 (GRCh38) genome build with BWA and is coordinate-sorted and indexed, as CRIS requires. Extract paired-end RNA-seq reads aligned to the putative immunoglobulin gene loci with sambamba (v0.7.0), then convert the extracted alignments to paired-end FASTQ with Picard SamToFastq (picard v2.20.3). Assemble the extracted FASTQ reads de novo with Trinity (v2.1.1) using CRIS default resource settings (threads=4, memory=16G; minimum 16GB RAM and 4 threads required). The resulting Ig transcript contigs are subsequently aligned to reference Ig sequences (e.g., with IgBLAST/BLAST) to enumerate and classify SHMs. Validate the assembly by confirming IGHV-D-J contigs are recovered; for the supplied SRR1814049_test.bam test run, CRIS should report IGHV gene IGHV3-74 at 94.6 percent identity.

## Related tools

- **CRIS** (Pipeline that extracts Ig-locus reads, runs Trinity assembly, and produces Ig transcript outputs; distributed as Docker image or bash script) — https://github.com/Rashedul/CRIS
- **sambamba** (Extracts RNA-seq reads aligned to putative immunoglobulin gene loci from the sorted BAM (v0.7.0))
- **picard** (Picard SamToFastq (v2.20.3) converts extracted alignments to paired-end FASTQ input for Trinity)
- **trinity** (De novo assembles the extracted FASTQ reads into Ig transcript contigs (v2.1.1, threads=4, memory=16G))
- **blast** (Downstream alignment of assembled contigs to reference Ig sequences (v2.9.0))

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- Assembly yields IGHV-D-J contigs rather than fragmented or non-Ig contigs
- Test run on SRR1814049_test.bam reports IGHV gene: IGHV3-74 and percent identity: 94.6
- Output files are produced with expected names: .IgBLAST_out.txt, .ig-transcripts.sortedbyTPM.txt, and .ig-transcripts.sortedbyTPM.fasta
- Ig-transcripts.sortedbyTPM.txt contains Ig-transcript sequence IDs sorted by TPM expression values, matching the FASTA in sortedbyTPM.fasta
- IgBLAST output contains percent identity and alignment between Ig-transcript and top germline IGHV gene hits

## Limitations

- Minimum 16GB RAM and 4 threads are required to run CRIS; threads=4 and memory=16G are the defaults
- Requires a minimum of 25M sequence reads per sample; both ribodepleted and polyA selected RNA-seq datasets are appropriate
- The input BAM must be aligned to hg38 (GRCh38) using BWA and be coordinate-sorted and indexed; alignments to other builds or mappers are out of scope
- CRIS was validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia; performance in other disease contexts is not established here

## Evidence

- [abstract] performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences: "performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [full_text] The paired-end reads extracted from the putative Ig loci are used as input for de novo transcriptome assembly with Trinity (v2.1.1): [evidence span withheld — anchor status: not_found]
- [readme] bam file must be aligned to hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed: [evidence span withheld — anchor status: not_found]
- [readme] expected mutational status for test run

- IGHV gene: IGHV3-74
- Percent identity: 94.6: "# expected mutational status for test run 

- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."

---
name: star-alignment-of-short-rna-reads
description: Use when a small RNA-seq FASTQ (adapter- and quality-trimmed by COMPSRA
  QC) must be mapped to hg19/hg38 to enable annotation of miRNA, piRNA, snRNA, snoRNA,
  tRNA and circRNA molecules with COMPSRA.
when_to_use_negative:
- The input is raw small RNA-seq FASTQ that has not been adapter-trimmed and quality-filtered
  by COMPSRA's QC module (adapter TGGAATTCTCGGGTGCCAAGG, -rb 4, -rh/-rt/-rr 20).
- The goal is microbial (archaea/bacteria/fungi/viruses) profiling only — that is
  the BLAST-based Microbe Module operating on reads that this alignment step could
  not map to hg19/hg38.
- The reads are long (>200 nt) transcript reads; COMPSRA/STAR here is configured for
  short (<200 nt) small RNA local end alignment.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: COMPSRA
  role: Wraps and invokes STAR in its Alignment Module; supplies QC output as alignment
    input and routes aligned/unmapped reads to the Annotation and Microbe modules
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default RNA sequence aligner used to map qualified small RNA reads to hg19/hg38
    with default small-RNA parameters
- name: Java
  role: Runtime (JRE 8+) required to execute COMPSRA.jar and its ToolKit commands
    for downloading STAR and building the hg38 index
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:0e7db8c2150d0129c28ae123a0c26ee531af89c7c0ed1601be87e40bdd128309
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# star-alignment-of-short-rna-reads

## Summary

Align quality-controlled small RNA-seq reads to the hg38 human reference genome with STAR, as the Alignment Module of the COMPSRA platform, producing the BAM file that feeds COMPSRA's Annotation Module and an unmapped-reads file that feeds the Microbe Module.

## When to use

Use when a small RNA-seq FASTQ (adapter- and quality-trimmed by COMPSRA QC) must be mapped to hg19/hg38 to enable annotation of miRNA, piRNA, snRNA, snoRNA, tRNA and circRNA molecules with COMPSRA.

## When NOT to use

- The input is raw small RNA-seq FASTQ that has not been adapter-trimmed and quality-filtered by COMPSRA's QC module (adapter TGGAATTCTCGGGTGCCAAGG, -rb 4, -rh/-rt/-rr 20).
- The goal is microbial (archaea/bacteria/fungi/viruses) profiling only — that is the BLAST-based Microbe Module operating on reads that this alignment step could not map to hg19/hg38.
- The reads are long (>200 nt) transcript reads; COMPSRA/STAR here is configured for short (<200 nt) small RNA local end alignment.

## Inputs

- COMPSRA QC-qualified small RNA-seq FASTQ file (length-grouped, e.g., sample01_17to50_FitRead.fastq.gz)
- hg38 reference genome and prebuilt STAR index (downloaded/built via COMPSRA ToolKit, -mbi)
- STAR executable installed in the COMPSRA plug directory

## Outputs

- STAR_Aligned.out.bam BAM file of genome-mapped reads for the Annotation Module
- STAR_Aligned...UnMapped.bam FASTA/BAM of reads not mapped to the human genome, for the Microbe Module
- Alignment QC/report files under the module output directory

## How to apply

Take the qualified-reads FASTQ produced by COMPSRA's QC module (e.g., sample01_17to50_FitRead.fastq.gz, after adapter TGGAATTCTCGGGTGCCAAGG trimming, -rb 4 random-base removal, -rh/-rt/-rr 20 quality filtering and -rlh 8,17 length grouping) and run the Alignment Module with -aln -mt star against -ref hg38. Before first use, obtain STAR and the reference genome via the ToolKit (java -jar COMPSRA.jar -tk -dr -ck star and -ck star_hg38) and build the genome index once with -mbi; the article notes this is run on a server with 30 GB RAM and index building costs about 3 hours. STAR uses default small-RNA parameters (local end alignment, --outFilterMismatchNmax 1, --outFilterMatchNmin 16), which are customizable on the command line. The result is a STAR_Aligned.out.bam for annotation against the six small RNA classes; reads that could not be mapped to the human genome are saved into a FASTA/BAM file for input into the Microbe Module.

## Related tools

- **COMPSRA** (Wraps and invokes STAR in its Alignment Module; supplies QC output as alignment input and routes aligned/unmapped reads to the Annotation and Microbe modules) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default RNA sequence aligner used to map qualified small RNA reads to hg19/hg38 with default small-RNA parameters)
- **Java** (Runtime (JRE 8+) required to execute COMPSRA.jar and its ToolKit commands for downloading STAR and building the hg38 index)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -aln -mt star -mbi -in ./example_out/sample01/sample01_17to50_FitRead.fastq.gz -out ./example_out/sample01/sample01_17to50_FitRead
```

## Evaluation signals

- STAR index was built on first run (-mbi) with hg38 downloaded via star_hg38; the command completes within ~3 hours on a 30 GB RAM server without memory errors.
- Output BAM (sample01_17to50_FitRead_STAR_Aligned.out.bam) exists and the downstream Annotation Module runs against it with -ac 1,2,3,4,5,6.
- The unmapped-reads file (STAR_Aligned_UnMapped.bam/FASTA) is produced for the Microbe Module, consistent with mapped + unmapped reads equaling the qualified input reads.
- Reproducing the 12 serum samples end-to-end yields annotation counts matching the reported 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs within a small tolerance.
- STAR parameters match the small-RNA defaults (--outFilterMismatchNmax 1, --outFilterMatchNmin 16, local end alignment) unless explicitly overridden.

## Limitations

- Building the STAR index on first use costs about 3 hours and requires a server with at least 30 GB RAM; if it fails, the STAR binary must be made executable via chmod.
- COMPSRA only supports hg19 and hg38 (plus rat rno5/rno6 for some databases) as reference genomes for STAR alignment.
- STAR parameters default to small-RNA settings; changing them may break comparability with published results such as the 12-serum-sample benchmark.
- The exact COMPSRA software version used in the article's results is not stated.

## Evidence

- [readme] COMPSRA uses STAR as its default RNA sequence aligner with default parameters which are customizable on the command line.: "COMPSRA uses STAR as its default RNA sequence aligner with default parameters which are customizable on the command line."
- [methods] Qualified reads from the QC module output are first mapped to the target genome (e.g., hg19/hg38), and then aligned reads are quantified and annotated in the Annotation Module: "Qualified reads from the QC module output are first mapped to the target genome (e.g., hg19/hg38), and then aligned reads are quantified and annotated in the Annotation Module"
- [readme] The qualified reads that were not mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database from UCSC using BLAST.: "The qualified reads that could not be mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database from UCSC using BLAST."
- [results] We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [readme] -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours.: "This process will cost about 3 hours."

---
name: blastn-expectation-threshold-filtering
description: Use when you have Trinity-assembled transcript contigs from RNA-seq and
  need to separate bona fide immunoglobulin IGHV-containing transcripts from non-Ig
  contigs and assign each to its closest germline IGHV gene.
when_to_use_negative:
- Input reads are unassembled FASTQ rather than assembled transcript contigs — the
  filter operates on Trinity contigs, not raw reads
- The goal is full Ig repertoire profiling across many clonotypes rather than reconstructing
  the dominant IGHV sequence from bulk RNA-seq
- The input BAM is not aligned to hg38 (GRCh38) and coordinate-sorted/indexed, which
  CRIS requires before Ig-locus read extraction
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2830
tools:
- name: BLAST
  role: blastn (v2.9.0) aligns Trinity-assembled contigs against the custom IMGT IGHV
    reference with E-value cutoff 20 to identify Ig-transcripts and their top germline
    IGHV hits
  repo: https://github.com/Rashedul/CRIS
- name: Trinity
  role: de novo transcript assembly (v2.1.1) upstream of the blastn filtering step,
    producing the contig FASTA that is screened against IMGT IGHV
  repo: https://github.com/Rashedul/CRIS
- name: Salmon
  role: quantifies the blastn-filtered Ig-transcripts (v0.8.1, k-mer 31 bp) to produce
    TPM values used to rank and flag the dominant clone
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:1aaeb84e1360efee5d84cf50da837b33910a3fd82d6390139a69ed4a858a7386
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST
  - CRIS
  - Salmon
  - Trinity
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# blastn-expectation-threshold-filtering

## Summary

Identify immunoglobulin transcripts among de novo assembled contigs by running BLAST+ (v2.9.0) blastn against a custom IMGT IGHV reference database with an expectation value (E-value) cutoff of 20, recording each transcript's top germline IGHV hit for downstream TPM quantification and dominant-clone selection.

## When to use

Use when you have Trinity-assembled transcript contigs from RNA-seq and need to separate bona fide immunoglobulin IGHV-containing transcripts from non-Ig contigs and assign each to its closest germline IGHV gene.

## When NOT to use

- Input reads are unassembled FASTQ rather than assembled transcript contigs — the filter operates on Trinity contigs, not raw reads
- The goal is full Ig repertoire profiling across many clonotypes rather than reconstructing the dominant IGHV sequence from bulk RNA-seq
- The input BAM is not aligned to hg38 (GRCh38) and coordinate-sorted/indexed, which CRIS requires before Ig-locus read extraction

## Inputs

- Trinity-assembled Ig candidate transcript FASTA (from CRIS upstream de novo assembly of RNA-seq reads extracted at Ig loci)
- Custom IMGT IGHV germline reference database formatted for BLAST
- Input RNA-seq BAM aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed (upstream context)

## Outputs

- Set of Ig-transcripts identified as IGHV-containing with their top germline IGHV hits (percent identity and alignment, later in the .IgBLAST_out.txt)
- Ig-transcript FASTA passed onward to Salmon quantification
- TPM-sorted table of Ig-transcript sequence IDs (*.ig-transcripts.sortedbyTPM.txt) and TPM-sorted FASTA (*.ig-transcripts.sortedbyTPM.fasta)

## How to apply

Take the FASTA of Ig-candidate transcripts produced by the upstream Trinity (v2.1.1) de novo assembly step and run blastn from BLAST (v2.9.0) against a custom IMGT IGHV reference database, using an expectation value cutoff of 20 to retain Ig-transcripts and capture their top germline IGHV hits. The lenient E-value threshold keeps low-abundance or somatically hypermutated (SHM) BCR transcripts that would be missed by a strict cutoff. Pass the filtered Ig-transcript FASTA to Salmon (v0.8.1, k-mer 31 bp) for TPM quantification; the blastn hit annotation travels with each transcript so the highest-TPM productive clone can later be labeled with its IGHV gene and percent identity. Verify the filter by checking that the resulting TPM-sorted table lists Ig-transcript sequence IDs ordered by TPM and that, on the CRIS test dataset SRR1814049, the call resolves to IGHV3-74 at 94.6 percent identity.

## Related tools

- **BLAST** (blastn (v2.9.0) aligns Trinity-assembled contigs against the custom IMGT IGHV reference with E-value cutoff 20 to identify Ig-transcripts and their top germline IGHV hits) — https://github.com/Rashedul/CRIS
- **Trinity** (de novo transcript assembly (v2.1.1) upstream of the blastn filtering step, producing the contig FASTA that is screened against IMGT IGHV) — https://github.com/Rashedul/CRIS
- **Salmon** (quantifies the blastn-filtered Ig-transcripts (v0.8.1, k-mer 31 bp) to produce TPM values used to rank and flag the dominant clone)

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- The blastn-filtered set contains only contigs with a germline IGHV hit, each annotated with its top IMGT IGHV gene and percent identity/alignment
- The downstream TPM-sorted table (SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt) exists and contains Ig-transcript sequence IDs ordered by TPM values
- On the supplied test BAM SRR1814049_test.bam, the pipeline calls IGHV3-74 with percent identity 94.6
- The companion TPM-sorted FASTA (SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta) contains Ig-transcript sequences in the same TPM-sorted order as the table
- The highest-TPM transcript is flagged as the dominant clone, consistent with the reported IGHV3-74 assignment

## Limitations

- The E-value cutoff of 20 is deliberately permissive; transcripts passing the filter still require downstream TPM ranking and productivity/percent-identity checks before the dominant clone is called
- CRIS targets the immunoglobulin heavy chain variable region (IGHV) from bulk RNA-seq and classifies SHM status; it is not a full multi-locus Ig repertoire profiler
- Requires a minimum of 25M reads per RNA-seq sample (ribodepleted or polyA selected), 16GB RAM and 4 threads

## Evidence

- [methods] blastn/IMGT Ig-transcript identification and Salmon TPM quantification: [evidence span withheld — anchor status: not_found]
- [abstract] IMGT IGHV reference database and E-value cutoff: "performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [readme] TPM-sorted output and dominant-clone flagging: "SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt 
Contains Ig-transcript sequence IDs and sorted by expression (TPM) values."
- [methods] Test-run expected IGHV call used for validation: [evidence span withheld — anchor status: not_found]
- [readme] Dependency list confirming blast and salmon versions: "Dependencies: ```picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1), igblast (v1.14.0), jellyfish (v2.2.10)```"

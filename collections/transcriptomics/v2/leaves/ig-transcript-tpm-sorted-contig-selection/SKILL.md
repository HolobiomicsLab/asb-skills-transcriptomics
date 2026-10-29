---
name: ig-transcript-tpm-sorted-contig-selection
description: Use when a CRIS run has produced Ig-transcript outputs (.ig-transcripts.sortedbyTPM.txt/.fasta
  and .IgBLAST_out.txt) and the top-expressed contig must be selected and assigned
  to a germline IGHV gene (e.g. IGHV3-74) for percent-identity comparison.
when_to_use_negative:
- Input is a raw RNA-seq BAM or FASTQ that has not been processed by CRIS — no TPM-sorted
  Ig-transcript files exist yet, so run CRIS first (requires ≥25M reads, ≥16GB RAM,
  4 threads).
- The goal is bulk B-cell receptor repertoire profiling of many clonotypes rather
  than selecting a single top-expressed IGHV transcript for hypermutation status classification.
- The BAM is not aligned to the hg38 (GRCh38) genome build with BWA and coordinate-sorted/indexed
  — CRIS extraction of Ig-locus reads and downstream TPM ranking are not valid on
  other builds.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2830
tools:
- name: CRIS
  role: Produces the Ig-transcript files sorted by TPM from RNA-seq BAM; supplies
    the contigs being selected
  repo: https://github.com/Rashedul/CRIS
- name: BLAST
  role: Aligns the selected top-TPM contig to germline Ig reference sequences for
    IGHV gene assignment (v2.9.0)
- name: IgBLAST
  role: Reports percent identity and alignment between the Ig-transcript and top germline
    IGHV gene hits (v1.14.0)
- name: seqkit
  role: Used within the CRIS pipeline for sequence handling of the TPM-sorted FASTA
    contigs (v0.12.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:7eb8a778653a156205d51187fddeb46ebd2782a9b5b6f5fcf32af9ee665f9402
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST
  - CRIS
  - IgBLAST
  - seqkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ig-transcript-tpm-sorted-contig-selection

## Summary

Identifies the top-expressed immunoglobulin transcript contig from CRIS output by ranking Ig-transcript FASTA sequences by TPM expression values, then assigning the selected contig to its germline IGHV gene via BLAST/IgBLAST alignment. This selection step is what determines the IGHV sequence used for downstream percent-identity and SHM classification.

## When to use

Use when a CRIS run has produced Ig-transcript outputs (.ig-transcripts.sortedbyTPM.txt/.fasta and .IgBLAST_out.txt) and the top-expressed contig must be selected and assigned to a germline IGHV gene (e.g. IGHV3-74) for percent-identity comparison.

## When NOT to use

- Input is a raw RNA-seq BAM or FASTQ that has not been processed by CRIS — no TPM-sorted Ig-transcript files exist yet, so run CRIS first (requires ≥25M reads, ≥16GB RAM, 4 threads).
- The goal is bulk B-cell receptor repertoire profiling of many clonotypes rather than selecting a single top-expressed IGHV transcript for hypermutation status classification.
- The BAM is not aligned to the hg38 (GRCh38) genome build with BWA and coordinate-sorted/indexed — CRIS extraction of Ig-locus reads and downstream TPM ranking are not valid on other builds.

## Inputs

- CRIS output file '<sample>.ig-transcripts.sortedbyTPM.txt' (Ig-transcript sequence IDs sorted by expression TPM values)
- CRIS output file '<sample>.ig-transcripts.sortedbyTPM.fasta' (Ig-transcript FASTA sequences sorted by TPM)
- CRIS output file '<sample>.IgBLAST_out.txt' (percent identity and alignment between Ig-transcript and top germline IGHV hits)
- Germline immunoglobulin reference sequences (for BLAST v2.9.0 / IgBLAST v1.14.0 assignment)

## Outputs

- Top-expressed Ig transcript contig ID and its nucleotide sequence
- Germline IGHV gene assignment (e.g. IGHV3-74) for the selected contig
- Reconstruction length and percent identity of the selected contig against the assigned IGHV germline allele (e.g. TSV rows for CRIS vs V'DJer comparison)

## How to apply

After running CRIS on an hg38-aligned, coordinate-sorted BAM, read the '.ig-transcripts.sortedbyTPM.txt' file, which contains Ig-transcript sequence IDs ranked by expression (TPM) values, and take the top-ranked contig as the top-expressed Ig transcript. Use the corresponding '.ig-transcripts.sortedbyTPM.fasta' file to obtain the contig nucleotide sequence, and align it to germline Ig reference sequences with BLAST v2.9.0 / IgBLAST v1.14.0 to assign the germline IGHV gene (e.g. IGHV3-74). From the '.IgBLAST_out.txt' output, extract the percent identity and alignment between the Ig-transcript and the top germline IGHV hit. In benchmark settings such as the US-1422368 comparison, record the reconstruction length and percent identity of the selected contig (e.g. CRIS 295/296 bp at 91.2% identity versus a 226 bp V'DJer contig at 89.4%) so partial versus full-length IGHV reconstruction effects can be quantified. Ranking by TPM is the decision point: it grounds gene assignment in expression evidence rather than arbitrary contig order.

## Related tools

- **CRIS** (Produces the Ig-transcript files sorted by TPM from RNA-seq BAM; supplies the contigs being selected) — https://github.com/Rashedul/CRIS
- **BLAST** (Aligns the selected top-TPM contig to germline Ig reference sequences for IGHV gene assignment (v2.9.0))
- **IgBLAST** (Reports percent identity and alignment between the Ig-transcript and top germline IGHV gene hits (v1.14.0))
- **seqkit** (Used within the CRIS pipeline for sequence handling of the TPM-sorted FASTA contigs (v0.12.0))

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G && head -1 SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt
```

## Evaluation signals

- The chosen contig ID corresponds to the first entry in '<sample>.ig-transcripts.sortedbyTPM.txt' (highest TPM), not an arbitrary row of the FASTA.
- The IgBLAST output assigns a single top germline IGHV gene (test run expected: IGHV3-74) with a reported percent identity (test run expected: 94.6%).
- Reported reconstruction length is consistent with the contig sequence in the sorted FASTA (e.g. 295 bp CRIS contig against the 296 bp IGHV3-74*03 reference at 91.2% identity).
- Percent-identity comparisons to other assemblers reproduce benchmark deltas attributable to reconstruction length (CRIS 91.2% vs V'DJer 89.4% for US-1422368, with the 69 bp extension containing two mutations).
- Mutational status derived from the selected contig is consistent with CRIS's expected output format listing IGHV gene and percent identity.

## Limitations

- Selection assumes the top-TPM Ig transcript represents the clinically relevant clonotype; CRIS targets IGHV hypermutation status classification (e.g. CLL) rather than full repertoire diversity.
- Identity values depend on reconstruction length: partial reconstruction (as with V'DJer's 226 bp) lowers percent identity relative to near-full-length reconstruction (CRIS's 295/296 bp), so comparisons are only meaningful at matched reference coordinates.
- CRIS requires a minimum of 25M sequence reads per sample and both ribodepleted and polyA-selected RNA-seq datasets are supported; low-depth samples may not yield a reliable top-expressed contig.

## Evidence

- [readme] CRIS output files: "SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt Contains Ig-transcript sequence IDs and sorted by expression (TPM) values."
- [readme] TPM-sorted FASTA: "SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta Contains Ig-transcript fasta sequences sorted by expression (TPM) values."
- [readme] IgBLAST output content: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
- [abstract] Assembly and alignment workflow: "performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [methods] Test run expectation: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [abstract] Pipeline description: "CRIS extracts RNA-seq reads aligned to Ig gene loci"

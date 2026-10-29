---
name: ighv-germline-alignment-percent-identity-computation
description: Use when you have de novo assembled Ig-transcript contigs from RNA-seq
  and need to assign each contig to a germline IGHV gene, compute percent identity
  to the germline allele, or benchmark whether reconstruction length affects reported
  identity.
when_to_use_negative:
- Input is raw reads rather than assembled Ig-transcript contigs — the identity computation
  requires de novo assembly output (e.g. CRIS/trinity contigs) first
- BAM is aligned to a build other than hg38 (GRCh38) or is not coordinate-sorted and
  indexed
- Sample RNA-seq has fewer than 25M reads, below CRIS's stated minimum for reliable
  reconstruction
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3396
tools:
- name: CRIS
  role: Produces the full-length IGHV transcript contigs and the TPM-sorted Ig-transcript
    files whose identity is computed
  repo: https://github.com/Rashedul/CRIS
- name: IgBLAST
  role: Aligns the assembled Ig-transcript contig to germline IGHV genes and reports
    percent identity and alignment (version 1.14.0)
- name: BLAST
  role: Aligns reconstructed contigs to germline Ig reference sequences to assign
    the IGHV gene (version 2.9.0)
- name: seqkit
  role: FASTA/sequence utilities for the Ig-transcript contigs in the CRIS pipeline
    (version 0.12.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:a93cbc7b90884b3f071ef193dedb583f318f532c7719c5347299708784f9312d
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3396
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ighv-germline-alignment-percent-identity-computation

## Summary

Assigns reconstructed Ig-transcript contigs to their germline IGHV gene (e.g. IGHV3-74*03) by aligning to germline Ig references with BLAST/IgBLAST and computes reconstruction length and percent identity, the metric CRIS uses for SHM status and the benchmark used to show full-length versus partial IGHV reconstruction changes identity (295/296 bp at 91.2% vs 226 bp at 89.4%).

## When to use

Use when you have de novo assembled Ig-transcript contigs from RNA-seq and need to assign each contig to a germline IGHV gene, compute percent identity to the germline allele, or benchmark whether reconstruction length affects reported identity.

## When NOT to use

- Input is raw reads rather than assembled Ig-transcript contigs — the identity computation requires de novo assembly output (e.g. CRIS/trinity contigs) first
- BAM is aligned to a build other than hg38 (GRCh38) or is not coordinate-sorted and indexed
- Sample RNA-seq has fewer than 25M reads, below CRIS's stated minimum for reliable reconstruction

## Inputs

- Coordinate-sorted, indexed RNA-seq BAM aligned to hg38 (GRCh38) using BWA
- CRIS output FASTA of Ig-transcript sequences sorted by TPM (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta)
- Germline Ig reference sequences (IGHV alleles such as IGHV3-74*03)
- IgBLAST/BLAST alignment output (e.g. SRR1814049_test.bam.IgBLAST_out.txt)

## Outputs

- Assigned germline IGHV gene per contig with alignment details
- Percent identity of each reconstructed contig against its germline allele
- Reconstruction length in bp (e.g. 295/296 bp for CRIS vs 226 bp for V'DJer on US-1422368)
- Comparison table such as US-1422368_IGHV_reconstruction_comparison.tsv

## How to apply

After de novo assembly (e.g. CRIS with trinity v2.1.1), identify the top-expressed Ig transcript contig from the output FASTA sorted by TPM (ig-transcripts.sortedbyTPM.fasta / .txt). Align the contig to germline Ig reference sequences using BLAST v2.9.0 and/or IgBLAST v1.14.0; IgBLAST's output file (e.g. SRR1814049_test.bam.IgBLAST_out.txt) contains the percent identity and alignment between the Ig transcript and the top germline IGHV gene hits. From the IgBLAST alignment, extract the alignment length and percent identity against the assigned allele (e.g. IGHV3-74*03). For benchmarking partial versus full-length reconstruction, compute the same length/identity pair for both tools' contigs and attribute identity differences to the additional bases — e.g. the extra 69 bp of the full-length CRIS reconstruction contained two mutations, explaining the 1.2% identity difference over V'DJer. Run CRIS on a coordinate-sorted, indexed hg38 BAM with default 4 threads and 16G assembly memory.

## Related tools

- **CRIS** (Produces the full-length IGHV transcript contigs and the TPM-sorted Ig-transcript files whose identity is computed) — https://github.com/Rashedul/CRIS
- **IgBLAST** (Aligns the assembled Ig-transcript contig to germline IGHV genes and reports percent identity and alignment (version 1.14.0))
- **BLAST** (Aligns reconstructed contigs to germline Ig reference sequences to assign the IGHV gene (version 2.9.0))
- **seqkit** (FASTA/sequence utilities for the Ig-transcript contigs in the CRIS pipeline (version 0.12.0))

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- CRIS test run on SRR1814049_test.bam must report IGHV gene IGHV3-74 with percent identity 94.6
- SRR1814049_test.bam.IgBLAST_out.txt exists and contains percent identity plus the alignment between Ig-transcript and top germline IGHV gene hits
- For US-1422368, computed values must reproduce CRIS 295/296 bp at 91.2% identity and V'DJer 226 bp at 89.4% for IGHV3-74*03
- The 69 bp extension in the full-length CRIS reconstruction must contain two mutations that account for the 1.2% identity difference
- The chosen contig must be the top-expressed Ig transcript in ig-transcripts.sortedbyTPM.txt (sorted by TPM)

## Limitations

- Partial IGHV reconstruction (as produced by V'DJer, covering only a portion of the IGHV gene segment) yields lower reported percent identity than full-length reconstruction, so identity values are not comparable across tools with different reconstruction extents
- Requires a minimum of 25M sequence reads per sample; both ribodepleted and polyA selected RNA-seq are acceptable
- Requires 16GB RAM and 4 threads minimum, and the BAM must be hg38-aligned, coordinate-sorted and indexed

## Evidence

- [abstract] aligns the resulting contigs to reference Ig sequences to enumerate and classify SHMs in the IGHV gene sequence: "performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [readme] IgBLAST output contains percent identity and alignment to top germline hits: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
- [methods] Expected test-run identity: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [intro] BAM must be hg38-aligned, coordinate-sorted and indexed: [evidence span withheld — anchor status: not_found]

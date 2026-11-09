---
name: transcript-abundance-quantification
description: Use when a de novo Trinity assembly has produced candidate Ig transcripts
  that must be abundance-quantified and ranked by TPM to identify the dominant B-cell
  clone from RNA-seq data.
when_to_use_negative:
- Input is raw RNA-seq reads rather than assembled, Ig-filtered transcript contigs
  — Salmon quantification here is applied only after Trinity assembly and blastn/IMGT
  filtering
- Input BAM is not aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed
  — upstream CRIS steps will fail before quantification is reached
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Salmon
  role: Quantifies Ig-transcript abundances from the blastn-filtered Trinity contigs
    using k-mer size 31; produces the TPM values used to rank transcripts and identify
    the dominant clone
  repo: https://github.com/Rashedul/CRIS
- name: BLAST
  role: blastn (v2.9.0) against the custom IMGT IGHV database with expectation cutoff
    20 identifies which assembled contigs are Ig transcripts prior to Salmon quantification
- name: Trinity
  role: De novo assembles the Ig transcripts (v2.1.1) that become the input transcript
    set for TPM quantification
- name: CRIS
  role: Wrapper pipeline that chains assembly, blastn filtering, Salmon TPM quantification,
    and dominant-clone flagging
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:5a81aacb4c7bebf707e66fffe10abbd08fbb6b5b0820e14b1283f441b82b58bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST
  - CRIS
  - Salmon
  - Trinity
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# transcript-abundance-quantification

## Summary

Quantifies Ig-transcript abundances with Salmon (v0.8.1) using a 31 bp k-mer after blastn filtering against IMGT IGHV references, then ranks transcripts by TPM to flag the highest-expressing productive transcript as the dominant clone in the CRIS pipeline.

## When to use

Use when a de novo Trinity assembly has produced candidate Ig transcripts that must be abundance-quantified and ranked by TPM to identify the dominant B-cell clone from RNA-seq data.

## When NOT to use

- Input is raw RNA-seq reads rather than assembled, Ig-filtered transcript contigs — Salmon quantification here is applied only after Trinity assembly and blastn/IMGT filtering
- Input BAM is not aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed — upstream CRIS steps will fail before quantification is reached

## Inputs

- Trinity-assembled, blastn-filtered Ig transcript fasta (candidate immunoglobulin transcripts with top germline IGHV hits)
- Custom IMGT IGHV reference database used by blastn (BLAST v2.9.0) for Ig-transcript identification

## Outputs

- Ig-transcript table sorted by expression TPM values (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt with sequence IDs ordered by TPM)
- Ig-transcript fasta sorted by expression TPM values (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta)
- Dominant clone call: the highest-TPM productive transcript with its IGHV assignment and percent identity

## How to apply

Take the Ig-transcript fasta produced by the upstream Trinity de novo assembly and blastn (BLAST v2.9.0) filtering against the custom IMGT IGHV reference database (expectation value cutoff 20). Quantify these filtered Ig transcripts with Salmon v0.8.1 using a k-mer size of 31 bp, which is appropriate for short assembled contig quantification. Rank the Ig transcripts by TPM (transcripts per million) and write both a TPM-sorted table (sequence IDs ordered by TPM) and a TPM-sorted fasta. Mark the highest-TPM productive transcript as the dominant clone; for the CRIS test dataset (SRR1814049) the expected dominant call is IGHV3-74 at 94.6 percent identity.

## Related tools

- **Salmon** (Quantifies Ig-transcript abundances from the blastn-filtered Trinity contigs using k-mer size 31; produces the TPM values used to rank transcripts and identify the dominant clone) — https://github.com/Rashedul/CRIS
- **BLAST** (blastn (v2.9.0) against the custom IMGT IGHV database with expectation cutoff 20 identifies which assembled contigs are Ig transcripts prior to Salmon quantification)
- **Trinity** (De novo assembles the Ig transcripts (v2.1.1) that become the input transcript set for TPM quantification)
- **CRIS** (Wrapper pipeline that chains assembly, blastn filtering, Salmon TPM quantification, and dominant-clone flagging) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- The TPM-sorted table exists and contains Ig-transcript sequence IDs ordered by expression (TPM) values
- The TPM-sorted fasta exists with sequences in the same TPM order as the table
- For the supplied test dataset (SRR1814049_test.bam), the dominant-clone call is consistent with the reported IGHV3-74 assignment at 94.6 percent identity
- Salmon was run with k-mer size 31 on only the blastn-filtered Ig transcripts, not the full Trinity assembly

## Limitations

- Minimum of 25M sequence reads per sample is recommended for the CRIS pipeline overall; low-depth libraries may yield unreliable TPM ranking
- Both ribodepleted and polyA selected RNA-seq datasets are appropriate; other library preparations are not discussed
- Requires 16GB RAM and 4 threads; executables (salmon v0.8.1, blast v2.9.0, trinity v2.1.1, etc.) must be accessible from the user's PATH
- TPM ranking flags only the single highest-TPM productive transcript as the dominant clone; polyclonal repertoires are not fully characterized by this step

## Evidence

- [methods] The Ig transcripts resultant from the blastn filtering were quantified with Salmon (v0.8.1) using a k-mer of 31 bp, and the transcript with the highest TPM (transcripts per million) value was marked as the dominant clone.: [evidence span withheld — anchor status: not_found]
- [readme] Contains Ig-transcript sequence IDs and sorted by expression (TPM) values.: "Contains Ig-transcript sequence IDs and sorted by expression (TPM) values."
- [methods] Run blastn (BLAST v2.9.0) against the custom IMGT IGHV reference database using an expectation value cutoff of 20 to identify Ig-transcripts and their top germline IGHV hits.: [evidence span withheld — anchor status: not_found]
- [methods] Test run expected IGHV gene: IGHV3-74 with percent identity 94.6: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"

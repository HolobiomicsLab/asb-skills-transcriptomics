---
name: sanger-concordance-benchmarking
description: Use when validating a computationally inferred IGHV mutation status or
  dominant-clone IGHV gene call against available clinical Sanger-based classification
  for the same CLL samples.
when_to_use_negative:
- Input BAMs are not aligned to hg38 (GRCh38) with BWA or are not coordinate-sorted/indexed
  — CRIS requires this exact pre-processing
- No independent clinical Sanger-based IGHV classification exists for the samples,
  so there is no ground truth to benchmark against
- The goal is general B-cell receptor repertoire reconstruction rather than validating
  binary SHM status against a clinical standard
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2830
tools:
- name: CRIS
  role: Reconstructs the immunoglobulin IGHV-D-J sequences from RNA-seq and predicts
    SHM status; its output is what is benchmarked against Sanger calls
  repo: https://github.com/Rashedul/CRIS
- name: igblast
  role: Produces the per-sample IgBLAST output file with percent identity and top
    germline IGHV gene hits used to assign mCLL/uCLL status
- name: salmon
  role: Quantifies Ig-transcript expression (TPM) so the dominant (highest-TPM) clone
    can be identified for SHM classification
- name: trinity
  role: De novo assembly of Ig transcripts whose contigs are aligned to reference
    Ig sequences
- name: sambamba
  role: Coordinate-sorts and validates BAM inputs prior to running CRIS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:978f02e4e088d996716db47df8fa0194c14d6f293558fbe27ed2fc4145cdd77b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - Docker
  - blast
  - igblast
  - picard
  - salmon
  - sambamba
  - seqkit
  - trinity
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# sanger-concordance-benchmarking

## Summary

Benchmarks RNA-seq-derived IGHV hypermutation status (mCLL/uCLL) and dominant-clone IGHV gene assignments from CRIS against clinical PCR-Sanger classification, treating Sanger calls as ground truth for CLL cohorts.

## When to use

Use when validating a computationally inferred IGHV mutation status or dominant-clone IGHV gene call against available clinical Sanger-based classification for the same CLL samples.

## When NOT to use

- Input BAMs are not aligned to hg38 (GRCh38) with BWA or are not coordinate-sorted/indexed — CRIS requires this exact pre-processing
- No independent clinical Sanger-based IGHV classification exists for the samples, so there is no ground truth to benchmark against
- The goal is general B-cell receptor repertoire reconstruction rather than validating binary SHM status against a clinical standard

## Inputs

- BAM files aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed, one per cohort sample
- Clinical PCR-Sanger IGHV mutation classification and dominant IGHV gene assignments per sample
- CRIS conda environment (environment.txt) providing picard, trinity, blast, seqkit, sambamba, salmon, igblast, jellyfish

## Outputs

- Per-sample mCLL/uCLL SHM status calls
- Per-sample dominant-clone IGHV gene and percent identity (from IgBLAST output)
- Compiled concordance table, e.g. EGAD00001004046_ighv_calls.tsv, with per-sample agreement vs clinical calls

## How to apply

For each cohort sample, run CRIS on a hg38 (GRCh38)-aligned, coordinate-sorted BAM (bash CRIS.sh -inbam <sample>.bam -outdir <dir>, default 4 threads and 16G assembly memory). Parse the per-sample IgBLAST output file to extract the percent identity and top germline IGHV gene hit for the dominant (highest-TPM) Ig transcript from the TPM-sorted transcript list. Assign somatic hypermutation status as mCLL when the dominant-clone percent identity to the germline IGHV gene is low (hypermutated) versus uCLL, using the full IGHV gene segment reconstructed by CRIS rather than a partial segment. Compile per-sample dominant-clone IGHV gene, percent identity, and mCLL/uCLL calls into a TSV (e.g. EGAD00001004046_ighv_calls.tsv), then compute per-sample agreement with the clinical PCR-Sanger classification. Concordance is judged at the level of both the binary SHM status call and the exact IGHV gene identity; 100% agreement on a cohort (as achieved on EGAD00001004046) constitutes a successful replication.

## Related tools

- **CRIS** (Reconstructs the immunoglobulin IGHV-D-J sequences from RNA-seq and predicts SHM status; its output is what is benchmarked against Sanger calls) — https://github.com/Rashedul/CRIS
- **igblast** (Produces the per-sample IgBLAST output file with percent identity and top germline IGHV gene hits used to assign mCLL/uCLL status)
- **salmon** (Quantifies Ig-transcript expression (TPM) so the dominant (highest-TPM) clone can be identified for SHM classification)
- **trinity** (De novo assembly of Ig transcripts whose contigs are aligned to reference Ig sequences)
- **sambamba** (Coordinate-sorts and validates BAM inputs prior to running CRIS)

## Examples

```
bash CRIS.sh -inbam /fullPath/SRR1814049_test.bam -outdir /fullPath/
```

## Evaluation signals

- CRIS mCLL/uCLL SHM status matches clinical Sanger classification in 100% of cohort samples
- CRIS-identified dominant-clone IGHV gene is identical to the clinical call in every case
- Each sample's IgBLAST output file yields a percent identity to the top germline IGHV gene that falls clearly on one side of the mutated/unmutated threshold
- Dominant Ig transcript is the highest-TPM entry in the .ig-transcripts.sortedbyTPM.txt file
- Test-bam sanity check: SRR1814049_test.bam returns IGHV3-74 with percent identity 94.6

## Limitations

- Requires a minimum of 25M sequence reads per sample and 16GB RAM / 4 threads; both ribodepleted and polyA-selected RNA-seq are supported
- Input BAMs must be aligned to hg38 (GRCh38) using BWA, coordinate-sorted and indexed; other alignments are unsupported
- Concordance was demonstrated specifically in chronic lymphocytic leukemia cohorts; performance outside CLL or against non-Sanger references is not established here

## Evidence

- [abstract] We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts: "We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
- [readme] CRIS validated against Sanger: "CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia."
- [readme] README output expectations: "# expected mutational status for test run 

- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [readme] Read extraction from IgBLAST output: "SRR1814049_test.bam.IgBLAST_out.txt
Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
- [intro] Input requirements: [evidence span withheld — anchor status: not_found]

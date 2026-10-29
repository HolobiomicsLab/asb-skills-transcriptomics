---
name: cohort-scale-rnaseq-batch-processing
description: Use when a multi-sample CLL RNA-seq cohort (BAMs aligned to hg38 with
  BWA) must be run through CRIS one sample at a time to produce cohort-level IGHV
  mutation-status calls for comparison with clinical Sanger classification.
when_to_use_negative:
- Input BAMs are aligned to a genome build other than hg38 (GRCh38) or are not coordinate-sorted/indexed
  — CRIS requires hg38 BWA alignments
- Samples have fewer than 25M sequence reads, below CRIS's stated minimum for ribodepleted
  or polyA-selected RNA-seq
- The goal is variant calling outside the immunoglobulin loci or T-cell receptor repertoire
  analysis — CRIS reconstructs Ig heavy-chain V-D-J specifically
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: CRIS
  role: Core pipeline run per cohort sample to reconstruct IGHV sequences and predict
    SHM status from RNA-seq
  repo: https://github.com/Rashedul/CRIS
- name: sambamba
  role: Verifies cohort BAMs are coordinate-sorted and indexed before CRIS execution
- name: picard
  role: CRIS dependency used during per-sample processing (v2.20.3)
- name: trinity
  role: CRIS dependency performing de novo assembly of Ig transcripts per sample (v2.1.1)
- name: blast
  role: CRIS dependency aligning assembled contigs to reference Ig sequences (v2.9.0)
- name: seqkit
  role: CRIS dependency for sequence manipulation (v0.12.0)
- name: igblast
  role: Produces per-sample IgBLAST output used to extract top germline IGHV gene
    hit and percent identity (v1.14.0)
- name: salmon
  role: CRIS dependency quantifying Ig transcript TPM used to select the dominant
    clone (v0.8.1)
- name: Docker
  role: Alternative containerized execution mode for running CRIS on each cohort BAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:77caeb440234b63384c89e0dc74c9ea5ecf3edb8c621ca0dc87364f3f29af226
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# cohort-scale-rnaseq-batch-processing

## Summary

Batch-apply the CRIS IGHV reconstruction pipeline across every sample of a cohort RNA-seq study (e.g. EGAD00001004046 CLL BAMs) and compile per-sample dominant-clone IGHV gene, percent identity, and mCLL/uCLL calls into one comparison table against clinical classifications.

## When to use

Use when a multi-sample CLL RNA-seq cohort (BAMs aligned to hg38 with BWA) must be run through CRIS one sample at a time to produce cohort-level IGHV mutation-status calls for comparison with clinical Sanger classification.

## When NOT to use

- Input BAMs are aligned to a genome build other than hg38 (GRCh38) or are not coordinate-sorted/indexed — CRIS requires hg38 BWA alignments
- Samples have fewer than 25M sequence reads, below CRIS's stated minimum for ribodepleted or polyA-selected RNA-seq
- The goal is variant calling outside the immunoglobulin loci or T-cell receptor repertoire analysis — CRIS reconstructs Ig heavy-chain V-D-J specifically

## Inputs

- Per-sample BAM files aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed (e.g. EGAD00001004046 cohort BAMs)
- CRIS conda environment.txt or Docker image cris:v1
- Clinical PCR-Sanger IGHV mutation classification table for the cohort (for comparison)

## Outputs

- Per-sample <sample>.bam.IgBLAST_out.txt (percent identity and alignment to top germline IGHV gene hits)
- Per-sample <sample>.bam.ig-transcripts.sortedbyTPM.txt (Ig-transcript IDs sorted by TPM)
- Per-sample <sample>.bam.ig-transcripts.sortedbyTPM.fasta (Ig-transcript FASTA sorted by TPM)
- Cohort-level per-sample table (e.g. EGAD00001004046_ighv_calls.tsv) with dominant-clone IGHV gene, percent identity, mCLL/uCLL calls, and clinical agreement

## How to apply

For each cohort sample, confirm the BAM is aligned to the hg38 (GRCh38) build with BWA, coordinate-sorted and indexed (verify with sambamba). Install CRIS dependencies via conda (conda create --name cris_env --file environment.txt) so picard, trinity, blast, seqkit, sambamba, salmon, igblast and jellyfish are on PATH, or use the Docker image. Run CRIS per sample with 'bash CRIS.sh -inbam <sample>.bam -outdir <dir>' using the default 4 threads and 16G maximum assembly memory; each run needs >=25M reads and >=16GB RAM. Parse each sample's IgBLAST output to extract the top germline IGHV gene hit and percent identity for the dominant (highest-TPM) Ig transcript, then assign mCLL/uCLL from that percent identity. Compile all per-sample calls into a cohort TSV and compare against clinical PCR-Sanger classification; validation requires 100% agreement on both SHM status and dominant-clone IGHV gene.

## Related tools

- **CRIS** (Core pipeline run per cohort sample to reconstruct IGHV sequences and predict SHM status from RNA-seq) — https://github.com/Rashedul/CRIS
- **sambamba** (Verifies cohort BAMs are coordinate-sorted and indexed before CRIS execution)
- **picard** (CRIS dependency used during per-sample processing (v2.20.3))
- **trinity** (CRIS dependency performing de novo assembly of Ig transcripts per sample (v2.1.1))
- **blast** (CRIS dependency aligning assembled contigs to reference Ig sequences (v2.9.0))
- **seqkit** (CRIS dependency for sequence manipulation (v0.12.0))
- **igblast** (Produces per-sample IgBLAST output used to extract top germline IGHV gene hit and percent identity (v1.14.0))
- **salmon** (CRIS dependency quantifying Ig transcript TPM used to select the dominant clone (v0.8.1))
- **Docker** (Alternative containerized execution mode for running CRIS on each cohort BAM)

## Examples

```
bash CRIS.sh -inbam /fullPath/SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- Validation criterion: CRIS mCLL/uCLL SHM status matches clinical PCR-Sanger classification in 100% of cohort samples
- Dominant-clone (highest-TPM) IGHV gene identified by CRIS is identical to the clinical IGHV gene call in every sample
- Each sample's IgBLAST output reports a percent identity to the top germline IGHV gene; uCLL/mCLL assignment derives from the 98% identity convention applied to that value
- Test-run sanity check: supplied SRR1814049_test.bam yields IGHV gene IGHV3-74 with percent identity 94.6
- Per-sample output files (IgBLAST_out.txt, ig-transcripts.sortedbyTPM.txt/.fasta) exist for every cohort sample before compiling the cohort table

## Limitations

- Requires a minimum of 25M sequence reads per sample and at least 16GB RAM and 4 threads per run
- BAMs must be aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed; other alignments are unsupported
- Validated specifically against clinical PCR-Sanger classification in chronic lymphocytic leukemia cohorts; concordance in other disease contexts is not established
- Default 4 threads and 16G assembly memory may need adjustment for very large cohorts

## Evidence

- [abstract] CRIS extracts RNA-seq reads aligned to Ig gene loci, performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences: "CRIS extracts RNA-seq reads aligned to Ig gene loci, performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [intro] bam file must be aligned to hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed: [evidence span withheld — anchor status: not_found]
- [methods] installing dependencies using conda: conda create --name cris_env --file environment.txt: "# installing dependencies using conda
conda create --name cris_env --file environment.txt"
- [abstract] The SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent CLL cohorts: "We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
- [readme] A minimum of 16GB RAM and 4 threads are required to run CRIS; bam must be aligned to hg38, coordinate-sorted and indexed: "* A minimum of 16GB RAM and 4 threads are required to run CRIS 
* bam file must be aligned to hg38 (GRCh38) genome build using"
- [readme] bash CRIS.sh -inbam /fullPath/SRR1814049_test.bam -outdir /fullPath/: "bash CRIS.sh -inbam /fullPath/SRR1814049_test.bam -outdir /fullPath/"
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."

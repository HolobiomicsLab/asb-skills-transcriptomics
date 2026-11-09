---
name: somatic-hypermutation-status-classification
description: Use when you have CLL RNA-seq BAMs aligned to hg38 (GRCh38), coordinate-sorted
  and indexed, and need to call IGHV somatic hypermutation status (mCLL vs uCLL) or
  benchmark those calls against Sanger-based clinical labels.
when_to_use_negative:
- Input BAMs are not aligned to hg38 (GRCh38), or are not coordinate-sorted and indexed
  — CRIS requires this specific setup
- The goal is to reconstruct T-cell receptor repertoires or non-IGHV loci; CRIS targets
  the immunoglobulin heavy chain variable region
- Libraries have fewer than 25M reads per sample or are neither ribodepleted nor polyA-selected
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CRIS
  role: End-to-end pipeline that extracts Ig-locus reads from the BAM, assembles Ig
    transcripts, and predicts IGHV hypermutation status from RNA-seq
  repo: https://github.com/Rashedul/CRIS
- name: trinity v2.1.1
  role: De novo assembly of immunoglobulin transcripts extracted from the RNA-seq
    BAM
- name: blast v2.9.0
  role: Aligns assembled Ig contigs to reference Ig sequences to enumerate and classify
    SHMs in the IGHV gene
- name: igblast v1.14.0
  role: Produces the output file containing percent identity and alignment to the
    top germline IGHV gene hit used for mCLL/uCLL classification
- name: salmon v0.8.1
  role: Estimates TPM so Ig-transcript IDs and FASTA sequences can be sorted by expression
- name: picard v2.20.3
  role: CRIS dependency for BAM processing within the pipeline
- name: seqkit v0.12.0
  role: CRIS dependency for sequence file manipulation of Ig transcripts
- name: sambamba v0.7.0
  role: Coordinate-sorting of BAMs to satisfy CRIS input requirements
- name: Docker
  role: Containerized execution of CRIS (build cris:v1 image, run CRIS_docker.sh,
    copy outputs with docker cp)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:c0df8b4d53287ea152a24144a4f737e22e78e04383e52d92d87fc647a88b6391
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - Docker
  - blast
  - blast v2.9.0
  - igblast v1.14.0
  - jellyfish
  - picard
  - picard v2.20.3
  - salmon
  - salmon v0.8.1
  - sambamba v0.7.0
  - seqkit
  - seqkit v0.12.0
  - trinity
  - trinity v2.1.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# somatic-hypermutation-status-classification

## Summary

Classify CLL samples as IGHV-mutated (mCLL) or unmutated (uCLL) by reconstructing full-length immunoglobulin heavy-chain variable region transcripts from RNA-seq with the CRIS pipeline and measuring percent identity to the top germline IGHV hit, enabling concordance checks against clinical PCR-Sanger labels.

## When to use

Use when you have CLL RNA-seq BAMs aligned to hg38 (GRCh38), coordinate-sorted and indexed, and need to call IGHV somatic hypermutation status (mCLL vs uCLL) or benchmark those calls against Sanger-based clinical labels.

## When NOT to use

- Input BAMs are not aligned to hg38 (GRCh38), or are not coordinate-sorted and indexed — CRIS requires this specific setup
- The goal is to reconstruct T-cell receptor repertoires or non-IGHV loci; CRIS targets the immunoglobulin heavy chain variable region
- Libraries have fewer than 25M reads per sample or are neither ribodepleted nor polyA-selected

## Inputs

- BAM file(s) aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed
- Ribodepleted or polyA-selected RNA-seq with >= 25M reads per sample
- Matched clinical PCR-Sanger IGHV hypermutation labels (for concordance evaluation)

## Outputs

- Per-sample IGHV gene / percent identity pair (e.g. IGHV3-74, 94.6%)
- IgBLAST output file (e.g. SRR1814049_test.bam.IgBLAST_out.txt) with percent identity and alignment to top germline IGHV hit
- Ig-transcript IDs and FASTA sequences sorted by TPM (e.g. .ig-transcripts.sortedbyTPM.txt / .fasta)
- Classification table of mCLL vs uCLL calls per sample with concordance, accuracy, sensitivity and specificity vs Sanger labels

## How to apply

Run CRIS (via the cris_env conda environment with `bash CRIS.sh`, or the Docker image) on each input BAM, using the defaults of 4 threads (-threads 4) and 16G assembly memory (-memory 16G); CRIS extracts reads aligned to Ig gene loci, de novo assembles Ig transcripts (Trinity v2.1.1), and aligns contigs to reference Ig sequences (BLAST v2.9.0) to enumerate SHMs. Parse each sample's IgBLAST output file to extract the top germline IGHV gene hit and its percent identity, using the test-run benchmark (SRR1814049: IGHV3-74, 94.6% identity) to validate the installation. Classify mCLL vs uCLL from the percent identity to the top germline IGHV hit and record the gene/identity pair per sample. Compile a per-sample classification table sorted by sample ID, compare against matched clinical PCR-Sanger labels, and compute concordance, accuracy, sensitivity and specificity, flagging discordant samples. Only ribodepleted or polyA-selected RNA-seq libraries with a minimum of 25M reads per sample are appropriate input.

## Related tools

- **CRIS** (End-to-end pipeline that extracts Ig-locus reads from the BAM, assembles Ig transcripts, and predicts IGHV hypermutation status from RNA-seq) — https://github.com/Rashedul/CRIS
- **trinity v2.1.1** (De novo assembly of immunoglobulin transcripts extracted from the RNA-seq BAM)
- **blast v2.9.0** (Aligns assembled Ig contigs to reference Ig sequences to enumerate and classify SHMs in the IGHV gene)
- **igblast v1.14.0** (Produces the output file containing percent identity and alignment to the top germline IGHV gene hit used for mCLL/uCLL classification)
- **salmon v0.8.1** (Estimates TPM so Ig-transcript IDs and FASTA sequences can be sorted by expression)
- **picard v2.20.3** (CRIS dependency for BAM processing within the pipeline)
- **seqkit v0.12.0** (CRIS dependency for sequence file manipulation of Ig transcripts)
- **sambamba v0.7.0** (Coordinate-sorting of BAMs to satisfy CRIS input requirements)
- **Docker** (Containerized execution of CRIS (build cris:v1 image, run CRIS_docker.sh, copy outputs with docker cp))

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- Test run on the supplied SRR1814049_test.bam yields IGHV3-74 with percent identity 94.6 — deviation indicates a broken install or input issue
- Each sample's IgBLAST_out.txt exists and contains a percent identity and alignment to a top germline IGHV gene hit
- Per-sample classification table is complete for all 51 samples, one gene/identity pair each, sorted by sample ID
- Concordance vs clinical PCR-Sanger labels reaches the published benchmark: 50/51 identical calls (98.3% accuracy, 100% sensitivity, 97.3% specificity), with DFCI-5121 as the single expected discordant case (Sanger mCLL vs CRIS uCLL)
- Ig-transcript FASTA outputs are correctly ordered by TPM expression values

## Limitations

- Input BAMs must be aligned to hg38 (GRCh38) using BWA, coordinate-sorted and indexed; other builds will fail
- Requires at least 16GB RAM and 4 threads; minimum of 25M sequence reads per sample
- Validated specifically in chronic lymphocytic leukemia cohorts; one cohort case (DFCI-5121) was discordant between Sanger mCLL and CRIS uCLL
- No discussion-section content was found in the provided document, so extended caveats from the authors are unavailable

## Evidence

- [intro] CRIS reconstructs the immunoglobulin heavy chain variable region IGHV gene, enumerates single nucleotide variants and predicts hypermutation status from RNA-seq datasets: "reconstructs the immunoglobulin heavy chain variable region IGHV gene, enumerates single nucleotide variants and predicts hypermutation status from RNA-seq datasets"
- [intro] CRIS validated against clinical PCR-Sanger classification in CLL: "CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia"
- [abstract] CRIS aligns contigs to reference Ig sequences to enumerate and classify SHMs in the IGHV gene sequence: "aligns the resulting contigs to reference Ig sequences to enumerate and classify SHMs in the IGHV gene sequence"
- [readme] README: BAM input requirements: "bam file must be aligned to hg38 (GRCh38) genome build using [BWA](http://bio-bwa.sourceforge.net/), coordinate-sorted and indexed"
- [readme] README: expected test-run output: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [readme] README: IgBLAST output description: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."

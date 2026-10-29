---
name: bcr-repertoire-inference-from-rnaseq
description: Use when bulk RNA-seq reads (ribo-depleted or polyA, ≥25M reads/sample)
  from B-cell/CLL samples must be converted into full-length IGHV gene reconstructions
  with SHM/hypermutation (IGHV-mutated vs unmutated) classification.
when_to_use_negative:
- Input BAM is aligned to a build other than hg38 (GRCh38), or is not coordinate-sorted
  and indexed — CRIS explicitly requires this configuration
- Samples have fewer than 25M RNA-seq reads, or data are not RNA-seq (e.g. targeted
  amplicon/PCR-Sanger BCR sequencing already provides full-length IGHV)
- Only partial IGHV coverage or repertoire-level clonotype counting is needed — tools
  like TRUST/MiXCR suffice, whereas CRIS targets complete IGHV gene reconstruction
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3170
tools:
- name: CRIS
  role: 'primary pipeline: extracts Ig-locus reads, assembles Ig transcripts, aligns
    contigs to Ig references to reconstruct the full IGHV gene and classify SHMs'
  repo: github.com/Rashedul/CRIS
- name: Docker
  role: containerization of the CRIS pipeline (cris:v1 image) for reproducible execution
- name: BWA
  role: aligns RNA-seq reads to hg38 (GRCh38) to produce the required coordinate-sorted
    input BAM
- name: SAMBAMBA
  role: coordinate-sorts the alignment BAM (SAMBAMBA sort) prior to CRIS input
- name: trinity v2.1.1
  role: de novo assembly of Ig transcripts extracted from the Ig gene loci
- name: blast v2.9.0
  role: alignment of assembled contigs to reference Ig sequences
- name: picard v2.20.3
  role: BAM processing dependency of the CRIS pipeline
- name: seqkit v0.12.0
  role: sequence manipulation dependency of the CRIS pipeline
- name: V'DJer
  role: comparison tool in the IGHV-reconstruction benchmark; assembles only ~75%
    of IGHV
- name: TRUST v3.0.3
  role: comparison tool in the benchmark; assembles only ~41.5% of IGHV
- name: MiXCR v3.0.3
  role: comparison tool in the benchmark; assembles <10% of IGHV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:f379538c1d9cc3ae91166b269d081e5ff9a78f06e3172a3436f0ad98e21da01d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - CRIS
  - Docker
  - MiXCR
  - MiXCR v3.0.3
  - SAMBAMBA
  - TRUST
  - TRUST v3.0.3
  - V'DJer
  - blast v2.9.0
  - picard v2.20.3
  - seqkit v0.12.0
  - trinity v2.1.1
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# bcr-repertoire-inference-from-rnaseq

## Summary

Reconstruct the immunoglobulin heavy chain variable (IGHV) region from RNA-seq BAMs using CRIS, which extracts Ig-locus reads, de novo assembles Ig transcripts, and aligns contigs to germline references via IgBLAST to enumerate SHMs and predict hypermutation status, benchmarked against V'DJer, TRUST, and MiXCR.

## When to use

Use when bulk RNA-seq reads (ribo-depleted or polyA, ≥25M reads/sample) from B-cell/CLL samples must be converted into full-length IGHV gene reconstructions with SHM/hypermutation (IGHV-mutated vs unmutated) classification.

## When NOT to use

- Input BAM is aligned to a build other than hg38 (GRCh38), or is not coordinate-sorted and indexed — CRIS explicitly requires this configuration
- Samples have fewer than 25M RNA-seq reads, or data are not RNA-seq (e.g. targeted amplicon/PCR-Sanger BCR sequencing already provides full-length IGHV)
- Only partial IGHV coverage or repertoire-level clonotype counting is needed — tools like TRUST/MiXCR suffice, whereas CRIS targets complete IGHV gene reconstruction

## Inputs

- BAM file aligned to hg38 (GRCh38), coordinate-sorted and indexed (e.g. from BWA + SAMBAMBA sort)
- Bulk RNA-seq data: ribo-depleted or polyA selected, minimum 25M reads per sample
- Docker image cris:v1 (or conda environment.txt with picard v2.20.3, trinity v2.1.1, blast v2.9.0, seqkit v0.12.0, sambamba v0.7.0, salmon v0.8.1, igblast v1.14.0, jellyfish v2.2.10)
- Supplied test BAM SRR1814049_test.bam for install validation

## Outputs

- <sample>.IgBLAST_out.txt — percent identity and alignment between Ig-transcript and top germline IGHV gene hits
- <sample>.ig-transcripts.sortedbyTPM.txt — Ig-transcript sequence IDs sorted by expression (TPM)
- <sample>.ig-transcripts.sortedbyTPM.fasta — Ig-transcript FASTA sequences sorted by TPM
- Reconstructed full IGHV gene sequence with percent identity, SNVs, and predicted SHM (hypermutation) status
- Benchmark table (percent IGHV assembled and runtime per tool, e.g. benchmark_table.tsv)

## How to apply

First align reads to the hg38 (GRCh38) genome build with BWA and produce a coordinate-sorted, indexed BAM (SAMBAMBA sort may be used). Build the CRIS Docker image (cris:v1) or install dependencies (picard v2.20.3, trinity v2.1.1, blast v2.9.0, seqkit v0.12.0, sambamba v0.7.0, salmon v0.8.1, igblast v1.14.0, jellyfish v2.2.10) via the provided conda environment.txt; a minimum of 4 threads and 16G RAM is required. Run CRIS on the input BAM, optionally validating the install with the supplied SRR1814049_test.bam (expected: IGHV3-74, 94.6% identity). CRIS extracts reads aligned to Ig gene loci, de novo assembles Ig transcripts (Trinity), and aligns contigs to reference Ig sequences to enumerate and classify SHMs across the entire IGHV gene, producing IgBLAST output plus TPM-sorted transcript FASTA files. For benchmarking, run V'DJer, TRUST (v3.0.3), and MiXCR (v3.0.3) on identical BAMs at the same thread count and tabulate percent IGHV assembled and wall-clock runtime.

## Related tools

- **CRIS** (primary pipeline: extracts Ig-locus reads, assembles Ig transcripts, aligns contigs to Ig references to reconstruct the full IGHV gene and classify SHMs) — github.com/Rashedul/CRIS
- **Docker** (containerization of the CRIS pipeline (cris:v1 image) for reproducible execution)
- **BWA** (aligns RNA-seq reads to hg38 (GRCh38) to produce the required coordinate-sorted input BAM)
- **SAMBAMBA** (coordinate-sorts the alignment BAM (SAMBAMBA sort) prior to CRIS input)
- **trinity v2.1.1** (de novo assembly of Ig transcripts extracted from the Ig gene loci)
- **blast v2.9.0** (alignment of assembled contigs to reference Ig sequences)
- **picard v2.20.3** (BAM processing dependency of the CRIS pipeline)
- **seqkit v0.12.0** (sequence manipulation dependency of the CRIS pipeline)
- **V'DJer** (comparison tool in the IGHV-reconstruction benchmark; assembles only ~75% of IGHV)
- **TRUST v3.0.3** (comparison tool in the benchmark; assembles only ~41.5% of IGHV)
- **MiXCR v3.0.3** (comparison tool in the benchmark; assembles <10% of IGHV)

## Examples

```
sudo docker run --name cris_analysis -v /fullPath/SRR1814049_test.bam:/cris/SRR1814049_test.bam -t cris:v1 bash CRIS_docker.sh SRR1814049_test.bam threads=4 memory=16G
```

## Evaluation signals

- Test-run sanity check on SRR1814049_test.bam: expected output is IGHV gene IGHV3-74 with percent identity 94.6
- Output files <sample>.IgBLAST_out.txt, .ig-transcripts.sortedbyTPM.txt, and .ig-transcripts.sortedbyTPM.fasta are present and non-empty after each run
- Percent IGHV gene reconstructed per sample approaches ~99.74% (as reported for 16 GSE66228 CLL samples), far above V'DJer (75.44%), TRUST (41.5%), and MiXCR (<10%)
- Runtime at 16 threads matches reported scale: CRIS ~3.07 min vs V'DJer 3.50 min and TRUST ~36 min
- SHM (mutated vs unmutated) calls are concordant with clinical PCR-Sanger hypermutation classification when validation data are available

## Limitations

- Requires substantial compute: minimum 16GB RAM and 4 threads; memory and thread parameters must be passed to the Docker run
- Input BAM must be aligned specifically to hg38 (GRCh38) with BWA and be coordinate-sorted and indexed — other builds or aligners are not supported per the README
- Requires at least 25M reads per sample; validated only in the CLL (chronic lymphocytic leukemia) context against PCR-Sanger classification
- Existing tools (V'DJer, TRUST, MiXCR) generate only partial IGHV sequences, so cross-tool percent-IghV comparisons reflect this structural limitation

## Evidence

- [abstract] we describe a bioinformatic pipeline, CRIS (Complete Reconstruction of Immunoglobulin IGHV-D-J Sequences) that uses RNA sequencing (RNA-seq) datasets: "we describe a bioinformatic pipeline, CRIS (Complete Reconstruction of Immunoglobulin IGHV-D-J Sequences) that uses RNA sequencing (RNA-seq) datasets"
- [intro] bam file must be aligned to hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed: [evidence span withheld — anchor status: not_found]
- [abstract] extracts RNA-seq reads aligned to Ig gene loci: "CRIS extracts RNA-seq reads aligned to Ig gene loci, performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences to enumerate and classify SHMs in the IGHV gene"
- [methods] sudo docker run --name cris_analysis ... threads=4 memory=16G: "sudo docker run --name cris_analysis -v /fullPath/SRR1814049_test.bam:/cris/SRR1814049_test.bam -t cris:v1 bash CRIS_docker.sh SRR1814049_test.bam threads=4 memory=16G"
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."
- [readme] expected mutational status for test run: IGHV gene: IGHV3-74, Percent identity: 94.6: "# expected mutational status for test run 

- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [results] On the 16 GSE66228 CLL RNA-seq samples, CRIS reconstructed on average 99.74% of the IGHV gene, whereas V'DJer assembled only 75.44% and TRUST only 41.5%: [evidence span withheld — anchor status: not_found]

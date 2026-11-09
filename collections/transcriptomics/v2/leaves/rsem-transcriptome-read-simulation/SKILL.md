---
name: rsem-transcriptome-read-simulation
description: Use when a controlled benchmark of transcriptome assemblers is needed
  and a ground-truth transcriptome (e.g., hg38 NCBI RefSeq) must be paired with realistic
  simulated reads.
when_to_use_negative:
- Data are long-read RNA-seq (PacBio or ONT) — this simulation workflow was built
  for 100-bp paired-end short reads
- The benchmark requires measuring performance on real biological variation rather
  than a ground-truth-controlled simulation from RefSeq
- The assembly must be annotation-guided or use known transcript annotations, which
  the current Tiglon version does not support
tools:
- name: RSEM
  role: Generates the simulated paired-end reads with parameters learned from the
    real sample SRR7536920
- name: Hisat2
  role: Maps simulated reads to the reference genome to produce one of the alignment
    sets for assembly
  repo: https://github.com/DaehwanKimLab/hisat2
- name: STAR
  role: Maps simulated reads to the reference genome to produce the second alignment
    set for assembly
  repo: https://github.com/alexdobin/STAR
- name: Tiglon
  role: Downstream assembler consuming the merged Hisat2/STAR BAM alignments of the
    simulated reads
  repo: https://github.com/yutingsdu/Tiglon-v.1.1
- name: Cuffcompare
  role: Evaluates assembled transcripts against the RefSeq transcriptome to count
    correct transcripts, precision, and F-score
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:8e3ab032de6a05d8fc9e36e5e34a29276ab9dd5b810baaf2f1f2b8f1b1ef98a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaehwanKimLab/hisat2
  tools:
  - Cuffcompare
  - Hisat2
  - RSEM
  - STAR
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# rsem-transcriptome-read-simulation

## Summary

Generate a paired-end RNA-seq simulation with RSEM in which the simulation-model parameters are learned from a real human RNA-seq sample (SRR7536920), producing ~52 million 100-bp paired-end reads from the hg38 NCBI RefSeq transcriptome for assembler benchmarking.

## When to use

Use when a controlled benchmark of transcriptome assemblers is needed and a ground-truth transcriptome (e.g., hg38 NCBI RefSeq) must be paired with realistic simulated reads.

## When NOT to use

- Data are long-read RNA-seq (PacBio or ONT) — this simulation workflow was built for 100-bp paired-end short reads
- The benchmark requires measuring performance on real biological variation rather than a ground-truth-controlled simulation from RefSeq
- The assembly must be annotation-guided or use known transcript annotations, which the current Tiglon version does not support

## Inputs

- Real RNA-seq sample used to learn simulation parameters (SRA: SRR7536920)
- Reference transcriptome annotation: NCBI RefSeq for human genome GRCh38/hg38

## Outputs

- Simulated FASTQ of ~52 million 100-bp paired-end reads with known transcript-of-origin
- BAM alignments of the simulated reads against the reference genome from Hisat2 and STAR
- simulated_benchmark.csv with one row per assembler condition (correct transcripts, precision, F-score; 7 rows)

## How to apply

Learn the RSEM (Li and Dewey, 2011) simulation-model parameters from a real RNA-seq sample — here the human dataset with NCBI SRA accession SRR7536920 — so read quality, expression, and fragment distributions match real data. Use the learned model to simulate ~52 million 100-bp paired-end reads from the hg38 (GRCh38) NCBI RefSeq transcriptome, giving each read a known transcript-of-origin. Map the simulated reads to the reference genome with two splice-aware aligners (Hisat2 and STAR) to produce the alignment sets consumed by downstream assemblers (Tiglon, StringTie2, Scallop). Evaluate the resulting assemblies against the RefSeq transcriptome with Cuffcompare, counting correct transcripts, precision, and F-score; the Tiglon paper reports 22,928 correct transcripts, 64.01% precision, and a 0.480 F-score for Tiglon on this simulated dataset.

## Related tools

- **RSEM** (Generates the simulated paired-end reads with parameters learned from the real sample SRR7536920)
- **Hisat2** (Maps simulated reads to the reference genome to produce one of the alignment sets for assembly) — https://github.com/DaehwanKimLab/hisat2
- **STAR** (Maps simulated reads to the reference genome to produce the second alignment set for assembly) — https://github.com/alexdobin/STAR
- **Tiglon** (Downstream assembler consuming the merged Hisat2/STAR BAM alignments of the simulated reads) — https://github.com/yutingsdu/Tiglon-v.1.1
- **Cuffcompare** (Evaluates assembled transcripts against the RefSeq transcriptome to count correct transcripts, precision, and F-score)

## Evaluation signals

- Simulation output contains ~52 million paired-end reads of 100-bp length
- Simulation-model parameters trace back to the real human RNA-seq dataset SRR7536920
- Simulated reads map to the human reference genome with both Hisat2 and STAR, yielding two alignment sets
- Benchmark table contains precision and F-score columns and reproduces Tiglon's reported simulated-dataset results: 22,928 correct transcripts, 64.01% precision, 0.480 F-score
- Cuffcompare evaluation resolves to 7 assembler-condition rows (Tiglon/StringTie2/Scallop under Hisat2, STAR, and merged alignments, plus StringTie2-Merge)

## Limitations

- Simulation reflects the transcript composition of the hg38 NCBI RefSeq annotation and the parameter source sample SRR7536920; conclusions may not transfer to other annotations or expression profiles
- Current Tiglon version accepts only BAM files generated by Hisat(2) and STAR, so simulated reads must be mapped with one of these aligners
- Tiglon is not compatible with long-read RNA-seq datasets (PacBio, ONT), so this short-read simulation cannot support long-read assembler benchmarks

## Evidence

- [results] RSEM simulated dataset (~52 million 100-bp paired-end reads): "RSEM (Li and Dewey, 2011) to generate a simulated dataset that contained ∼52 million 100-bp length paired-end reads"
- [results] Simulation parameters learned from a real sample: "The parameters of the simulation model were learned from the real human RNA-seq dataset with the NCBI SRA accession code of SRR7536920"
- [readme] Downstream assembler accepts only Hisat(2)/STAR BAMs: "The current version of Tiglon can accepts BAM files generated by Hisat(2) and Star."

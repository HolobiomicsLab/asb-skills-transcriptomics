---
name: fusion-gene-detection-benchmarking
description: Use when comparing fusion gene caller sensitivity across simulated or
  real paired-end RNA-seq data, especially when testing detection limits under progressive
  subsampling or shallow sequencing depth.
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  only supports paired-end RNA-seq
- The fusions of interest do not include the user-anchored target gene — Anchored-fusion
  will not report them
- A de novo (unanchored) survey of all fusions is required — Anchored-fusion is targeted
  by design
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: Anchored-fusion
  role: Targeted fusion detector being benchmarked; anchors a gene of interest and
    recovers non-unique short-read matches with HVLD false-positive filtering
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion
  role: De novo bulk RNA-seq competing method (v1.12.0)
  repo: https://github.com/STAR-Fusion
- name: FusionInspector
  role: De novo bulk RNA-seq competing method (v2.8.0)
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher
  role: De novo bulk RNA-seq competing method (v1.30)
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba
  role: De novo bulk RNA-seq competing method (v2.4.0)
  repo: https://github.com/suhrig/arriba
- name: BWA-MEM
  role: Aligns paired-end reads to the anchored gene reference sequence
- name: BLAT
  role: Local alignment of candidate chimeric reads to predict the precise fusion
    location
- name: samtools
  role: Read alignment handling and subsampling of read pairs to generate depth series
- name: bedtools
  role: Interval operations during candidate processing
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:0c985e85302117bfe3ce0abc736dffa22a409c17acd03cac0670bbad8dee496a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - BLAT
  - BWA
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - PyTorch
  - STAR-Fusion
  - bedtools
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# fusion-gene-detection-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark targeted fusion gene detection (Anchored-fusion) against de novo callers (STAR-Fusion, FusionInspector, FusionCatcher, Arriba) on simulated, bulk RNA-seq, and subsampled single-cell/bulk data to quantify sensitivity differences at low sequencing depth.

## When to use

Use when comparing fusion gene caller sensitivity across simulated or real paired-end RNA-seq data, especially when testing detection limits under progressive subsampling or shallow sequencing depth.

## When NOT to use

- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion only supports paired-end RNA-seq
- The fusions of interest do not include the user-anchored target gene — Anchored-fusion will not report them
- A de novo (unanchored) survey of all fusions is required — Anchored-fusion is targeted by design

## Inputs

- Paired-end FASTQ files (e.g., CCLE bRNA-seq from SRA:SRP186687 for K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12)
- Target gene transcript reference fasta (downloaded via NCBI Gene)
- Genome fasta (hg38 or hg19) and compatible gene annotation GTF
- Simulated fusion transcript reads (e.g., BCR-ABL1, CIC-DUX4, DUX4-IGH combined with GM12878 RNA-seq)
- Positive training samples (PCAWG fusion genes, Synapse:syn10003873) and homologous gene file for HVLD filter training

## Outputs

- Tool-by-depth fusion detection matrix (caller × subsampling depth, per fusion event)
- Per-tool spanning and split read counts supporting each fusion event
- Minimum read coverage at which each tool detects each fusion (detection-limit table, e.g., simulated_fusion_benchmark_results.csv)

## How to apply

Prepare a target gene transcript fasta (via NCBI Gene) plus a compatible hg38/hg19 genome fasta and GTF annotation, then run Anchored_Fusion.py which anchors the gene of interest, aligns reads with BWA-MEM, performs BLAT local alignment of candidate chimeric reads, and filters false positives with the HVLD deep learning module. Run STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 on the same paired-end FASTQ inputs. For depth-sensitivity testing, generate 8 sequencing depths by 7 successive rounds of random halving of read pairs (e.g., with samtools view subsampling), ending below 1 million reads, and run all callers at each depth. Tally spanning and split reads supporting each fusion event and compile a tool-by-depth detection matrix. Judge correctness by verifying the expected pattern: Anchored-fusion detects the target fusions (e.g., BCR-ABL1, CIC-DUX4) at the lowest coverage, and in CCLE bRNA-seq (SRA:SRP186687) it detects fusions after 7-fold subsampling to <1M reads where all four competing tools fail.

## Related tools

- **Anchored-fusion** (Targeted fusion detector being benchmarked; anchors a gene of interest and recovers non-unique short-read matches with HVLD false-positive filtering) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion** (De novo bulk RNA-seq competing method (v1.12.0)) — https://github.com/STAR-Fusion
- **FusionInspector** (De novo bulk RNA-seq competing method (v2.8.0)) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher** (De novo bulk RNA-seq competing method (v1.30)) — https://github.com/ndaniel/fusioncatcher
- **Arriba** (De novo bulk RNA-seq competing method (v2.4.0)) — https://github.com/suhrig/arriba
- **BWA-MEM** (Aligns paired-end reads to the anchored gene reference sequence)
- **BLAT** (Local alignment of candidate chimeric reads to predict the precise fusion location)
- **samtools** (Read alignment handling and subsampling of read pairs to generate depth series)
- **bedtools** (Interval operations during candidate processing)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Anchored-fusion detects BCR-ABL1 and CIC-DUX4 at the lowest coverage of all tested tools in the simulated benchmark (Figure 3A/3B)
- DUX4-IGH detection limit for Anchored-fusion equals one-fourth of FusionCatcher's coverage limit
- In CCLE bRNA-seq (SRA:SRP186687), Anchored-fusion detects the known fusion after all 7 subsampling rounds (<1M read depth) where STAR-Fusion, FusionInspector, FusionCatcher, and Arriba all fail (Figure 4 pattern)
- Spanning and split read counts are tallied for every called fusion event at every depth, yielding a complete caller-by-depth matrix
- Genome fasta and GTF annotation versions are compatible (both hg38 or both hg19)

## Limitations

- Fusion genes that do not include the user-provided target gene will not be detected
- Anchored-fusion requires paired-end RNA-seq and cannot be used on single-end RNA-seq, whole-genome, or whole-exome data
- Users must supply the reference sequence of their target gene
- For small input datasets, retraining the HVLD filter is unreliable; use the default model parameters provided on GitHub
- Using the pre-trained (non-retrained) filter may cause deviations in results since it was not trained on the user's input data

## Evidence

- [results] Anchored-fusion successfully detected the corresponding fusion genes even when the datasets were subsampled 7 times with less than 1 M sequence depth, which none of the other competing tools achieved.: [evidence span withheld — anchor status: not_found]
- [results] Among all methods tested on simulated fusion transcripts, Anchored-fusion demonstrated the best detection capability: [evidence span withheld — anchor status: not_found]

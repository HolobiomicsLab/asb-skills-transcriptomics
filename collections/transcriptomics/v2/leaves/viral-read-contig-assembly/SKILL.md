---
name: viral-read-contig-assembly
description: Use when unmapped RNA-seq reads (48 bp fragments in *_unmapped.fastq)
  have been scored by the viRNAtrap TensorFlow model and predicted viral reads must
  be extended into contigs for downstream viral identification.
when_to_use_negative:
- Input reads are still aligned to the human genome — viRNAtrap is applied only to
  unaligned RNA reads (not mapped to hg19 or phiX) to reduce running time
- Sequences come from organisms other than human/viral — model predictions for such
  sequences are not defined and other-organism models are future work
- You expect to overwrite or append to existing output files — virnatrap-predict refuses
  to overwrite pre-existing outputs
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0781
tools:
- name: viRNAtrap
  role: Alignment-free pipeline that scores unmapped reads with the deep learning
    model and assembles predicted viral reads into contigs (run via virnatrap-predict)
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow/Keras
  role: Precomputed 48 bp read-classification model that supplies seed scores (>0.7)
    and per-extension average scores (<0.5 termination)
  repo: keras-team/keras
- name: blastn
  role: Post-processing comparison of assembled viral contigs against three curated
    viral databases (word size 15, E-value 0.01) to assign viral taxonomy
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:81e1af8ac05ac0c300b5bb752bb33e1f55756a8b77986beedff7db56dfa17c20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - NumPy
  - TensorFlow/Keras
  - blastn
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# viral-read-contig-assembly

## Summary

Assemble deep-learning-predicted viral reads from unmapped RNA-seq FASTQ into longer viral contigs using viRNAtrap's alignment-free, overlap-extension assembler, so contigs can be assigned taxonomy via blastn against curated viral databases.

## When to use

Use when unmapped RNA-seq reads (48 bp fragments in *_unmapped.fastq) have been scored by the viRNAtrap TensorFlow model and predicted viral reads must be extended into contigs for downstream viral identification.

## When NOT to use

- Input reads are still aligned to the human genome — viRNAtrap is applied only to unaligned RNA reads (not mapped to hg19 or phiX) to reduce running time
- Sequences come from organisms other than human/viral — model predictions for such sequences are not defined and other-organism models are future work
- You expect to overwrite or append to existing output files — virnatrap-predict refuses to overwrite pre-existing outputs

## Inputs

- Unmapped RNA-seq reads as FASTQ files named *_unmapped.fastq (paired-end files concatenated per sample, one file per sample)
- Pretrained 48 bp Keras/TensorFlow model file model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- Compiled C assembly library src/assemble_read_c.so (for fast mode)

## Outputs

- Per-sample FASTA-format contig files with .txt suffix in the output directory (e.g. output_contigs/example_contigs.txt)
- Contig FASTA headers embedding the model score in brackets, e.g. >contig2[0.8009399]
- Empty output file for input files yielding zero predicted viral contigs

## How to apply

Score each 48 bp read in the unmapped FASTQ with the pretrained Keras/TensorFlow model (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5) and select reads with score > 0.7 as assembly seeds. Grow each contig alignment-free by searching the read pool for a read sharing an exact 24 bp k-mer with the current contig end and picking the maximal-extension read to extend leftward and rightward; at each step recompute model scores over the extended region and stop extending in a direction once the average score falls below 0.5. Concatenate left and right extensions into the final contig and emit one FASTA file per input sample with a .txt suffix, embedding the score in the header in brackets (e.g. >contig2[0.8009399]). For speed, compile the C assembly routine (gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c) and run in fast mode with --fastmode 1, optionally parallel with --multi_proc 1, verifying the C path reproduces the Python contigs. Never overwrite pre-existing output files — delete them before rerunning.

## Related tools

- **viRNAtrap** (Alignment-free pipeline that scores unmapped reads with the deep learning model and assembles predicted viral reads into contigs (run via virnatrap-predict)) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow/Keras** (Precomputed 48 bp read-classification model that supplies seed scores (>0.7) and per-extension average scores (<0.5 termination)) — keras-team/keras
- **blastn** (Post-processing comparison of assembled viral contigs against three curated viral databases (word size 15, E-value 0.01) to assign viral taxonomy)

## Examples

```
gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c; virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1
```

## Evaluation signals

- Diff generated output_contigs/example_contigs.txt against expected_output/output_py.txt — correct if no differences, or only small differences in less-significant digits of bracketed scores (e.g. >contig2[0.8009399] vs >contig2[0.8009398])
- In fast mode, output matches expected_output/output_c.txt (C implementation reproduces the same contigs as the native Python loop)
- Each contig extension step uses a read sharing an exact 24 bp k-mer with the current contig end, and extension halts per direction when average model score drops below 0.5
- One-to-one correspondence between input FASTQ files and output files; inputs with zero predicted contigs still produce an (empty) output file; no pre-existing output file is overwritten
- At TCGA scale (task_005), assembled contigs blastn-matched at word size 15 / E-value 0.01 give per-sample virus calls with TPR > 95% against TCGA clinical annotations for HR-ɑHPV (288 CESC / 61 HNSC), HBV (85 LIHC), HCV (13 LIHC)

## Limitations

- Sensitivity is only assured when more than five viral reads are present in the sample
- It is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures
- Model predictions for sequences from organisms other than human/viral are not defined; models for other organisms are left to future work
- viRNAtrap treats each distinct input file as a distinct sample, so paired-end files must be concatenated per sample before running
- Rerunning with the same input/output directories fails unless previous output files are removed first

## Evidence

- [readme] fast mode rationale: "The fast mode calls a C library to assemble the viral contigs from the model-predicted viral reads."
- [readme] compile command: "gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c"
- [readme] output format: "The output files are in FASTA format but have the suffix .txt because experience has shown that Mac user prefer the suffix .txt"
- [readme] validation by diff: "The installation is correct if the above diff command retruns either no differences or small differences in the less significant digits for the scores in brackets"
- [readme] no overwrite behavior: "virrnatrap-predict will not overwrite output files that already exist."
- [intro] unaligned-read input: "The computational framework, named viRNAtrap, was then applied to unaligned RNA reads (to reduce the running time of viRNAtrap), to detect viral reads and assemble predicted viral contigs"

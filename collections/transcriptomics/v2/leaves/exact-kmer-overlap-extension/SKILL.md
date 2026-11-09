---
name: exact-kmer-overlap-extension
description: Use when assembling viral contigs from unmapped RNAseq FASTQ reads without
  an aligner, using model-predicted viral reads as seeds and exact k-mer overlap extension
  to grow contigs.
when_to_use_negative:
- Reads are already aligned to the human genome (hg19) or to phiX phage — viRNAtrap
  is applied only to unaligned reads to reduce running time.
- The analysis requires full-fledged genome assembly with mismatch/gap tolerance —
  this greedy method only extends through exact 24bp k-mer overlaps.
- An output file for the same input already exists — virnatrap-predict will not overwrite
  existing output files and must have them removed first.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3474
tools:
- name: viRNAtrap
  role: Alignment-free pipeline that scores unmapped 48bp reads with its deep learning
    model and assembles predicted viral reads into contigs by exact 24bp k-mer overlap
    extension
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend for the precomputed model that scores each 48bp read; scores drive
    seed selection (>0.7) and extension termination (average <0.5)
- name: keras
  role: Model framework in which the precomputed 48bp-read classifier is stored and
    loaded for scoring during assembly
  repo: keras-team/keras
- name: ctypes / assemble_read_c.c
  role: 'Optional fast mode: the C assembly routine compiled as a shared library and
    called via ctypes instead of the native Python loop'
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:1ac3c3c254c3916aff146e0a04a079972ed3019773ea1cc38a19b9619aa94c30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - TensorFlow
  - ctypes / assemble_read_c.c
  - keras
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# exact-kmer-overlap-extension

## Summary

Alignment-free greedy contig assembly used in viRNAtrap: seed on deep-learning-scored viral 48bp reads, then iteratively extend left and right by finding reads sharing an exact 24bp k-mer with the current contig end, guided by model scores.

## When to use

Use when assembling viral contigs from unmapped RNAseq FASTQ reads without an aligner, using model-predicted viral reads as seeds and exact k-mer overlap extension to grow contigs.

## When NOT to use

- Reads are already aligned to the human genome (hg19) or to phiX phage — viRNAtrap is applied only to unaligned reads to reduce running time.
- The analysis requires full-fledged genome assembly with mismatch/gap tolerance — this greedy method only extends through exact 24bp k-mer overlaps.
- An output file for the same input already exists — virnatrap-predict will not overwrite existing output files and must have them removed first.

## Inputs

- Directory of FASTQ files of unmapped RNAseq reads with filenames ending in *_unmapped.fastq
- Precomputed Keras/TensorFlow model file (model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5) for 48bp reads

## Outputs

- FASTA-format contig files with .txt suffix, one output file per input FASTQ, headers embedding the score in brackets (e.g. >contig2[0.8009399])

## How to apply

Score each fixed-length 48bp read in the unmapped FASTQ with the precomputed Keras/TensorFlow model (model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5) and keep reads with score > 0.7 as assembly seeds. For each seed, search the read pool for the read sharing an exact 24bp k-mer with the current contig end and select the maximal-extension read to grow the contig leftward and rightward (alignment-free overlap-extension). At each extension step, recompute model scores over the extended region and stop extending in a direction once the average model score falls below 0.5. Concatenate left and right extensions into the final contig and emit contigs as FASTA records with a .txt suffix, embedding the score in the header in brackets (e.g. >contig2[0.8009399]); do not overwrite pre-existing output files. Optionally enable fast mode by compiling the C routine with 'gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c' and calling it via ctypes, verifying it reproduces the same contigs as the native Python loop.

## Related tools

- **viRNAtrap** (Alignment-free pipeline that scores unmapped 48bp reads with its deep learning model and assembles predicted viral reads into contigs by exact 24bp k-mer overlap extension) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend for the precomputed model that scores each 48bp read; scores drive seed selection (>0.7) and extension termination (average <0.5))
- **keras** (Model framework in which the precomputed 48bp-read classifier is stored and loaded for scoring during assembly) — keras-team/keras
- **ctypes / assemble_read_c.c** (Optional fast mode: the C assembly routine compiled as a shared library and called via ctypes instead of the native Python loop) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1
```

## Evaluation signals

- diff the generated output_contigs/example_contigs.txt against expected_output/output_py.txt; the result is correct if there are no differences, or only small differences in the less significant digits of the bracketed scores (e.g. >contig2[0.8009399] vs >contig2[0.8009398]).
- In fast mode, the C implementation must reproduce the same contigs as the native Python implementation (compare against expected_output/output_c.txt).
- Every output contig header must embed a bracketed model score (e.g. >contig2[0.8009399]).
- There must be a one-to-one correspondence between input FASTQ files and output files; inputs yielding zero predicted viral contigs produce an empty output file.
- All contigs should be extendable only via exact 24bp k-mer overlaps; directions whose average model score falls below 0.5 must be terminated.

## Limitations

- Extension only proceeds through exact 24bp k-mer overlaps, so reads with sequencing errors in the overlap region break contig growth.
- Contig identity depends entirely on the model scores; the sensitivity guarantee is stated only for cases with more than five viral reads present.
- viRNAtrap treats each distinct input file as a distinct sample, so paired reads stored separately must be concatenated into one combined file before assembly.
- Existing output files are never overwritten; rerunning with the same inputs requires manually removing previous output files.

## Evidence

- [results] Seed selection threshold: [evidence span withheld — anchor status: not_found]
- [abstract] Alignment-free extension: "an alignment-free pipeline to identify viral reads and assemble viral contigs"
- [intro] Application to unaligned reads: "The computational framework, named viRNAtrap, was then applied to unaligned RNA reads (to reduce the running time of viRNAtrap), to detect viral reads and assemble predicted viral contigs"
- [intro] Model trained on 48bp fragments: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [readme] Fast mode C compilation: "gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c"
- [readme] Output validation by diff: "diff expected_output/output_py.txt output_contigs/example_contigs.txt"
- [readme] No overwrite behavior: "virrnatrap-predict will not overwrite output files that already exist."
- [readme] Paired reads concatenation: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample."

---
name: novel-splice-site-fragment-indexing
description: 'Use when quantifying reads on novel splicing events (ES, IR, AA, AD,
  AP, IE) without a spliced aligner: you have a genome FASTA, a GTF annotation, and
  a known read length.'
when_to_use_negative:
- Input is already a fortuna index or a pre-built H. sapiens index for the matching
  read length (75/100/150bp) — download it rather than rebuilding
- The annotation is not in GTF format or has not been segmented by bin/processGTF.sh
  — fortuna --index requires the preprocessed .rg.gtf
- Goal is full spliced alignment (e.g. building a genome BAM of spliced reads) — use
  a spliced aligner like STAR instead; fortuna optionally refines with its output
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0102
tools:
- name: fortuna
  role: Segments the GTF (processGTF.sh) and runs the --index command that guesses
    novel transcript fragments and builds the kallisto index
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Index engine built from the fragment set of annotated and guessed novel transcripts
  repo: https://github.com/pachterlab/kallisto
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:98eb28d2799b3b28a8dedd74662c95af2182f819049c22a8fdd1d8e141aa11ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - kallisto
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# novel-splice-site-fragment-indexing

## Summary

Build a kallisto index over sequence fragments of annotated and 'guessed' novel transcripts — combinations of annotated splice sites — so that alignment-free pseudoalignment can count reads supporting novel splicing events.

## When to use

Use when quantifying reads on novel splicing events (ES, IR, AA, AD, AP, IE) without a spliced aligner: you have a genome FASTA, a GTF annotation, and a known read length.

## When NOT to use

- Input is already a fortuna index or a pre-built H. sapiens index for the matching read length (75/100/150bp) — download it rather than rebuilding
- The annotation is not in GTF format or has not been segmented by bin/processGTF.sh — fortuna --index requires the preprocessed .rg.gtf
- Goal is full spliced alignment (e.g. building a genome BAM of spliced reads) — use a spliced aligner like STAR instead; fortuna optionally refines with its output

## Inputs

- Genome sequence in FASTA format
- Transcript annotation in GTF format (e.g. GENCODE human release 42, GRCh38.p13)
- Target read length (-rl, set to the length of the longest read in the experiment)

## Outputs

- Segmented annotation (.rg.gtf) with NodeId subexon segments
- kallisto index over annotated and guessed novel transcript fragments

## How to apply

Preprocess the GENCODE GTF (e.g. human release 42, GRCh38.p13) with fortuna's bin/processGTF.sh to compute minimal disjoint subexon segments bounded by splice sites, producing the .rg.gtf segmented annotation. Then run 'fortuna --index' with the genome FASTA, the preprocessed GTF, and a target read length (-rl set to the longest read); fortuna 'guesses' novel transcript fragments from annotated splice sites and builds the kallisto index from the fragment set (steps B and C of the fortuna pipeline). Pre-built indexes for H. sapiens at read lengths 75bp, 100bp, and 150bp can be downloaded instead of building from scratch. Rationale: by enumerating novel combinations of annotated splice sites before quantification, downstream pseudoalignment can assign reads to fragments containing novel junctions, yielding signature counts and event annotations without alignment.

## Related tools

- **fortuna** (Segments the GTF (processGTF.sh) and runs the --index command that guesses novel transcript fragments and builds the kallisto index) — https://github.com/canzarlab/fortuna
- **kallisto** (Index engine built from the fragment set of annotated and guessed novel transcripts) — https://github.com/pachterlab/kallisto

## Examples

```
bash bin/processGTF.sh /path/to/the/annotation.gtf && ./fortuna --index genome.fa sample.rg.gtf -rl 100
```

## Evaluation signals

- The .rg.gtf output exists and subexon NodeId fields match the input GTF coordinates used later in cnt.tsv and alt.tsv
- On the shipped example dataset (sample 4, patient id 11352.p1, ~240,000 STAT1 reads), fortuna maps over 99% of reads and finds slightly over 1000 reads supporting the novel exon skipping at chr2:190984393-190986854
- Pre-built index read length (75/100/150bp) matches or exceeds the -rl setting used at indexing time and the actual experimental read length
- The --index step completes without errors given C++11 and CMake >=2.8.12 dependencies, with bundled kallisto and htslib

## Limitations

- Novel fragments are 'guessed' from annotated splice sites only; junctions involving entirely unannotated splice sites are missed unless incorporated via 'fortuna --refine' from a spliced aligner's BAM
- The index depends on a single target read length; reads longer than -rl are not covered, and pre-built indexes are only provided for H. sapiens at 75bp, 100bp, and 150bp
- Building from source requires C++11, CMake >=2.8.12, zlib, and autoconf 2.69 (macOS users may need 'brew install autoconf@2.69')

## Evidence

- [readme] fortuna starts (A) by "guessing" novel transcripts based on annotated splice sites. It then (B) creates a set of sequence fragments of annotated and guessed novel transcripts: "fortuna starts (A) by "guessing" novel transcripts based on annotated splice sites. It then (B) creates a set of sequence fragments"
- [readme] From this set of fragments we build a kallisto [2] index (C) and use it to efficiently pseudoalign reads to fragments (D): "From this set of fragments we build a kallisto [2] index (C) and use it to efficiently pseudoalign reads to fragments (D)"
- [readme] A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [methods] Steps B and C are executed using fortuna's "--index" command. Given a genome sequence in FASTA format, a preprocessed transcript annotation (GTF), and a target read length: "Steps B and C are executed using fortuna's "--index" command. Given a genome sequence in FASTA format, a preprocessed transcript annotation (GTF)"
- [readme] If you want to use fortuna with human transcripts annoated in GENCODE (https://www.gencodegenes.org/human/), release 42 (GRCh38.p13, access date 17.12.2022): "human transcripts annoated in GENCODE (https://www.gencodegenes.org/human/), release 42 (GRCh38.p13, access date 17.12.2022)"

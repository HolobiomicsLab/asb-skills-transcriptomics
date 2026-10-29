---
name: splice-site-combination-enumeration
description: Use when reconstructing fortuna's Step B fragment-set construction (F
  from F1:1 and Fmax) from a preprocessed GTF, or when verifying that a fragment set
  is correct (all fragments derivable from T) and complete (any length-l read is mappable).
when_to_use_negative:
- The read sequences contain splice junctions absent from any annotation and you need
  de novo splice-site discovery — this skill only combines annotated splice sites;
  novel sites require fortuna's optional step of incorporating sites found by a spliced
  aligner on reads unmapped in step (D)
- You need full transcript-level quantification rather than counts of elementary splicing
  units (signature counts) from pseudoaligned fragments
- The annotation has not been segmented (no NodeId subexon space); running --index
  on a raw GTF would not produce valid signature names
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: fortuna
  role: Enumerates feasible splice-site combinations and builds the trimmed/merged
    fragment set F via fortuna --index; includes bin/processGTF.sh for segmentation
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Pseudoaligns length-l reads to the constructed fragment set; bundled with
    fortuna and used to test Theorem 1 completeness
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Bundled dependency of fortuna used when reading/writing genomic sequence and
    annotation data
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:13a6a3cfe98e6b2e85bc4f6209787421d7eb27a61101b680f3294902eca4ff67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - CMake
  - fortuna
  - htslib
  - kallisto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# splice-site-combination-enumeration

## Summary

Enumerates feasible transcript signatures for fortuna by combining annotated donor and acceptor splice sites per gene, then builds the trimmed, merged transcript fragment set F from the annotated-fragment set F1:1 and the maximally permissive set Fmax so that every length-l read from a transcript in T can be pseudoaligned by kallisto.

## When to use

Use when reconstructing fortuna's Step B fragment-set construction (F from F1:1 and Fmax) from a preprocessed GTF, or when verifying that a fragment set is correct (all fragments derivable from T) and complete (any length-l read is mappable).

## When NOT to use

- The read sequences contain splice junctions absent from any annotation and you need de novo splice-site discovery — this skill only combines annotated splice sites; novel sites require fortuna's optional step of incorporating sites found by a spliced aligner on reads unmapped in step (D)
- You need full transcript-level quantification rather than counts of elementary splicing units (signature counts) from pseudoaligned fragments
- The annotation has not been segmented (no NodeId subexon space); running --index on a raw GTF would not produce valid signature names

## Inputs

- Genome sequence in FASTA format
- Transcript annotation in GTF format (e.g. GENCODE human release 42, GRCh38.p13)
- Segmented/preprocessed GTF (e.g. annotation.rg.gtf or sample.rg.gtf) with NodeId subexon identifiers, produced by bin/processGTF.sh
- Target read length (e.g. 75bp, 100bp, or 150bp) used for the -rl parameter

## Outputs

- Fragment FASTA catalog with signature-encoding, pipe-separated subexon-ID headers (e.g. 223832|223833|223836)
- Fragment descriptions in GTF format (via -outgtf, optional)
- kallisto index built from the fragment set F
- Fragment-set size and redundancy statistics for F, F1:1, and Fmax

## How to apply

First preprocess the GTF annotation with fortuna's bin/processGTF.sh to obtain minimal disjoint subexons bounded by splice sites, identified by the NodeId field. Per gene, enumerate feasible signatures by combining annotated donor/acceptor sites: build F1:1 (fragments of annotated transcripts) and Fmax (the maximally permissive guess, obtained with -lgo 0, combining all donor/acceptor sites of the same gene), capped at the default 25000 fragments per gene (-Mc). Apply the trimming and merging construction (Supplementary Section S2.2, equations 1-4) to F1:1 and Fmax to derive the final fragment set F, emitting sequences via fortuna --index with -outfa/-infa/-ind and the target read length -rl set to the longest read in the sample. Write the fragment catalog with signature-encoding FASTA headers of pipe-separated subexon IDs (e.g. 223832|223833|223836) and optionally fragment descriptions via -outgtf. Finally run Theorem 1 property tests: every fragment must be derivable from some transcript in T, and any synthetic length-l read pseudoalignable by kallisto must map to at least one fragment; confirm the size/redundancy ordering |F| < |F1:1| with F less redundant than Fmax (Supplementary Table S9).

## Related tools

- **fortuna** (Enumerates feasible splice-site combinations and builds the trimmed/merged fragment set F via fortuna --index; includes bin/processGTF.sh for segmentation) — https://github.com/canzarlab/fortuna
- **kallisto** (Pseudoaligns length-l reads to the constructed fragment set; bundled with fortuna and used to test Theorem 1 completeness) — https://github.com/pachterlab/kallisto
- **htslib** (Bundled dependency of fortuna used when reading/writing genomic sequence and annotation data) — https://github.com/samtools/htslib

## Examples

```
bash processGTF.sh /path/to/the/annotation.gtf && ./fortuna --index -infa genome.fa -ind annotation.rg.gtf -rl 150 -outfa fragments.fa -outgtf fragments.gtf
```

## Evaluation signals

- Fragment FASTA parses and every header name references valid subexon NodeIds from the segmented GTF (e.g. pipe-separated IDs like 223832|223833|223836)
- Theorem 1 correctness test passes: every fragment in F is derivable from some transcript in T
- Theorem 1 completeness test passes: any synthetic length-l read pseudoalignable by kallisto maps to at least one fragment in F
- Size/redundancy ordering holds: |F| < |F1:1| and F is less redundant than Fmax, reproducing Supplementary Table S9
- Per-gene enumeration respects the default cap of 25000 fragments (-Mc) and Fmax generation uses -lgo 0

## Limitations

- Fragment generation is capped at a default maximum of 25000 fragments per gene (-Mc), which may exclude combinations for genes with many annotated splice sites
- Only annotated splice sites are combined; genuinely novel splice sites are recovered only via the optional step of incorporating sites found by any spliced aligner on reads unmapped in step (D)
- -rl must be set to the longest read in the sample; a mismatched target read length breaks the guarantee that any length-l read sampled from T is mappable to some fragment

## Evidence

- [intro] It then (B) creates a set of sequence fragments of annotated and guessed novel transcripts that contain all possible combinations of unspliced exonic segments.: "It then (B) creates a set of sequence fragments of annotated and guessed novel transcripts that contain all possible combinations of unspliced exonic segments."
- [methods] Steps B and C are executed using fortuna's "--index" command.: "Steps B and C are executed using fortuna's "--index" command."
- [methods] A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] The IDs of the subexons are taken from the input GTF file sample.rg.gtf (field NodeId).: "The IDs of the subexons are taken from the input GTF file sample.rg.gtf (field NodeId). Note that the GTF file has been pre-processed using script  bin/processGTF.sh ."
- [readme] The above example implies that a signature comprised of subexons 223832, 223833, and 223836 has a count of 2274.: "The above example implies that a signature comprised of subexons 223832, 223833, and 223836 has a count of 2274."

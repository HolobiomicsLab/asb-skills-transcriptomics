---
name: annotated-splice-site-recombination-fragment-enumeration
description: Use when building an alignment-free splicing quantification pipeline
  that must first guess novel transcript fragments from a GTF annotation's splice
  sites, before pseudoaligning reads with kallisto.
when_to_use_negative:
- When splice sites not present in the annotation must be included — fortuna's Step
  A only recombines annotated donor/acceptor sites; novel splice sites from spliced
  aligners are incorporated only later, from reads unmapped in pseudoalignment step
  (D)
- When a full alignment-based quantification is required — this enumeration feeds
  kallisto pseudoalignment, not align-and-count workflows
- When the GTF has not been segmented by bin/processGTF.sh — raw unsegmented GTF lacks
  the NodeId subexon records the enumeration requires
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3512
tools:
- name: fortuna
  role: Performs Step A fragment enumeration (-lgo strategy options, -Mc fragment
    cap, -outgtf output) and subsequent --index build
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Bundled dependency; the enumerated fragment set is turned into a kallisto
    index (Step C) for pseudoalignment (Step D)
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Bundled dependency of fortuna used for GTF/sequence I/O during annotation
    parsing
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:6dea61831525798a7cd3d749768aa4decd91a434b11c94c434f76ae06c9d7184
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - fortuna
  - htslib
  - kallisto
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# annotated-splice-site-recombination-fragment-enumeration

## Summary

Reconstructs fortuna's Step A virtual transcriptome (T^as_g): for each gene, enumerate novel transcript fragments by recombining annotated donor and acceptor splice sites only — implementing exon skipping and exon-boundary modification without ever merging touching exons.

## When to use

Use when building an alignment-free splicing quantification pipeline that must first guess novel transcript fragments from a GTF annotation's splice sites, before pseudoaligning reads with kallisto.

## When NOT to use

- When splice sites not present in the annotation must be included — fortuna's Step A only recombines annotated donor/acceptor sites; novel splice sites from spliced aligners are incorporated only later, from reads unmapped in pseudoalignment step (D)
- When a full alignment-based quantification is required — this enumeration feeds kallisto pseudoalignment, not align-and-count workflows
- When the GTF has not been segmented by bin/processGTF.sh — raw unsegmented GTF lacks the NodeId subexon records the enumeration requires

## Inputs

- Preprocessed/segmented GTF transcript annotation (e.g. annotation.rg.gtf from bin/processGTF.sh) with gene_id, transcript_id, and NodeId fields
- Genome sequence in FASTA format (consumed by fortuna --index for fragment sequence extraction)
- Target read length

## Outputs

- GTF file cataloging novel transcript fragments (fortuna's -outgtf-style output) preserving gene_id, transcript_id, NodeId
- Fragment set (T^as_g) capped at 25000 fragments per gene, used downstream to build a kallisto index

## How to apply

Parse a preprocessed (segmented) GTF — e.g. annotation.rg.gtf produced by bin/processGTF.sh, which computes minimal disjoint subexon segments bounded by splice sites and carries NodeId fields — to extract per-gene subexon records and annotated donor/acceptor sites. For each annotated transcript t in the gene's set Tg, generate derivative transcripts t' that skip exons of t and modify boundaries of remaining exons, using ONLY donor and acceptor sites contained in other transcripts of Tg; this is fortuna's least restrictive -lgo strategy (set T^ap_g). Constrain recombination to splice sites within a known reference transcript's TSS/TES span (-lgo option 2 restriction) and cap fragments per gene at the -Mc default of 25000. Do not merge touching exons when recombining. Write the fragment catalog as a GTF preserving gene_id, transcript_id, and NodeId fields from the input annotation.

## Related tools

- **fortuna** (Performs Step A fragment enumeration (-lgo strategy options, -Mc fragment cap, -outgtf output) and subsequent --index build) — https://github.com/canzarlab/fortuna
- **kallisto** (Bundled dependency; the enumerated fragment set is turned into a kallisto index (Step C) for pseudoalignment (Step D)) — https://github.com/pachterlab/kallisto
- **htslib** (Bundled dependency of fortuna used for GTF/sequence I/O during annotation parsing) — https://github.com/samtools/htslib

## Examples

```
bash bin/processGTF.sh /path/to/annotation.gtf && ./fortuna --index -g annotation.rg.gtf -f genome.fa -k 75 -Mc 25000 -lgo 2 -outgtf fragments.gtf
```

## Evaluation signals

- The fragment catalog must contain fortuna's t3/t4-type worked examples and exclude the Supplementary Fig. S2 counter-examples and fragment t7 (t7 is not generated because touching exons are never merged).
- On the STAT1 example (~240,000 reads, sample 4, patient id 11352.p1), downstream fortuna must map over 99% of reads and report slightly over 1000 reads supporting the novel exon skipping at chr2:190984393-190986854.
- alt.tsv must recover the expected record 'chr2 190984393 190986854 ... Locus_18066 ES 1256' referencing transcripts NM_007315.3, NM_139266.2, XM_006712718.1, XM_017004783.2, XR_001738914.2, XR_001738915.2.
- No enumerated fragment may use a donor or acceptor site absent from the annotated transcript set Tg, and every fragment must lie within a reference transcript's TSS/TES span (-lgo option 2).
- Fragment count per gene must not exceed the -Mc default of 25000.

## Limitations

- Recombination is restricted to annotated splice sites; truly novel donor/acceptor sites are out of scope for Step A and must come from a spliced aligner via the optional post-pseudoalignment step.
- The -Mc default caps enumeration at 25000 fragments per gene, which can truncate very complex recombination spaces.
- Novel splice-site candidates must satisfy the intronic motif conditions checked during fortuna's optional incorporation of unmapped-read splice sites (motifs of lengths 3, 10, and 3 around the site).

## Evidence

- [abstract] Enumeration constrained to annotated splice sites: "Here, we propose fortuna, a method that guesses novel combinations of annotated splice sites to create transcript fragments."
- [readme] Step A description in README: "fortuna starts (A) by "guessing" novel transcripts based on annotated splice sites. It then (B) creates a set of sequence fragments of annotated and guessed novel transcripts"
- [readme] GTF preprocessing requirement: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [methods] Index build consumes preprocessed GTF and FASTA: "Given a genome sequence in FASTA format, a preprocessed transcript annotation (GTF), and a target read length, it builds an index"
- [methods] STAT1 validation benchmark: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
- [intro] Optional novel splice site incorporation from unmapped reads: [evidence span withheld — anchor status: not_found]

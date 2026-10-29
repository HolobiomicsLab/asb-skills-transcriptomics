---
name: novel-splicing-event-annotation
description: Use when RNA-seq FASTQ reads must be quantified against a GTF annotation
  while simultaneously detecting unannotated splicing events (exon skipping, intron
  retention, alternative donor/acceptor) with per-event read counts, without a spliced
  aligner.
when_to_use_negative:
- The task requires full spliced genome alignment of all reads rather than counting
  reads supporting annotated-splice-site-combination events (use STAR or another aligner
  as the primary tool, at most via fortuna --refine)
- Read length does not match the -rl read length used to build the fortuna index (a
  separate index per read length, e.g. 75/100/150 bp, is required)
- Novel events must include splice sites absent from the annotation and no spliced-aligner
  BAM is available for --refine refinement
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: fortuna
  role: 'Core tool: preprocesses the GTF (bin/processGTF.sh), builds the kallisto
    fragment index (--index), pseudoaligns reads and annotates novel events (--quant),
    and incorporates aligner-identified novel splice sites (--refine)'
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Performs the pseudoalignment of reads to fortuna's transcript fragments; fortuna
    derives signature counts and event support from kallisto's equivalence classes
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Bundled dependency used by fortuna for reading high-throughput sequencing
    files (e.g. the spliced-aligner BAM during --refine)
  repo: https://github.com/samtools/htslib
- name: STAR
  role: Optional spliced aligner whose 1-pass genome BAM of unmapped-read-derived
    novel splice sites can be incorporated via 'fortuna --refine'; also used as a
    comparison aligner in benchmarks
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:a666040ba8c3a269c578caafbf3444da4204af0c234f79793398dfa030e02f55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - htslib
  - kallisto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# novel-splicing-event-annotation

## Summary

Annotation-free-of-alignment detection and counting of novel splicing events (ES, IR, AA, AD, AP, IE, XX) using fortuna: it guesses novel combinations of annotated splice sites, builds a kallisto fragment index, pseudoaligns reads, and emits per-event supporting read counts (alt.tsv) alongside elementary signature counts (cnt.tsv).

## When to use

Use when RNA-seq FASTQ reads must be quantified against a GTF annotation while simultaneously detecting unannotated splicing events (exon skipping, intron retention, alternative donor/acceptor) with per-event read counts, without a spliced aligner.

## When NOT to use

- The task requires full spliced genome alignment of all reads rather than counting reads supporting annotated-splice-site-combination events (use STAR or another aligner as the primary tool, at most via fortuna --refine)
- Read length does not match the -rl read length used to build the fortuna index (a separate index per read length, e.g. 75/100/150 bp, is required)
- Novel events must include splice sites absent from the annotation and no spliced-aligner BAM is available for --refine refinement

## Inputs

- Transcript annotation in GTF format (e.g. GENCODE human release 42, GRCh38.p13)
- Genome sequence in FASTA format
- RNA-seq reads in FASTQ (single-end, or paired-end left/right mate FASTQs)
- Optional: spliced-aligner BAM (e.g. STAR 1-pass genome BAM) for --refine

## Outputs

- Segmented annotation (.rg.gtf) with minimal disjoint subexon segments and NodeId fields
- kallisto index of annotated and guessed novel transcript fragments
- cnt.tsv signature counts (pipe-separated subexon IDs with per-signature counts, e.g. '223832|223833|223836\t2274')
- alt.tsv novel splicing events with chromosome, coordinates, affected transcripts, event type (ES/IR/AA/AD/AP/IE/XX), and supporting read counts
- cnt.refined.tsv / alt.refined.tsv when --refine is run with a spliced-aligner BAM

## How to apply

First preprocess the GTF with fortuna's bin/processGTF.sh to compute minimal disjoint subexon segments bounded by splice sites (producing the .rg.gtf segmented annotation). Build a kallisto-based index with 'fortuna --index' from the genome FASTA, the segmented GTF, and -rl set to the longest read length, using the -lgo strategy to guess novel transcript fragments from annotated splice sites (optionally capping them at -Mc 25000 fragments per gene). Run 'fortuna --quant' on each FASTQ (single- or paired-end, with --p/--pairs merging for mates) to pseudoalign reads and produce cnt.tsv (pipe-separated subexon IDs from the rg.gtf NodeId field with counts) and alt.tsv (novel events annotated as ES, IR, AA, AD, AP, IE, XX with supporting read counts). Optionally run 'fortuna --refine' with a STAR-generated BAM to incorporate novel splice sites from previously unmapped reads, yielding cnt.refined.tsv and alt.refined.tsv. The rationale is that combining annotated splice sites captures novel event combinations without alignment, which was ~7x faster than align-and-count and mapped mismatch-containing reads across novel junctions more accurately than STAR or Whippet.

## Related tools

- **fortuna** (Core tool: preprocesses the GTF (bin/processGTF.sh), builds the kallisto fragment index (--index), pseudoaligns reads and annotates novel events (--quant), and incorporates aligner-identified novel splice sites (--refine)) — https://github.com/canzarlab/fortuna
- **kallisto** (Performs the pseudoalignment of reads to fortuna's transcript fragments; fortuna derives signature counts and event support from kallisto's equivalence classes) — https://github.com/pachterlab/kallisto
- **htslib** (Bundled dependency used by fortuna for reading high-throughput sequencing files (e.g. the spliced-aligner BAM during --refine)) — https://github.com/samtools/htslib
- **STAR** (Optional spliced aligner whose 1-pass genome BAM of unmapped-read-derived novel splice sites can be incorporated via 'fortuna --refine'; also used as a comparison aligner in benchmarks)

## Examples

```
grep -P "chr2\t190984393\t190986854\t" alt.tsv
```

## Evaluation signals

- On the README example dataset (~240,000 reads from sample 4 mapping to STAT1), fortuna should map over 99% of reads and 'grep -P "chr2\t190984393\t190986854\t" alt.tsv' must return an ES event at gene Locus_18066 supported by 1256 reads
- cnt.tsv lines must be pipe-separated subexon IDs whose identifiers match the NodeId field of the processGTF.sh-generated sample.rg.gtf
- Rebuilt per-event supporting counts must reproduce Supplementary Table S3 (e.g. sample 20: 196 reads jointly supported by STAR and fortuna that Whippet left unmapped, with every STAR-mapped junction read identically mapped by fortuna)
- Drosophila tissue catalog should reproduce Fig. 6 / Supplementary Table S10 distributions: ~2% of events supported by >=150 reads, >60% by <=5 reads, and at RPM >= 1 ~7.5% shared vs 13%/15%/48% IFM/leg/brain-specific
- Simulated 75/100 bp benchmarks should reproduce the Fig. 4 / Supplementary Table S1 pattern, with fortuna mapping over 99% of simulated reads

## Limitations

- The --index step is read-length specific: separate indices must be built per read length (-rl 75 vs 100 vs 150), and a mismatched index/FASTQ length invalidates quantification
- Without --refine, novel events are restricted to combinations of already-annotated splice sites; splice sites found only by spliced aligners from unmapped reads require a BAM (e.g. STAR) to be incorporated
- Novel-fragment guessing is capped by the -Mc limit (25000 fragments per gene), which may miss event combinations in heavily annotated or complex genes
- Cross-tool comparison caveats: Whippet left some junction reads unmapped, and downstream counting tools (grep -P for alt.tsv) behave differently on Mac OS (omit -P or use ctrl+V tab)

## Evidence

- [abstract] fortuna guesses novel combinations of annotated splice sites to create transcript fragments and derives per-event counts via kallisto pseudoalignment: "Here, we propose fortuna, a method that guesses novel combinations of annotated splice sites to create transcript fragments."
- [abstract] Signature counts come from kallisto equivalence classes over elementary splicing units: "It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes."
- [readme] GTF preprocessing computes minimal disjoint splice-site-bounded segments before indexing: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] Example validation: >99% mapping rate and >1000 supporting reads for the chr2:190984393-190986854 ES event: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."

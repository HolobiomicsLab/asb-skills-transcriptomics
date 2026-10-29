---
name: spliceu-reference-construction
description: Use when preparing a scRNA-seq splicing-status analysis (e.g., Forseti
  prediction) that requires reads to align to both spliced and unspliced transcript
  versions; triggered by having a genome .fasta and .gtf but no augmented spliceu
  index.
when_to_use_negative:
- You already have a pre-built spliceu reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv)
  — rebuilding is unnecessary.
- Your analysis only needs standard spliced transcript quantification (e.g., GeneFull
  UMI counts alone) and does not require unspliced/ambiguous read disambiguation.
- You lack a genome FASTA and matching GTF for the same genome build, which are both
  required inputs.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Forseti
  role: Provides the build_spliceu.R preprocessing script in forseti/preprocess/ that
    constructs the spliceu augmented reference
  repo: https://github.com/COMBINE-lab/forseti
- name: R
  role: Language in which build_spliceu.R is implemented and invoked via Rscript
  repo: https://github.com/COMBINE-lab/forseti
- name: conda
  role: Creates the isolated build_spliceu_r environment from forseti/envs/build_spliceu_r.yml
    to handle R/Python conflicts
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:f82ed8421193a06cdf535d57e7af32629875fb13db431e91eb752261c6d291b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - R
  - bedtools
  - conda
  - samtools
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spliceu-reference-construction

## Summary

Builds a spliceu (spliced + unspliced) augmented transcriptome reference from a genome FASTA and gene annotation GTF, so each transcript has both a spliced (exon concatenation) and an unspliced (genomic span of all exons) representation, enabling read compatibility with both spliced and unspliced transcripts.

## When to use

Use when preparing a scRNA-seq splicing-status analysis (e.g., Forseti prediction) that requires reads to align to both spliced and unspliced transcript versions; triggered by having a genome .fasta and .gtf but no augmented spliceu index.

## When NOT to use

- You already have a pre-built spliceu reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv) — rebuilding is unnecessary.
- Your analysis only needs standard spliced transcript quantification (e.g., GeneFull UMI counts alone) and does not require unspliced/ambiguous read disambiguation.
- You lack a genome FASTA and matching GTF for the same genome build, which are both required inputs.

## Inputs

- Genome FASTA (e.g., refdata-gex-GRCh38-2020-A fasta/genome.fa)
- Gene annotation GTF (e.g., refdata-gex-GRCh38-2020-A genes/genes.gtf)
- Output index directory path
- Thread count (e.g., 12)

## Outputs

- spliceu.gtf (augmented gene annotation GTF with spliced + unspliced transcripts)
- spliceu.fa (augmented transcriptome FASTA)
- t2g_3col.tsv (transcript-to-gene mapping TSV)

## How to apply

Obtain a standard 10X-style reference (e.g., refdata-gex-GRCh38-2020-A containing genome.fa under fasta/ and genes.gtf under genes/), then create the isolated conda environment build_spliceu_r from forseti/envs/build_spliceu_r.yml to avoid R/Python conflicts. Run the R script build_spliceu.R with three arguments — genome FASTA, genes GTF, and output index directory — plus a thread count (12 in the paper's workflow); spliced transcripts are defined as the concatenation of the corresponding exons and each unspliced transcript as the underlying genomic span of all exons of the transcript. Move genome.fa and genes.gtf into the index directory so all reference files co-locate. The directory must then contain exactly three spliceu artifacts: spliceu.gtf, spliceu.fa, and t2g_3col.tsv, which are subsequently used to generate STAR genome indices and as inputs to forseti_predictor.

## Related tools

- **Forseti** (Provides the build_spliceu.R preprocessing script in forseti/preprocess/ that constructs the spliceu augmented reference) — https://github.com/COMBINE-lab/forseti
- **R** (Language in which build_spliceu.R is implemented and invoked via Rscript) — https://github.com/COMBINE-lab/forseti
- **conda** (Creates the isolated build_spliceu_r environment from forseti/envs/build_spliceu_r.yml to handle R/Python conflicts)

## Examples

```
Rscript $FST_SAMPLE_DIR/forseti/preprocess/build_spliceu.R $REF_DIR/fasta/genome.fa $REF_DIR/genes/genes.gtf $IDX_DIR 12
```

## Evaluation signals

- The index directory contains all three expected artifacts: spliceu.gtf, spliceu.fa, and t2g_3col.tsv.
- build_spliceu.R completes without error.
- genome.fa and genes.gtf have been moved into $IDX_DIR so all reference files co-locate.
- Downstream STAR genomeGenerate using spliceu.gtf (with --sjdbOverhang 100) succeeds against the built index.
- t2g_3col.tsv maps each augmented spliced/unspliced transcript back to its original gene.

## Limitations

- Separate conda environments are required: build_spliceu_r for index building and forseti_test for the rest of the workflow, to handle potential conflicts between R and Python environments.
- The augmented reference doubles the transcriptome content (both spliced and unspliced per transcript), and the spliceu index must be rebuilt from scratch for each new genome build/annotation combination.
- Input GTF and genome FASTA must correspond to the same reference build (e.g., GRCh38-2020-A).

## Evidence

- [results] The spliceu reference contains both a spliced and an unspliced representation for each transcript annotation: spliced transcripts are defined as the concatenation of the corresponding exons, and each unspliced transcript is defined as the underlying genomic span of all exons of the transcript.: [evidence span withheld — anchor status: not_found]
- [readme] The R script `build_spliceu.R` will build a _spliceu_ (spliced + unspliced), an augmented transcriptome reference consisting of a augmented gene annotation GTF file, a transcriptome FASTA file, and a transcript-to-gene mapping TSV file.: "build a _spliceu_ (spliced + unspliced), an augmented transcriptome reference consisting of a augmented gene annotation GTF file, a transcriptome FASTA file"
- [methods] Run the R preprocessing script: Rscript forseti/preprocess/build_spliceu.R $REF_DIR/fasta/genome.fa $REF_DIR/genes/genes.gtf $IDX_DIR 12: "Rscript forseti/preprocess/build_spliceu.R $REF_DIR/fasta/genome.fa $REF_DIR/genes/genes.gtf $IDX_DIR 12"
- [readme] To handle the potential conflicts for environments having both R and python, we have two seperate envs: _build_spliceu_r_ for building spliceu index.: "To handle the potential conflicts for environments having both R and python, we have two seperate envs"

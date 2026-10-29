---
name: illumina-sequencing-error-simulation
description: Use when building simulated scRNA-seq read benchmarks from a spliceu
  (spliced + unspliced) reference and the simulated FASTQs must carry realistic sequencing
  errors to faithfully mimic experimental ambiguity.
when_to_use_negative:
- When evaluating Forseti directly on experimental scRNA-seq FASTQs — real reads already
  carry sequencing errors and should not be re-simulated
- When the goal is a simulated benchmark without error realism (e.g. testing alignment-only
  behavior), InSilicoSeq error introduction can be omitted
- When the source library is single-end — the described simulation and evaluation
  workflow operates on paired-end reads
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: InSilicoSeq
  role: Introduces Illumina sequencing errors into the simulated paired-end reads
- name: Forseti
  role: Predicts splicing status on the error-bearing simulated reads; its AUC on
    simulated data validates the simulation against experimental data
  repo: https://github.com/COMBINE-lab/forseti
- name: simpleaf
  role: Processes the 10X PBMC1k (v3) FASTQs against the spliceu reference to obtain
    the 'S'/'U'/'A' counts that seed the simulation
- name: Nextflow (forseti-experiments pipeline)
  role: Orchestrates the simulation step that generates the two read sets with and
    without fundamental ambiguity
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:a87aca0fa4fc4342e0dee0d5fe79a210bbb6fa1c9422472014b3e919939a7cce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - InSilicoSeq
  - Nextflow (forseti-experiments pipeline)
  - Python
  - R
  - simpleaf
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Illumina sequencing error simulation

## Summary

Introduces realistic Illumina sequencing errors into simulated paired-end scRNA-seq reads using InSilicoSeq, so that benchmark reads (with and without fundamental splicing ambiguity) carry the error profile that Forseti would encounter on real data.

## When to use

Use when building simulated scRNA-seq read benchmarks from a spliceu (spliced + unspliced) reference and the simulated FASTQs must carry realistic sequencing errors to faithfully mimic experimental ambiguity.

## When NOT to use

- When evaluating Forseti directly on experimental scRNA-seq FASTQs — real reads already carry sequencing errors and should not be re-simulated
- When the goal is a simulated benchmark without error realism (e.g. testing alignment-only behavior), InSilicoSeq error introduction can be omitted
- When the source library is single-end — the described simulation and evaluation workflow operates on paired-end reads

## Inputs

- Simulated paired-end scRNA-seq reads (FASTQ) generated from spliceu spliced and unspliced transcripts, with and without fundamental ambiguity
- Empirical spliced/unspliced/ambiguous ('S'/'U'/'A') count table from simpleaf quantification against the spliceu reference
- spliceu augmented reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv) built by build_spliceu.R from refdata-gex-GRCh38-2020-A

## Outputs

- Illumina error-bearing paired-end FASTQ files for both simulation conditions (with and without fundamental ambiguity)
- Simulated evaluation sets whose count table partitions reads into 'U'/'S'/'A' categories for Forseti evaluation

## How to apply

After simulating paired-end reads from spliced and unspliced transcripts of the spliceu augmented reference (polyA priming sites chosen uniformly at random, fragment lengths sampled from the fragment length distribution), run the simulation twice: once permitting and once excluding the fundamental-ambiguity step (reads from polyA tail priming with read2 within the terminal exon, or cDNA fragments arising from an exon). Pass the simulated paired-end reads through InSilicoSeq to introduce Illumina sequencing errors into both condition sets. Simulations are seeded from experimental spliced/unspliced/ambiguous ('S'/'U'/'A') counts obtained by processing the 10X PBMC1k (v3) FASTQs with simpleaf against the spliceu reference. The error-bearing reads are then fed to Forseti; agreement of simulated-set AUC with experimental-set AUC validates that the simulation mimics real ambiguity.

## Related tools

- **InSilicoSeq** (Introduces Illumina sequencing errors into the simulated paired-end reads)
- **Forseti** (Predicts splicing status on the error-bearing simulated reads; its AUC on simulated data validates the simulation against experimental data) — https://github.com/COMBINE-lab/forseti
- **simpleaf** (Processes the 10X PBMC1k (v3) FASTQs against the spliceu reference to obtain the 'S'/'U'/'A' counts that seed the simulation)
- **Nextflow (forseti-experiments pipeline)** (Orchestrates the simulation step that generates the two read sets with and without fundamental ambiguity) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode simulation
```

## Evaluation signals

- Simulated paired-end FASTQ files exist for both simulation conditions (with and without fundamental ambiguity)
- The count table partitions simulated reads into the 'U'/'S'/'A' categories used to evaluate Forseti
- Forseti attains a mean AUC of ~0.92 on the simulated evaluation sets, consistent with ~0.88 on experimental data, supporting that the simulation mimics real ambiguity
- Simulated reads derive from the spliceu augmented reference with uniformly chosen polyA priming sites and fragment lengths sampled from the fragment length distribution

## Limitations

- Forseti is a mechanistic model and is unable to distinguish the two sources of fundamentally ambiguous reads (polyA tail priming with read2 in the terminal exon vs. exon-derived cDNA fragments), so this ambiguity class is irreducible in both simulation and prediction
- The simulation is seeded from human (refdata-gex-GRCh38-2020-A) data; the fragment length distribution model is claimed to generalize across species and tissues, but simulated error profiles are Illumina-specific
- Running the full forseti-experiments simulation/evaluation pipeline requires about 3 TB of disk space and a SLURM-configured Nextflow setup

## Evidence

- [readme] Using the the _spliceu_ augmented reference generated in step 1, we simulate two sets of sequencing reads with and without fundamental ambiguity: [evidence span withheld — anchor status: external_doc]
- [readme] fundamentally ambiguious reads are the reads that come from either polyA tail priming and the read2 is contained within the terminal exon: [evidence span withheld — anchor status: external_doc]
- [abstract] Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data.: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [methods] Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts.: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts."

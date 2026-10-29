---
name: polya-site-fragment-read-simulation
description: Use when you need simulated paired-end scRNA-seq FASTQs with known ground-truth
  splicing status—especially reads that are ambiguous ('A', compatible with both spliced
  and unspliced transcripts)—to evaluate a splicing-status or read-origin prediction
  model.
when_to_use_negative:
- When the goal is real-data quantification of spliced/unspliced/ambiguous counts
  from experimental FASTQs rather than ground-truth-labeled simulated reads
- When the downstream predictor is not evaluated against known splicing-status labels—simulation
  ground truth provides no benefit there
- When only a single-end or non-polyA-primed protocol is used, since the uniform polyA-site
  and fragment-length sampling assumptions would not apply
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Forseti
  role: Splicing-status predictor evaluated on the simulated read sets; provides the
    trained binding affinity and fragment length distribution models used in simulation
  repo: https://github.com/COMBINE-lab/forseti
- name: simpleaf
  role: Processes real PBMC1k FASTQs against the spliceu reference to obtain 'S'/'U'/'A'
    counts that seed and validate the simulation
- name: InSilicoSeq
  role: Introduces Illumina sequencing errors into the simulated paired-end reads
- name: Nextflow
  role: Runs the forseti-experiments pipeline 'simulation' mode that generates the
    two simulated read sets
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:222b27bbd60b35c3c88619bfd1ab74e60618908587241edb25b9f7980f329dcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - InSilicoSeq
  - Nextflow
  - Python
  - R
  - simpleaf
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# polya-site-fragment-read-simulation

## Summary

Simulate paired-end scRNA-seq read sets from a spliceu (spliced + unspliced) augmented reference by uniformly sampling polyA priming sites, drawing fragment lengths from an empirical fragment length distribution, and adding Illumina sequencing errors, once with and once without the fundamental-ambiguity-permitting step, to benchmark splicing-status predictors like Forseti.

## When to use

Use when you need simulated paired-end scRNA-seq FASTQs with known ground-truth splicing status—especially reads that are ambiguous ('A', compatible with both spliced and unspliced transcripts)—to evaluate a splicing-status or read-origin prediction model.

## When NOT to use

- When the goal is real-data quantification of spliced/unspliced/ambiguous counts from experimental FASTQs rather than ground-truth-labeled simulated reads
- When the downstream predictor is not evaluated against known splicing-status labels—simulation ground truth provides no benefit there
- When only a single-end or non-polyA-primed protocol is used, since the uniform polyA-site and fragment-length sampling assumptions would not apply

## Inputs

- Genome FASTA and gene annotation GTF (e.g. 10X refdata-gex-GRCh38-2020-A)
- Spliceu augmented reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv) built via build_spliceu.R
- Real paired-end scRNA-seq FASTQ files (e.g. 10X PBMC1k v3) for empirical fragment length distributions and S/U/A anchoring
- Empirical priming windows and fragment lengths fitted in prior pipeline steps (MLP binding affinity model, cubic-spline fragment length distribution)

## Outputs

- Simulated paired-end FASTQ files for two conditions: with and without fundamental ambiguity
- Count table partitioning simulated reads into 'S' (spliced), 'U' (unspliced), and 'A' (ambiguous) categories with ground-truth labels
- Evaluation set on which a predictor (e.g. Forseti) reports AUC, for comparison against experimental evaluation sets

## How to apply

First build a spliceu augmented reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv) from a genome FASTA and gene annotation GTF (e.g. 10X refdata-gex-GRCh38-2020-A) using the build_spliceu.R script (12 threads), so each transcript exists in spliced and unspliced versions. Process real paired-end data (e.g. 10X PBMC1k v3 FASTQs) with simpleaf against the spliceu reference to obtain 'S'/'U'/'A' counts that anchor the simulation. Simulate paired-end reads from the spliced and unspliced transcripts, choosing polyA priming sites uniformly at random and sampling fragment lengths from the fragment length distribution; run the simulation twice—once with and once without the fundamental-ambiguity-permitting step, where fundamentally ambiguous reads arise from polyA tail priming with read2 inside the terminal exon or from cDNA fragments arising entirely from an exon. Introduce Illumina sequencing errors into the simulated reads with InSilicoSeq, then quantify with a count table partitioning reads into 'U'/'S'/'A' categories to serve as ground truth for evaluation.

## Related tools

- **Forseti** (Splicing-status predictor evaluated on the simulated read sets; provides the trained binding affinity and fragment length distribution models used in simulation) — https://github.com/COMBINE-lab/forseti
- **simpleaf** (Processes real PBMC1k FASTQs against the spliceu reference to obtain 'S'/'U'/'A' counts that seed and validate the simulation)
- **InSilicoSeq** (Introduces Illumina sequencing errors into the simulated paired-end reads)
- **Nextflow** (Runs the forseti-experiments pipeline 'simulation' mode that generates the two simulated read sets) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode simulation
```

## Evaluation signals

- Both simulated paired-end FASTQ file sets (with and without fundamental ambiguity) exist and are non-empty
- The count table partitions reads into the 'U'/'S'/'A' categories, and the 'A' category is populated (fundamental-ambiguity condition)
- Forseti attains high AUC on simulated sets (mean AUC 0.92 on simulated data) consistent with its experimental-data AUC (0.88), supporting that the simulation mimics real ambiguity
- Simulated reads contain realistic Illumina sequencing errors introduced by InSilicoSeq
- The Nextflow 'simulation' mode completes after 'fit_models' (which provides the MLP binding affinity model and cubic-spline fragment length distribution)

## Limitations

- Forseti, as a mechanistic model, cannot distinguish between the two sources of fundamentally ambiguous reads (polyA tail priming with read2 in the terminal exon vs. cDNA fragments arising from an exon), so these are irreducible errors in evaluation
- The full forseti-experiments pipeline requires ~3 TB of disk space and a SLURM (or adapted) cluster configuration
- Nextflow cannot guarantee fetched FASTQ files are uncorrupted; the prepare_data step may fail and require removing successfully processed SRR lines from input_files/sample_url_sheet.csv and re-running
- Computing predictions on full datasets is expensive—e.g. the filtered PBMC1k BAM with 65,784,347 alignments takes ~3.3 hours on 32 threads

## Evidence

- [abstract] Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data.: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [readme] simulate two sets of sequencing reads with and without fundamental ambiguity: [evidence span withheld — anchor status: external_doc]
- [methods] Ambiguous reads compatible with both spliced and unspliced transcripts: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status"
- [readme] train MLP binding affinity and fit cubic-spline fragment length distribution: [evidence span withheld — anchor status: external_doc]
- [readme] spliceu augmented reference of spliced and unspliced transcript versions: [evidence span withheld — anchor status: external_doc]

---
name: reference-genome-expanded-reference-construction
description: 'Use when preparing a reference for alevin-fry/simpleaf single-cell RNA-seq
  quantification: you have a genome FASTA plus GTF/GFF3 annotation and need a spliced+intronic
  (splici) or spliced+unspliced (spliceu) expanded reference with a piscem index.'
when_to_use_negative:
- The reference is already indexed or the analysis consumes a pre-built piscem/alevin-fry
  index rather than raw genome FASTA + GTF
- Bulk RNA-seq or scATAC mapping, where piscem map-bulk/map-sc-atac use non-spliced
  or genome-mode references and splici transcriptome expansion is inappropriate
- Quantification against a probe set without a genome FASTA+GTF pair, unless you intend
  the probe-set CSV to FASTA conversion path rather than splici construction
tools:
- name: simpleaf
  role: Exposes the indexing phase as the `simpleaf index` sub-command; orchestrates
    roers and piscem build and propagates --use-piscem to later quant runs
  repo: https://github.com/COMBINE-lab/simpleaf
- name: roers
  role: Library used directly by simpleaf to construct the expanded splici/spliceu
    reference FASTA and transcript-to-gene mapping from genome FASTA + GTF
  repo: https://github.com/COMBINE-lab/roers
- name: piscem
  role: RNA back-end indexer invoked via `piscem build` on the expanded reference;
    also the mapper for subsequent quantification
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: Downstream consumer of the index and t2g file during generate-permit-list,
    collate, and quant
  repo: https://github.com/COMBINE-lab/alevin-fry
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:fcfbbbd9db56d25f4b5359686307d235d74014a744a16ddfc7e3a8c2958b4cc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - alevin-fry
  - piscem
  - roers
  - simpleaf
  - wget
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# reference-genome-expanded-reference-construction

## Summary

Builds an expanded (splici/spliceu) reference from a genome FASTA and GTF using roers via the simpleaf index command, then indexes it with piscem build, yielding the ref FASTA, 3-column transcript-to-gene map, and piscem index needed for alevin-fry single-cell quantification.

## When to use

Use when preparing a reference for alevin-fry/simpleaf single-cell RNA-seq quantification: you have a genome FASTA plus GTF/GFF3 annotation and need a spliced+intronic (splici) or spliced+unspliced (spliceu) expanded reference with a piscem index.

## When NOT to use

- The reference is already indexed or the analysis consumes a pre-built piscem/alevin-fry index rather than raw genome FASTA + GTF
- Bulk RNA-seq or scATAC mapping, where piscem map-bulk/map-sc-atac use non-spliced or genome-mode references and splici transcriptome expansion is inappropriate
- Quantification against a probe set without a genome FASTA+GTF pair, unless you intend the probe-set CSV to FASTA conversion path rather than splici construction

## Inputs

- Reference genome FASTA file (e.g., downloaded from Ensembl or GENCODE)
- Matching GTF/GFF3 gene annotation file
- Optional: comma-separated target list or probe-set CSV (index can take targets directly)
- ALEVIN_FRY_HOME configuration created via `simpleaf set-paths`

## Outputs

- ref/ subdirectory containing the splici (expanded) reference FASTA
- 3-column transcript-to-gene (t2g) mapping file copied into the index output
- index/ subdirectory containing the built piscem index

## How to apply

Run `simpleaf index` with --fasta and --gtf pointing at the genome FASTA and GTF; internally simpleaf calls roers to construct the expanded reference — by default spliced+intronic (splici) with k-mer length 31 and minimizer length 19 — and can alternatively build a spliced+unspliced (spliceu) reference or take targets directly. After the expanded reference is constructed, simpleaf invokes `piscem build` on it, producing an index directory. Setting the --use-piscem flag on simpleaf index records the mapper choice so subsequent simpleaf quant runs automatically map with piscem against this index. Optionally pass --dedup to roers to deduplicate identical sequences during expansion, and --struct-constraints when piscem >= 0.7.0. Validate that the output directory contains a ref/ subdirectory (splici FASTA and the copy of the 3-column transcript-to-gene file) and an index/ subdirectory (the piscem index).

## Related tools

- **simpleaf** (Exposes the indexing phase as the `simpleaf index` sub-command; orchestrates roers and piscem build and propagates --use-piscem to later quant runs) — https://github.com/COMBINE-lab/simpleaf
- **roers** (Library used directly by simpleaf to construct the expanded splici/spliceu reference FASTA and transcript-to-gene mapping from genome FASTA + GTF) — https://github.com/COMBINE-lab/roers
- **piscem** (RNA back-end indexer invoked via `piscem build` on the expanded reference; also the mapper for subsequent quantification) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream consumer of the index and t2g file during generate-permit-list, collate, and quant) — https://github.com/COMBINE-lab/alevin-fry

## Examples

```
simpleaf index --fasta genome.fa --gtf annotation.gtf  # defaults: splici reference, k=31, minimizer m=19
```

## Evaluation signals

- `simpleaf index` completes without error and the output directory contains both a `ref/` and an `index/` subdirectory
- The `ref/` subdirectory holds the splici reference FASTA and an `index/` subdirectory holds the piscem index together with the copy of the 3-column transcript-to-gene file
- piscem build succeeds, which requires the user file handle limit to be at least 2048 (`ulimit -n 2048`)
- With --use-piscem set on simpleaf index, subsequent simpleaf quant executions automatically use piscem with appropriate parameters as the mapper
- Default parameters observed: k-mer length 31 (must be odd and <= 31 for piscem) and minimizer length 19 (must be < k)

## Limitations

- piscem index building requires the file handle limit to be set to 2048; failures here are a common cause of build errors before filing GitHub issues
- piscem k must be odd and <= 31, and the minimizer size must be < k; on Apple silicon, threads (-t) should not exceed high-performance core count or build performance degrades severely
- roers intronic augmentation adds read-length-aware flanks and merged intronic sequence per gene, so the expanded reference content depends on intended read length and augmentation type (-a: intronic/gene-body/transcript-body)
- simpleaf source builds require piscem, alevin-fry, and wget to be located via ALEVIN_FRY_HOME / simpleaf set-paths before indexing

## Evidence

- [intro] simpleaf index encapsulates expanded-reference construction and piscem indexing in one command: "`simpleaf` encapsulates the process of creating an expanded reference for quantification into a single command (`index`)"
- [methods] Indexing pipeline: roers expanded reference then piscem build: "after the expanded reference is constructed, the resulting reference will be indexed with ``piscem build``"
- [methods] Input types and reference types supported by simpleaf index: "it will take a reference genome FASTA and GTF as input, from which it can build a spliced+intronic (splici) reference or a spliced+unspliced (spliceu) reference"
- [readme] roers role in building the expanded reference and t2g map (README): [evidence span withheld — anchor status: external_doc]
- [methods] roers --dedup option controlling expanded reference construction: "--dedup                  Deduplicate identical sequences in roers when building the expanded"
- [readme] piscem build constraints on k and minimizer (README): [evidence span withheld — anchor status: external_doc]

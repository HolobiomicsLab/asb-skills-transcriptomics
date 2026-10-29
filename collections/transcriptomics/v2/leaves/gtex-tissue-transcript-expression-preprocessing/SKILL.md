---
name: gtex-tissue-transcript-expression-preprocessing
description: Use when building a per-tissue sQTL catalog from GTEx V7 transcript expression
  and genotypes, or when reproducing the Garrido-Martín et al. counts of 210,485 cis
  sQTLs across 6,963 sGenes at 0.05 FDR.
when_to_use_negative:
- The analysis goal is univariate gene-expression QTL (eQTL) mapping rather than multivariate
  splicing phenotype testing — sQTLseekeR2 models vectors of transcript isoform relative
  abundances.
- The splicing phenotype is already reduced to LeafCutter intron excision ratios;
  this skill works on transcript-level expression tables with trId/geneId.
- Only effect annotation of already-called variants is needed (e.g. LoF prediction
  via LOFTEE/VEP) — no sQTL association testing is involved.
edam_operation: http://edamontology.org/operation_3236
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: sqtlseeker2-nf
  role: Nextflow pipeline orchestrating genotype indexing, transcript expression preprocessing,
    nominal and permuted sQTL association testing at 0.05 FDR
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: sQTLseekeR2
  role: R package performing the multivariate cis association test between splicing
    ratios and genetic variants, with covariate correction and permutation-based multiple
    testing correction
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine used to run sqtlseeker2-nf reproducibly per tissue group (pin
    NXF_VER, e.g. 22.04.0, since DSL1 is unsupported in recent releases)
- name: Docker
  role: Container runtime for the pipeline invocation (alternatively Singularity)
- name: STAR
  role: RNA-seq aligner used upstream to generate alignments for transcript quantification
    from GTEx tissue samples
- name: RSEM
  role: Transcript quantification producing the transcript-level expression estimates
    that feed the trexp input table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:ac1add53f10ffcea914e2254dcff67ded50008fe73f18b723283a859e78610e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - Docker
  - Nextflow
  - RSEM
  - STAR
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3236
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gtex-tissue-transcript-expression-preprocessing

## Summary

Prepare GTEx V7 tissue-level inputs (coordinate-sorted genotypes, transcript expression with trId/geneId, sample metadata with covariates, gene locations) and run the sqtlseeker2-nf Nextflow pipeline to call cis sQTLs at 0.05 FDR with sQTLseekeR2.

## When to use

Use when building a per-tissue sQTL catalog from GTEx V7 transcript expression and genotypes, or when reproducing the Garrido-Martín et al. counts of 210,485 cis sQTLs across 6,963 sGenes at 0.05 FDR.

## When NOT to use

- The analysis goal is univariate gene-expression QTL (eQTL) mapping rather than multivariate splicing phenotype testing — sQTLseekeR2 models vectors of transcript isoform relative abundances.
- The splicing phenotype is already reduced to LeafCutter intron excision ratios; this skill works on transcript-level expression tables with trId/geneId.
- Only effect annotation of already-called variants is needed (e.g. LoF prediction via LOFTEE/VEP) — no sQTL association testing is involved.

## Inputs

- GTEx V7 genotype file per tissue, coordinate-sorted, alleles coded 0/1/2 with -1 for missing
- Transcript expression table per tissue with trId and geneId columns (STAR/RSEM quantification)
- Metadata file with indId, sampleId, group columns and covariates
- Gene location file with chr, start, end and geneId columns

## Outputs

- result/groups/<group>/all-tests.nominal.tsv (nominal pass associations)
- result/groups/<group>/all-tests.permuted.tsv (permutation pass with empirical P-values)
- result/groups/<group>/sqtls-0.05fdr.permuted.tsv (significant sQTLs at 5% FDR)
- Aggregated per-tissue counts of significant sQTLs and splicing-affected genes (sGenes)

## How to apply

Assemble the four required inputs per GTEx V7 tissue: a genotype file sorted by coordinate with 0/1/2 allele coding (-1 for missing), a transcript expression file containing trId and geneId columns (from STAR/RSEM-quantified transcript abundances), a metadata file with indId, sampleId and group columns plus covariates, and a gene location file with chr, start, end and geneId. Launch sqtlseeker2-nf via Nextflow inside Docker or Singularity, passing --genotype, --trexp, --metadata, --genes and --dir for each tissue group; the pipeline first indexes the genotype file and preprocesses the transcript expression data before the nominal association pass. Set --mode permuted with --max_perm 1000 so the adaptive permutation pass follows the nominal pass and empirical P-values are computed, and enable covariate regression with --covariates true. Apply --fdr 0.05 to call significant sQTLs and optionally --min_md 0.05 to enforce the minimum effect size. Rationale: sQTLseekeR2 tests the multivariate vector of transcript isoform relative abundances rather than single events, controlling multiple testing via the permutation schema.

## Related tools

- **sqtlseeker2-nf** (Nextflow pipeline orchestrating genotype indexing, transcript expression preprocessing, nominal and permuted sQTL association testing at 0.05 FDR) — https://github.com/guigolab/sqtlseeker2-nf
- **sQTLseekeR2** (R package performing the multivariate cis association test between splicing ratios and genetic variants, with covariate correction and permutation-based multiple testing correction) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine used to run sqtlseeker2-nf reproducibly per tissue group (pin NXF_VER, e.g. 22.04.0, since DSL1 is unsupported in recent releases))
- **Docker** (Container runtime for the pipeline invocation (alternatively Singularity))
- **STAR** (RNA-seq aligner used upstream to generate alignments for transcript quantification from GTEx tissue samples)
- **RSEM** (Transcript quantification producing the transcript-level expression estimates that feed the trexp input table)

## Examples

```
nextflow run guigolab/sqtlseeker2-nf -with-docker --genotype geno.tsv.gz --trexp expr.tsv --metadata meta.tsv --genes genes.txt --dir results --mode permuted --max_perm 1000 --covariates true --fdr 0.05 --min_md 0.05
```

## Evaluation signals

- Each tissue group produces result/groups/<group>/sqtls-0.05fdr.permuted.tsv containing an fdr column computed across empirical P-values at the 0.05 threshold.
- all-tests.nominal.tsv and all-tests.permuted.tsv exist for every tissue group defined in the metadata file.
- Aggregated per-tissue sQTL totals reproduce the reported GTEx V7 catalog: 210,485 cis sQTLs affecting 6,963 sGenes (6,685 protein-coding and 278 lincRNAs).
- Input schema checks: genotype file sorted by coordinate with 0/1/2 coding and -1 missing; expression table has trId and geneId columns; metadata has indId, sampleId, group.
- Genes are tested only when enough transcripts are expressed; minimum effect size filter --min_md 0.05, if applied, must not drop counts below reproducibility tolerance.

## Limitations

- Differences in sGene vs eGene counts may be confounded by different preprocessing steps and statistical methodologies applied in sQTL vs eQTL analyses.
- Recent Nextflow releases no longer support DSL1, so NXF_VER must be pinned (e.g. 22.04.0) for the workflow to run.
- The minimum effect size filter (--min_md, default 0.05) and LD-based variant clustering (--ld, default 0 = no clustering) change the resulting catalog and must be fixed when reproducing counts.

## Evidence

- [readme] Index the genotype file. `index.genotype` compresses and indexes the genotype file to optimize the: [evidence span withheld — anchor status: external_doc]
- [readme] Test for association between splicing ratios and genetic variants in cis (nominal pass): "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [methods] ./nextflow run guigolab/sqtlseeker2-nf -with-docker: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
- [methods] --min_md MIN_MD minimum effect size reported (default: 0.05): "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [intro] we have developed a pipeline for efficient and reproducible sQTL mapping: "we have developed a pipeline for efficient and reproducible sQTL mapping"

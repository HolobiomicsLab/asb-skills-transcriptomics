---
name: covariate-adjusted-genotype-association-testing
description: Use when genotype data and transcript expression (e.g., GTEx V7 per-tissue)
  are available and the goal is to detect variants affecting the global splicing pattern
  of genes rather than single univariate splicing traits.
when_to_use_negative:
- The phenotype of interest is univariate gene expression (eQTL mapping) rather than
  a multivariate vector of transcript isoform ratios; sQTLseekeR2 is built for global
  changes in isoform relative abundances.
- You need co-localization, fine-mapping, or a statistical test for tissue-specific
  heteropleiotropic effects; the paper states currently available methods are not
  directly applicable within this multivariate, non-parametric framework.
- Only LeafCutter intron excision ratios without matched per-gene transcript expression
  and metadata/covariates are available, since the pipeline requires genotype, trexp,
  metadata and gene location files per tissue.
edam_operation: http://edamontology.org/operation_3236
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: sQTLseekeR2
  role: 'Statistical engine: indexes the genotype file, preprocesses transcript expression,
    tests variant association with splicing ratios, provides covariate correction,
    permutation-based multiple testing correction and LD-based variant clustering'
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow workflow wrapper that runs the nominal pass, optional permuted pass
    and FDR control per tissue group in a container
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine used to execute sqtlseeker2-nf reproducibly; pin with NXF_VER
    (e.g., 22.04.0) since recent releases drop DSL1
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Docker
  role: Container runtime in which the pipeline is launched (-with-docker) to ensure
    reproducible dependencies
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:659e5cbfd9af512d1d5382e2a1c39eed29c3d2378bf58e72851bd3f61747293a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3236
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# covariate-adjusted-genotype-association-testing

## Summary

Map cis splicing QTLs by testing genetic variants against a multivariate vector of transcript isoform relative abundances with sQTLseekeR2, regressing out covariates and controlling multiple testing via a permutation-based FDR schema, orchestrated reproducibly by the sqtlseeker2-nf Nextflow/Docker pipeline.

## When to use

Use when genotype data and transcript expression (e.g., GTEx V7 per-tissue) are available and the goal is to detect variants affecting the global splicing pattern of genes rather than single univariate splicing traits.

## When NOT to use

- The phenotype of interest is univariate gene expression (eQTL mapping) rather than a multivariate vector of transcript isoform ratios; sQTLseekeR2 is built for global changes in isoform relative abundances.
- You need co-localization, fine-mapping, or a statistical test for tissue-specific heteropleiotropic effects; the paper states currently available methods are not directly applicable within this multivariate, non-parametric framework.
- Only LeafCutter intron excision ratios without matched per-gene transcript expression and metadata/covariates are available, since the pipeline requires genotype, trexp, metadata and gene location files per tissue.

## Inputs

- Genotype file sorted by coordinate with 0/1/2 allele coding and -1 for missing genotypes
- Transcript expression file with trId and geneId columns (e.g., from STAR/RSEM on GTEx V7 RNA-seq)
- Metadata file with indId, sampleId and group columns plus covariate columns for regression
- Gene location file with chr, start, end and geneId columns

## Outputs

- Per-group nominal association results (all-tests.nominal.tsv)
- Per-group permuted results with empirical P-values (all-tests.permuted.tsv)
- Significant sQTL catalog per tissue at 0.05 FDR (sqtls-0.05fdr.permuted.tsv)
- Aggregated per-tissue counts of sQTLs and splicing-affected genes (sGenes)

## How to apply

For each tissue group, prepare four inputs: a coordinate-sorted genotype file (0/1/2 coding, -1 missing), a transcript expression file with trId and geneId columns, a metadata file with indId, sampleId and group columns plus covariates, and a gene location file with chr, start, end and geneId. Launch the sqtlseeker2-nf pipeline in Docker with --mode permuted and --max_perm 1000 to run the adaptive permutation pass after the nominal pass, and set --covariates true so the covariate correction implemented in sQTLseekeR2 regresses confounders out of the transcript ratios. Apply --fdr 0.05 for multiple-testing control across empirical P-values and optionally --min_md 0.05 as the minimum effect size filter. The multivariate design is essential because sQTLs tend to involve multiple splicing events and target the global splicing pattern of genes, so variants are tested against the full vector of intron excision ratios rather than one splicing metric at a time. Collect per-group outputs (all-tests.nominal.tsv, all-tests.permuted.tsv, sqtls-0.05fdr.permuted.tsv) under result/groups/<group>/ and aggregate per-tissue sQTL and sGene counts.

## Related tools

- **sQTLseekeR2** (Statistical engine: indexes the genotype file, preprocesses transcript expression, tests variant association with splicing ratios, provides covariate correction, permutation-based multiple testing correction and LD-based variant clustering) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow workflow wrapper that runs the nominal pass, optional permuted pass and FDR control per tissue group in a container) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine used to execute sqtlseeker2-nf reproducibly; pin with NXF_VER (e.g., 22.04.0) since recent releases drop DSL1) — https://github.com/guigolab/sqtlseeker2-nf
- **Docker** (Container runtime in which the pipeline is launched (-with-docker) to ensure reproducible dependencies) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
NXF_VER=22.04.0 nextflow run guigolab/sqtlseeker2-nf -with-docker --genotype geno.tsv --trexp tre.tsv --metadata meta.tsv --genes genes.tsv --dir out --mode permuted --max_perm 1000 --covariates true --fdr 0.05 --min_md 0.05
```

## Evaluation signals

- Each tissue's sqtls-0.05fdr.permuted.tsv exists and contains an fdr column computed across empirical P-values, with all reported entries at fdr <= 0.05.
- Aggregated per-tissue sQTL and sGene totals reproduce the reported GTEx V7 catalog: 210,485 cis sQTLs affecting 6,963 sGenes (6,685 protein-coding genes and 278 lincRNAs).
- All four required inputs per tissue are present and schema-correct (genotype sorted by coordinate; trexp has trId and geneId; metadata has indId, sampleId, group and covariates; gene file has chr, start, end, geneId).
- Both nominal and permuted passes complete (--max_perm 1000), producing all-tests.nominal.tsv and all-tests.permuted.tsv per group under result/groups/<group>/.
- The covariates flag is enabled (--covariates true) so covariate regression is reflected in the model before association testing.

## Limitations

- Paired sQTL/eQTL effects cannot be attributed to the same variant versus two different variants in LD.
- No specific statistical test exists to assess tissue-specific heteropleiotropic effects, and heteropleiotropy results should be considered preliminary.
- Co-localization and fine-mapping methods are not directly applicable within this multivariate, non-parametric framework.
- Differences in sGene versus eGene counts could be confounded by different preprocessing steps and statistical methodologies.

## Evidence

- [readme] Test for association between splicing ratios and genetic variants in cis (nominal pass): "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [readme] README pipeline steps including permutation pass and multiple testing control: "Obtain an empirical P-value for each phenotype (permutation pass, optional)"
- [discussion] Pipeline implementation basis: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [discussion] Multivariate rationale for testing splicing as a vector phenotype: [evidence span withheld — anchor status: not_found]
- [methods] Minimum effect size filter parameter: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [methods] Concrete pipeline launch command from methods: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
- [intro] sQTLs tend to involve multiple splicing events: "we observe that sQTLs tend to involve multiple splicing events"

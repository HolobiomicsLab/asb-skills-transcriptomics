---
name: multivariate-sqtl-mapping
description: Use when RNA-seq transcript expression (transcript usage ratios) and
  genotypes from the same individuals are available and the goal is to identify genetic
  variants affecting alternative splicing genome-wide, e.g. GTEx V7 tissue-by-tissue
  sQTL catalogs.
when_to_use_negative:
- The aim is univariate splicing-event QTL mapping (e.g. intron excision ratios from
  LeafCutter) rather than testing multivariate shifts in a gene's transcript isoform
  proportions
- Only gene-level expression is available with no transcript-level expression file
  (trId/geneId), so splicing ratios cannot be computed
- The goal is annotating coding variant consequences (LoF/splice-site effects), which
  is the role of tools like LOFTEE, not sQTL mapping
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: sQTLseekeR2
  role: 'Core statistical engine: multivariate testing of variant association with
    a gene''s vector of transcript isoform relative abundances, permutation-based
    multiple testing correction, covariate correction, and optional LD-based variant
    clustering'
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow workflow wrapper that chains genotype indexing, transcript expression
    preprocessing, nominal and permuted sQTL passes, and FDR control per tissue group
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow execution engine; install pinned with NXF_VER (e.g. 22.04.0) since
    DSL1 is no longer supported in recent releases
- name: Docker
  role: Container runtime for reproducible execution of the pipeline (or Singularity
    as an alternative)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:ef25d5be3da59fae93ef0895e395bfdd184fa48b2efc9d2b8d7fc1ce2eba2841
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multivariate-sqtl-mapping

## Summary

Map splicing quantitative trait loci (sQTLs) by testing variants against a vector of transcript isoform relative abundances per gene, using sQTLseekeR2 via the sqtlseeker2-nf Nextflow/Docker pipeline, rather than treating each splicing event as a univariate phenotype.

## When to use

Use when RNA-seq transcript expression (transcript usage ratios) and genotypes from the same individuals are available and the goal is to identify genetic variants affecting alternative splicing genome-wide, e.g. GTEx V7 tissue-by-tissue sQTL catalogs.

## When NOT to use

- The aim is univariate splicing-event QTL mapping (e.g. intron excision ratios from LeafCutter) rather than testing multivariate shifts in a gene's transcript isoform proportions
- Only gene-level expression is available with no transcript-level expression file (trId/geneId), so splicing ratios cannot be computed
- The goal is annotating coding variant consequences (LoF/splice-site effects), which is the role of tools like LOFTEE, not sQTL mapping

## Inputs

- Coordinate-sorted genotype file per tissue (genotypes coded 0/1/2, missing as -1)
- Transcript expression file with trId and geneId columns (transcript relative abundances)
- Metadata file with indId, sampleId, group columns and covariates
- Gene location file with chr, start, end and geneId
- GTEx V7 tissue transcript expression and genotypes

## Outputs

- all-tests.nominal.tsv (nominal pass results per group)
- all-tests.permuted.tsv (empirical P-values from the permutation pass)
- sqtls-0.05fdr.permuted.tsv (significant cis sQTLs at 0.05 FDR with fdr column)
- Aggregated per-tissue sQTL counts and sGene counts (e.g. 210,485 sQTLs across 6,963 sGenes for GTEx V7)

## How to apply

Prepare four inputs per tissue group from GTEx V7: a coordinate-sorted genotype file (0/1/2 coding, -1 for missing), a transcript expression file with trId and geneId columns, a metadata file with indId, sampleId, group and covariate columns, and a gene location file with chr, start, end and geneId. Run sqtlseeker2-nf with Nextflow (pin NXF_VER, e.g. 22.04.0, since DSL1 is unsupported in recent releases) inside Docker or Singularity, passing --genotype, --trexp, --metadata, --genes and --dir for each group. Run a nominal pass first, then the adaptive permutation pass with --mode permuted and --max_perm 1000 to obtain empirical P-values, enabling covariate regression with --covariates true; the multivariate framework tests association with the whole vector of transcript proportions, capturing sQTLs that shift a gene's global splicing pattern rather than single events. Call significant sQTLs at --fdr 0.05 and optionally apply the minimum effect size filter --min_md 0.05, then collect sqtls-0.05fdr.permuted.tsv per group under result/groups/<group>/ and aggregate per-tissue sQTL and sGene counts.

## Related tools

- **sQTLseekeR2** (Core statistical engine: multivariate testing of variant association with a gene's vector of transcript isoform relative abundances, permutation-based multiple testing correction, covariate correction, and optional LD-based variant clustering) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow workflow wrapper that chains genotype indexing, transcript expression preprocessing, nominal and permuted sQTL passes, and FDR control per tissue group) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow execution engine; install pinned with NXF_VER (e.g. 22.04.0) since DSL1 is no longer supported in recent releases)
- **Docker** (Container runtime for reproducible execution of the pipeline (or Singularity as an alternative))

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker --genotype gtex.v7.tissue.genotypes --trexp gtex.v7.tissue.trexp --metadata gtex.v7.tissue.metadata --genes genes.tsv --dir results --mode permuted --max_perm 1000 --covariates true --fdr 0.05
```

## Evaluation signals

- Each tissue's sqtls-0.05fdr.permuted.tsv exists and contains an fdr column computed across empirical P-values at the 0.05 threshold
- Aggregated per-tissue significant sQTL totals reproduce the reported GTEx V7 catalog counts (210,485 cis sQTLs affecting 6,963 sGenes: 6,685 protein-coding and 278 lincRNAs)
- The permutation pass was run with --mode permuted and --max_perm 1000 after the nominal pass, so empirical P-values are present in all-tests.permuted.tsv
- Genotype file is correctly sorted by coordinate and missing genotypes are coded -1; transcript expression file contains trId and geneId columns
- Covariate regression was enabled (--covariates true) so P-values are covariate-adjusted

## Limitations

- The pipeline cannot distinguish whether paired sQTL/eQTL effects are caused by the same variant or by two different variants in LD
- No specific statistical test exists to assess tissue-specific heteropleiotropic effects within this framework
- Currently available co-localization and fine-mapping methods are not directly applicable within the multivariate, non-parametric framework
- Recent Nextflow releases no longer support DSL1, so the workflow must be run with a pinned NXF_VER (e.g. 22.04.0)

## Evidence

- [intro] Multivariate sQTL testing rationale: "we propose an approach that takes into account the intrinsically multivariate nature of alternative splicing: variants are tested for association with a vector of AS phenotypes"
- [methods] Pipeline implementation: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [methods] Effect size filter parameter: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [readme] Pipeline analysis steps: "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [readme] Permutation-based correction enhancement: "Permutation-based schema for multiple testing correction "

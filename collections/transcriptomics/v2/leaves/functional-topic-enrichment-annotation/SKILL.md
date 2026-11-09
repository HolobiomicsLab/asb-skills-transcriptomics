---
name: functional-topic-enrichment-annotation
description: 'Use when interpreting MUSIC topic model output from Perturb-seq/CROP-seq/CRISP-seq
  data: after Get_topics()/Select_topic_number() have fit topics, to assign each topic
  a functional label via enrichment before computing perturbation impacts.'
when_to_use_negative:
- When the data has not yet been modeled with topics (no Get_topics/Select_topic_number
  output) — enrichment on raw gene lists misses the topic structure
- When the organism is not supported by the chosen species annotation (Topic_func_anno
  invoked with species="Hs" for human; non-human data needs a supported species value)
- When the goal is perturbation calling rather than functional interpretation — use
  Rank_overall/Rank_specific outputs instead of re-annotating topics
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MUSIC
  role: Provides Topic_func_anno(), the function that annotates each topic's function
    using gene occurrence probabilities from the topic model
  repo: https://github.com/bm2-lab/MUSIC
- name: clusterProfiler
  role: Performs the functional enrichment annotation on the genes selected within
    each topic
- name: R
  role: Runtime environment for MUSIC; requires R>=3.4.1 installed via devtools
  repo: https://github.com/bm2-lab/MUSIC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:5a5be240bd98f6dd3a5e61df8d5dd7af951e3b2a27ceb5cbbead0542ccf74087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - clusterProfiler
  - devtools
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# functional-topic-enrichment-annotation

## Summary

Annotates the biological function of each topic learned by MUSIC's topic model over single-cell CRISPR screening data by performing functional enrichment with clusterProfiler on the genes with the highest occurrence probabilities in each topic, providing the functional interpretation layer for downstream perturbation effect prioritization.

## When to use

Use when interpreting MUSIC topic model output from Perturb-seq/CROP-seq/CRISP-seq data: after Get_topics()/Select_topic_number() have fit topics, to assign each topic a functional label via enrichment before computing perturbation impacts.

## When NOT to use

- When the data has not yet been modeled with topics (no Get_topics/Select_topic_number output) — enrichment on raw gene lists misses the topic structure
- When the organism is not supported by the chosen species annotation (Topic_func_anno invoked with species="Hs" for human; non-human data needs a supported species value)
- When the goal is perturbation calling rather than functional interpretation — use Rank_overall/Rank_specific outputs instead of re-annotating topics

## Inputs

- Fitted MUSIC topic model gene occurrence probabilities per topic (from Get_topics()/Select_topic_number() on the filtered, imputed expression matrix)
- Species identifier, e.g. species="Hs" for human datasets
- Upstream filtered expression profile and perturbation_information from Cell_qc/Data_imputation/Cell_filtering

## Outputs

- Per-topic functional enrichment annotations (gene sets/terms per topic from clusterProfiler)
- Functionally labeled topics used by Diff_topic_distri(), Rank_specific(), Correlation_perturbation() and TPDS computation

## How to apply

After selecting the optimal number of topics (MUSIC searches topic_number=c(4:6) with Get_topics() and Select_topic_number()), extract the occurrence probabilities of genes within each topic and use only the top genes from step 1 as input to functional enrichment. Run Topic_func_anno(species="Hs"), which internally calls clusterProfiler to perform the enrichment annotation (e.g., GO/pathway terms) for each topic. The resulting topic-function assignments let downstream steps (Diff_topic_distri, Rank_overall, Rank_specific, Correlation_perturbation, TPDS) interpret perturbation effects in a functional topic-specific way rather than as discrete clusters, since topic models give each cell a proportion of membership in each functional topic. Ensure genes are annotated for the correct species ("Hs" for human datasets such as the Perturb-seq UPR and Jurkat data) and that the expression matrix was preprocessed (MRP/RP filtering, SAVER imputation, Cell_qc and Cell_filtering) so topic gene probabilities are computed on quality-controlled data.

## Related tools

- **MUSIC** (Provides Topic_func_anno(), the function that annotates each topic's function using gene occurrence probabilities from the topic model) — https://github.com/bm2-lab/MUSIC
- **clusterProfiler** (Performs the functional enrichment annotation on the genes selected within each topic)
- **R** (Runtime environment for MUSIC; requires R>=3.4.1 installed via devtools) — https://github.com/bm2-lab/MUSIC

## Examples

```
topic_func_anno <- Topic_func_anno(species = "Hs")
```

## Evaluation signals

- Each topic receives a non-empty enrichment annotation from clusterProfiler (no topic left unannotated)
- Topic annotations are biologically coherent with the perturbation panel (e.g., in the Perturb-seq UPR dataset, topics should support distinguishing PERK vs IRE1α vs ATF6 branch effects, with TPDS recapitulating PERK=94.0, IRE1alpha=23.2, ATF6=11.0)
- Species argument matches the dataset (species="Hs" for human Perturb-seq/Jurkat data); wrong species yields empty annotations
- Downstream Rank_specific() and Correlation_perturbation() outputs reproduce paper-reported relationships (e.g., cor(Cebpb,Rela)=0.99, cor(TP53,MLH1)=0.99, cor(LCK,ZAP70)=0.93) when annotations are correct
- Topic number was selected within the recommended scope (4 to 6) before annotation

## Limitations

- MUSIC recommended topic number is restricted to a narrow scope (4 to 6), which constrains topic granularity before enrichment
- Topic-function annotation depends on gene occurrence probabilities after SAVER imputation and filtering steps; the article notes future improvements are expected to refine such filtering steps
- Off-target consideration is limited to knockout screens (not CRISPRi), which affects how topic assignments map to perturbations

## Evidence

- [discussion] Genes selected by step 1 are used to perform the functional enrichment annotation with clusterProfiler32: "Genes selected by step 1 are used to perform the functional enrichment annotation with clusterProfiler32"
- [discussion] MUSIC obtains the occurrence probabilities of genes available in each topic: "MUSIC obtains the occurrence probabilities of genes available in each topic"
- [discussion] topic modeling was performed on the entire screen dataset to compare the impact of different perturbations under the same background: "topic modeling was performed on the entire screen dataset to compare the impact of different perturbations under the same background"
- [readme] crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F): "crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F)"
- [readme] A key feature of topic model is that it allows each perturbed sample to process a proportion of membership in each functional topic rather than to categorize the sample into a discrete cluster: "A key feature of topic model is that it allows each perturbed sample to process a proportion of membership in each functional topic rather than to categorize the sample into a discrete cluster"

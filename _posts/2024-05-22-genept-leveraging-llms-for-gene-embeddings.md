---
layout: post
title: "GenePT: Leveraging LLMs for Gene and Cell Embeddings"
date: 2024-05-22 12:00:00 +0800
description: "A summary of the GenePT model, its downstream tasks, evaluation metrics, and comparison with single-cell foundation models."
tags: [GenePT, Bio, LLM, RNA-seq]
categories: [life-sciences, deep-learning]
giscus_comments: true
---

This article introduces a model called GenePT, which utilizes large language models (such as GPT-3.5) to generate gene and cell embedding representations for single-cell biology analysis. The authors designed a series of downstream tasks to validate the effectiveness of the GenePT model and compared it with existing single-cell foundation models (such as Geneformer and scGPT). The following is a summary of the downstream tasks, evaluation methods, metrics, and datasets mentioned in the article.

### Model Complexity

The specific number of parameters for the scGPT model is not directly mentioned. However, the article mentions some architectural details of the model, such as:

- Pre-trained base model: embedding size of 512, containing 12 stacked transformer blocks, each with 8 attention heads.
- Fully connected layer: hidden layer size of 512.

### Downstream Tasks

- **Gene Function Category Prediction**: Predict which of the 15 most common functional categories a gene belongs to.
- **Gene Property Prediction**: Including distinguishing dosage-sensitive from insensitive transcription factors, distinguishing different gene methylation states, etc.
- **Gene-Gene Interaction Prediction**: Predict whether two genes interact.
- **Protein-Protein Interaction Prediction**: Predict whether there is an interaction between two proteins.
- **Cell Type Annotation**: Predict cell types based on cell embedding representations.
- **Batch Effect Assessment**: Evaluate the ability of embedding representations to remove batch effects while retaining biological information.

### Evaluation Methods and Metrics

- **Accuracy**: Number of correct predictions divided by total predictions.
- **Precision**: Number of correctly predicted positives divided by total predicted positives.
- **Recall**: Number of correctly predicted positives divided by actual positives.
- **F1 Score**: Harmonic mean of precision and recall, measuring the balance between accuracy and completeness.
- **ROC-AUC**: Area under the receiver operating characteristic curve, measuring classifier performance across all possible classification thresholds.
- **Adjusted Rand Index (ARI)** and **Adjusted Mutual Information (AMI)**: Evaluate consistency between clustering results and true labels.

### Author's Metric Performance

The article mentions that GenePT performs comparably to Geneformer and other models across multiple tasks, and even better on some tasks.

#### Gene Function Category Prediction

- **GenePT**: Overall accuracy reaches 96%, with only small misclassifications in class-specific accuracy.

#### Gene-Gene Interaction Prediction (GGI)

- **GenePT**: Using the test GGI dataset with shared Gene Ontology (GO) annotations, ROC-AUC is 0.82.

#### Protein-Protein Interaction Prediction (PPI)

- **GenePT**: On three different PPI datasets, ROC-AUC values are:
  - Literature-derived dataset: ROC-AUC not explicitly given, but mentioned to outperform other models.
  - Comprehensive detection dataset: ROC-AUC not explicitly given, but mentioned to outperform other models.
  - Tissue-specific protein-protein functional network dataset: ROC-AUC not explicitly given, but mentioned to outperform other models.

#### Cell Type Annotation

**GenePT-w** and **GenePT-s** performance on multiple datasets compared to **scGPT** and **Geneformer**:

**Aorta Dataset**:
- GenePT-w: ARI=0.12, AMI=0.12, ASW=0.01
- GenePT-s: ARI=0.09, AMI=0.12, ASW=-0.04

**Artery Dataset**:
- GenePT-w: ARI=0.47, AMI=0.64, ASW=0.18
- GenePT-s: ARI=0.36, AMI=0.59, ASW=0.15

**Bones Dataset**:
- GenePT-w: ARI=0.12, AMI=0.21, ASW=-0.01
- GenePT-s: ARI=0.21, AMI=0.29, ASW=0.02

**Myeloid Dataset** (cancer types):
- GenePT-w: ARI=0.25, AMI=0.27, ASW=0.02
- GenePT-s: ARI=0.17, AMI=0.17, ASW=0.06

**Pancreas Dataset**:
- GenePT-w: ARI=0.49, AMI=0.69, ASW=0.15
- GenePT-s: ARI=0.30, AMI=0.50, ASW=0.10

**Multiple Sclerosis Dataset** (age):
- GenePT-w: ARI=0.07, AMI=0.13, ASW=-0.07
- GenePT-s: ARI=0.06, AMI=0.12, ASW=-0.03

#### Batch Effect Assessment

- **GenePT** performs well in removing batch effects while retaining biological information. Compared to **scGPT** and **Geneformer**, ARI values decrease on multiple datasets, indicating smaller batch effects.

ARI and AMI are used to measure consistency between clustering results and true labels, while ASW is used to evaluate the cohesion and separation of clustering results.
GenePT demonstrates comparable or better performance than existing models on most tasks.

### Datasets

- **Gene Function Category Prediction**: Uses specific gene function category datasets (geneformer).
- **Gene Property Prediction**: Uses open-source data provided by Theodoris et al.
- **Gene-Gene Interaction Prediction**: Uses benchmark datasets based on shared Gene Ontology annotations by Du et al.
- **Protein-Protein Interaction Prediction**: Uses multiple datasets, including HuRI, Lit-BM, and tissue-specific protein-protein functional networks provided by Greene et al.
- **Cell Type Annotation**: Uses datasets from the circulatory system (Aorta, Artery), bone tissue (Bones, Myeloid), pancreas, and immune cells from healthy individuals and multiple sclerosis patients.

### Summary

| Task Type | Metric | GenePT Performance | Dataset Name | Baseline/Comparison Method | Reference Link |
|---|---|---|---|---|---|
| Gene Functionality Class Prediction | Accuracy | 96% | - | Geneformer, etc. | - |
| Gene Property Prediction Tasks | Various | Not specified | - | Gene2vec, scGPT | [Theodoris et al. 2023](https://www.nature.com/articles/s41586-023-05903-5) |
| Gene-Gene Interaction Prediction | ROC-AUC | 0.82 | GEO expression data | Gene2vec/scGPT/Geneformer | [Du et al. 2019](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-019-5977-5) |
| Protein-Protein Interaction Prediction | ROC-AUC | Outperforms other models | HuRI, Lit-BM, Tissue-specific PPI networks | Other models | [HuRI dataset](https://www.nature.com/articles/nature5803-402), [Lit-BM dataset](https://www.cell.com/cell/fulltext/S0092-8674(14)01212-6), [Tissue-specific PPI networks](https://www.nature.com/articles/ng.3295) |
| Cell Type Annotation | ARI, AMI, ASW | Varies by dataset | Aorta, Artery, Bones, etc. | scGPT, Geneformer | [Chaffin et al. 2022](https://www.nature.com/articles/nature608174), [Li et al. 2020](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7232955/) |
| Batch Effect Assessment | ARI | Reduced, indicating smaller batch effects | Cardiomyocyte dataset, Aorta dataset | Geneformer, scGPT | [Chaffin et al. dataset](https://www.ebi.ac.uk/gxa/home), [Li et al. dataset](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE149769) |

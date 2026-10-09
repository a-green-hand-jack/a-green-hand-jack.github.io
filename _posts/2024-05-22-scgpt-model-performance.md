---
layout: post
title: "scGPT: Model Performance and Downstream Tasks"
date: 2024-05-22 12:00:00 +0800
description: "A comprehensive summary of the scGPT foundation model and its evaluation across cell type annotation, perturbation prediction, multi-batch integration, and gene regulatory network inference."
tags: [scGPT, Bio, LLM, RNA-seq]
categories: [life-sciences, deep-learning]
giscus_comments: true
---

# scGPT Model Performance Across Different Tasks

## Benchmark and Comparison Methods by Task

The following is a summary of the benchmark or comparison methods used by the authors in different tasks, along with their performance on corresponding metrics.

### Cell Type Annotation on scRNA-seq Data

- **Benchmark methods**: scBERT and TOSICA
- **Metrics**: Accuracy, Precision, Recall, Macro F1 Score
- **Performance**: Specific numerical values are not provided in the article, but it is noted that scGPT outperforms both methods on all classification metrics.

### Genetic Perturbation Response Prediction

- **Comparison methods**: GEARS and linear regression model
- **Metrics**: Pearson delta and Pearson delta on differentially expressed genes (DE genes)
- **Performance**: Specific numerical values are not provided, but it is emphasized that scGPT achieved the highest scores on all three datasets, consistently outperforming other methods by 5-20% in predicting post-perturbation gene expression changes.

### Multi-batch scRNA-seq Data Integration

- **Comparison methods**: Seurat, Harmony, and scVI
- **Metrics**: AvgBIO score (including NMIcell, ARIcell, and ASWcell), AvgBATCH score (including ASWbatch and GraphConn)
- **Performance**: Specific numerical values are not provided, but it is noted that scGPT outperforms these methods on the PBMC 10k dataset.

### Single-cell Multi-omics Data Integration

- **Comparison methods**: Seurat (v.4), scGLUE, and scMoMat
- **Metrics**: NMIcell, ARIcell, ASWcell, AvgBIO, ASWbatch, GraphConn, AvgBATCH
- **Performance**: Specific numerical values for scGLUE and scMoMat are not provided, but on the 10x Multiome PBMC dataset, scGPT is the only method that successfully generates a unique cluster for CD8+ naive cells. On the BMMC dataset, scGPT shows a more distinct clustering structure than Seurat (v.4).

### Gene Regulatory Network Inference

- **Benchmark method**: Co-expression network based on expression correlation
- **Metrics**: Not explicitly mentioned, but pathway enrichment analysis was performed to verify the quality of gene programs
- **Performance**: Specific numerical values for the co-expression network are not provided, but it is noted that scGPT identifies significantly more enriched pathways than co-expression methods at all resolutions.

# Datasets Used for scGPT Validation

The authors used multiple datasets to validate the effectiveness of the scGPT model, evaluating different tasks and metrics. The following are the datasets used, corresponding tasks, and evaluation metrics:

- **CELLxGENE scRNA-seq Dataset**:
  - **Task**: Used for pre-training to build a whole-human foundation model.
  - **Metrics**: Not explicitly mentioned, but this is the foundational dataset for model construction.

- **Multiple Sclerosis (MS) Dataset**:
  - **Task**: Cell type annotation.
  - **Metrics**: Accuracy, Precision, Recall, Macro F1 Score.

- **Myeloid Dataset**:
  - **Task**: Cell type annotation, genetic perturbation response prediction.
  - **Metrics**: Same as above.

- **Human Pancreas Dataset**:
  - **Task**: Cell type annotation.
  - **Metrics**: Same as above.

- **PBMC 10k Dataset**:
  - **Task**: Multi-batch scRNA-seq data integration.
  - **Metrics**: AvgBIO score (NMIcell, ARIcell, ASWcell), AvgBATCH score (ASWbatch, GraphConn).

- **Perirhinal Cortex Dataset**:
  - **Task**: Multi-batch scRNA-seq data integration.
  - **Metrics**: Same as above.

- **COVID-19 Dataset**:
  - **Task**: Multi-batch scRNA-seq data integration.
  - **Metrics**: Same as above.

- **Adamson, Norman, and Replogle Datasets**:
  - **Task**: Genetic perturbation response prediction.
  - **Metrics**: Pearson delta, Pearson delta on differentially expressed genes (DE genes).

- **10x Multiome PBMC Dataset**:
  - **Task**: Single-cell multi-omics (scMultiomic) data integration.
  - **Metrics**: NMIcell, ARIcell, ASWcell, AvgBIO, ASWbatch, GraphConn, AvgBATCH.

- **BMMC Dataset**:
  - **Task**: Single-cell multi-omics data integration.
  - **Metrics**: Same as above.

- **ASAP PBMC Dataset**:
  - **Task**: Single-cell multi-omics data integration.
  - **Metrics**: Same as above.

- **Immune Human Dataset**:
  - **Task**: Gene regulatory network inference.
  - **Metrics**: Not explicitly mentioned, but network quality is verified through consistency with known biology and functional groups.

# Detailed Validation Results

## Cell Type Annotation on scRNA-seq Data

**Performance**: scGPT achieved high precision (>0.8) on the human pancreas dataset, and most cell type predictions in the polygon confusion matrix were accurate. On disease datasets (such as Multiple Sclerosis MS), the scGPT model achieved high accuracy of about 0.85 in cell type annotation. On the tumor-infiltrating myeloid dataset, scGPT showed high precision in distinguishing immune cell subtypes.

## Genetic Perturbation Response Prediction

**Performance**: Evaluating scGPT's perturbation prediction capability on three Perturb-seq datasets, scGPT performed excellently in predicting perturbation responses for unseen genes. Measured by the Pearson delta metric, scGPT achieved the highest scores on all datasets, consistently outperforming other methods by 5-20%, especially in predicting gene expression changes after perturbation.

## Multi-batch scRNA-seq Data Integration

**Performance**: In integration performance evaluation on COVID-19, PBMC 10k, and perirhinal cortex datasets, scGPT demonstrated superior integration performance in cell type clustering and batch effect correction, with AvgBIO scores 5-10% higher than other methods.

## Single-cell Multi-omics Data Integration

**Performance**: On the 10x Multiome PBMC dataset, scGPT is the only method that successfully generates a unique cluster for CD8+ naive cells. On the BMMC dataset, scGPT shows a more distinct clustering structure than Seurat (v.4), with AvgBIO scores improved by 9%.

## Gene Regulatory Network Inference

**Performance**: scGPT can successfully identify gene groups related to T cell activation, such as CD3 gene groups encoding the T3 complex, as well as genes related to B cell signaling and HLA class I molecule co-receptors. On the Immune Human dataset, the scGPT model highlights CD gene networks associated with specific immune cell types.

# Summary Table

| Task Type | Task Description | Model Performance | Evaluation Metrics | Dataset | Baseline/Comparison Method |
|---|---|---|---|---|---|
| Cell Type Annotation | Classification and annotation of scRNA-seq data | High precision (>0.8) on human pancreas; ~0.85 accuracy on MS dataset | Accuracy, Precision, Recall, Macro F1 | MS dataset, Human Pancreas dataset, Myeloid dataset | scBERT, TOSICA |
| Genetic Perturbation Response Prediction | Predicting cellular response to genetic perturbation | Outperforms other methods by 5-20% on three Perturb-seq datasets | Pearson delta, Pearson delta on DE genes | Adamson, Norman, Replogle datasets | GEARS, Linear regression |
| Multi-batch scRNA-seq Integration | Integrating scRNA-seq data from different batches | Successfully separates all cell types on PBMC 10k; AvgBIO 5-10% higher | AvgBIO (NMIcell, ARIcell, ASWcell), AvgBATCH (ASWbatch, GraphConn) | COVID-19, PBMC 10k, Perirhinal Cortex datasets | Seurat, Harmony, scVI |
| Single-cell Multi-omics Integration | Integrating single-cell multi-omics data | Unique CD8+ naive cluster on 10x Multiome PBMC; clearer structure on BMMC | NMIcell, ARIcell, ASWcell, AvgBIO, ASWbatch, GraphConn, AvgBATCH | 10x Multiome PBMC, BMMC, ASAP PBMC datasets | Seurat (v.4), scGLUE, scMoMat |
| Gene Regulatory Network Inference | Inferring gene regulatory networks | Reveals biologically meaningful gene regulatory networks | Not explicitly mentioned; verified through known biology | Immune Human dataset | Co-expression network |

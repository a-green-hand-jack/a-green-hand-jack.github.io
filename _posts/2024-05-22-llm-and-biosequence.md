---
layout: post
title: "LLM and BioSequence: GenePT, scGPT, and Geneformer"
date: 2024-05-22 12:00:00 +0800
description: "A survey of how large language models are applied to biological sequences, covering GenePT, scGPT, and Geneformer with their downstream tasks and evaluation metrics."
tags: [Bio, LLM, DNA, RNA-seq, NLP]
categories: [life-sciences, deep-learning]
giscus_comments: true
---

# Preface

With the development of various sequencing technologies, accompanied by the sacrifice of hair by researchers around the world, human knowledge bases have gained a great deal of data related to genomes, transcriptomes, and proteomes. But how to reasonably utilize these hard-won data has become a big problem. For a long time, advances in bioinformatics have compensated for this deficiency. However, these methods often involve a problem of "not being general enough," meaning a certain method can only be used on one type of task or dataset. At the same time, the success of attention mechanisms in natural language generation has attracted people's attention: can we process sequence information like natural language?

Driven by such ideas, many excellent works have emerged. Here, the author focuses on three works: `Geneformer`, `GenePT`, and `scGPT`, and summarizes the downstream tasks designed to evaluate model expressiveness and their corresponding metrics.

# GenePT

GenePT uses the NCBI text description of a single gene from GPT-3.5 to generate gene embeddings. It generates single-cell embeddings through two methods: (i) averaging gene embeddings weighted by each gene's expression level; (ii) creating a sentence embedding for each cell using gene names sorted by expression level. These embeddings are then used to perform downstream tasks (such as classification of gene properties and cell types). GenePT demonstrates that literature-based large language model embeddings are a simple and effective way to obtain biological foundation models.

## Two-level Tasks

### Gene-level Tasks

- **Gene Function Category Prediction**: A multi-class prediction task based on the 15 most common functional gene categories. These category labels were curated as part of the `Geneformer` article.
- **Gene Property Prediction Tasks**: Includes 4 binary classification tasks based on open-source data provided by Theodoris et al.:
  - Distinguishing previously identified *dosage-sensitive transcription factors* from *dosage-insensitive transcription factors*
  - Distinguishing bivalent genes from non-methylated genes
  - Distinguishing Lys4-only methylated genes from non-methylated genes
  - Distinguishing long-range transcription factors from short-range transcription factors
- **Gene-Gene Interaction Prediction**: Uses the gene-gene interaction (GGI) benchmark developed and shared by Du et al. The training and test datasets include 200,000 sample pairs `(gene1, gene2, label)`, where `label` marks whether the pair of genes interact.
- **Protein-Protein (PPI) Interaction Prediction**:
  - Uses 3 datasets:
    - `The human binary protein interactions (HuRI) dataset`
    - `Comprehensive binary protein-protein interactions (Lit-BM)`
    - `Tissue-specific protein-protein functional interaction networks`
  - The basic form of these datasets is `(protein1, protein2, binary label)`
  - The binary label indicates whether there is an interaction between the two proteins
  - First uses the UniProt conversion tool to convert protein proteomic identifiers to gene names; if multiple gene names are returned, one is randomly selected
- **Unsupervised Exploration of Gene Programs**:
  - Examines interactions between genes, using `GenePT` embeddings from the *human immune tissue dataset* to construct a gene-gene interaction similarity network
  - Constructs gene networks based on cosine similarity between highly variable genes
  - Applies Louvain clustering to derive gene programs
  - Qualitatively compares *prominent gene program trends* with their *corresponding cell-specific expression levels*

### Cell-level Tasks

- **Evaluating association between embeddings and underlying cell states**:
  - Aorta: Includes 11 cell types, originally studied and published by Li et al., a random 20% data subset
  - Artery: Contains 10 cell types
  - Bone tissue:
    - Skeleton: Contains 7 cell types
    - Bone marrow: Contains 3 annotated cancer types and 11 cell types, totaling 13,468 cells
  - Pancreas: Contains 4,218 cells with 11 annotated cell types
  - Immune cells: Collected from healthy individuals and multiple sclerosis patients, totaling 3,430 cells with 18 annotated cell types from 12 donors
  - For each dataset and its associated metadata annotations, k-means clustering is applied on pre-trained GenePT, Geneformer, or scGPT embeddings to obtain clusters matching the number of classes in metadata annotations.
  - The number of clusters k is chosen to match the number of classes in metadata annotations.
  - **Adjusted Rand Index (ARI)** and **Adjusted Mutual Information (AMI)** are calculated to evaluate consistency between derived cluster labels and true metadata labels.
  - Higher alignment between inferred labels and actual labels (indicated by *higher* ARI or AMI values) indicates that embeddings capture more biological structure and signal.
  - **Average Silhouette Width (ASW)** is calculated using true annotations from original samples to evaluate cluster cohesion and separation.
- **Context-aware and Batch Integration**:
  - Evaluates whether GenePT-s embeddings are affected by common batch effects
  - Aorta dataset
  - Cardiomyocyte dataset

## Results by Task

When explaining results, it seems inevitable to mention metrics, so let's discuss them together here.

### Gene Function Category Classification

- 2D UMAP of GenePT embeddings
  - 34,000 genes
  - 15 categories
- $l_2$ regularized logistic regression
  - Training and test set split is 7:3
  - 15 different gene categories
  - `class-specific accuracies`
  - Essentially a confusion matrix

### Gene-Gene Interaction Prediction

- Compared 3 methods on the **GGI** dataset using **ROC-AUC**
  - GenePT
  - Gene2Vec
  - Geneformer
  - scGPT
  - Permuted
- Used $l_2$-regularized logistic classifier (LR classifier)

### Protein-Protein Interaction Prediction

- Metric is ROC-AUC
- 3 different models: GenePT, scGPT, geneformer
- 3 datasets: HuRI, Lit-BM, heart tissue

### Cell Type Specific Activation

- Studied cell type specific activation in GenePT-derived gene programs from the human immune tissue dataset through a "zero-shot" approach
- Constructed similarity graphs based on cosine similarity between GenePT embeddings
- Connected two genes with an edge if their similarity was greater than 0.9
- Performed Leiden clustering on the resulting graph with resolution 20

### GenePT Embeddings Predict Chromatin Dynamics and Dosage Sensitivity

- Predicting gene roles in network dynamics:
  - Dosage-sensitive vs. dosage-insensitive transcription factors
  - Bivalent genes vs. non-methylated genes
  - Lys4-only methylated vs. non-methylated genes
  - Long-range vs. short-range transcription factors
- Used scikit-learn with default parameters
- 5-fold cross-validated ROC-AUC
- $l_2$ penalized logistic regression (LR) or random forest (RF) classifier to evaluate GenePT and Gene2vec embedding performance

### GenePT Learns Biologically Meaningful Cell Representations

Evaluating association between different latent cell representations (obtained through LLM embeddings) and biological annotations (human manual labels):

- Analysis involved datasets representing cells from the circulatory system (aorta and artery), bone tissue (skeleton, bone marrow), pancreas, and immune cells collected from healthy individuals and multiple sclerosis patients.
- Used pre-trained Geneformer and scGPT embeddings for this task.
- Calculated Adjusted Rand Index (ARI) and Adjusted Mutual Information (AMI) to compare **k-means clustering derived labels with true annotations from original samples** (higher values indicate better alignment).
- Calculated **Average Silhouette Width (ASW)** using true annotations from original samples to evaluate cluster cohesion and separation.

### GenePT Embeddings Eliminate Batch Effects

Evaluating whether GenePT embeddings are robust to batch-related technical artifacts (such as patient variability).

- Used a 10% random sample from Chaffin et al.'s cardiomyocyte dataset, and a 20% random sample from the aorta dataset containing cells from healthy and dilated aorta
- Compared GenePT with pre-trained Geneformer and scGPT performance.
- Both samples were used to demonstrate Geneformer's utility.
- Goal was to distinguish different cell types
- In raw data, cell type and patient batch were strongly associated
  - High ARI between cell clusters and patient clusters
  - Indicates strong batch effects
  - But we don't want embedders to have such high batch effects
  - Fortunately, ARI for GenePT etc. is not high, indicating batch effects are not obvious

## Summary

As can be seen, the general approach for gene tasks is to first use an appropriate LLM to obtain embeddings for each gene, achieving the transformation from gene text/sequence to gene embedding. Then use the obtained gene embeddings to perform downstream tasks such as clustering or classification. This way, because we can use clustering or classification results and metrics to characterize how good the LLM's embedding effect is. Common metrics for clustering and classification algorithms include:

- **Clustering metrics**:
  - Adjusted Rand Index (ARI): Evaluates matching degree between clustering results and true labels; higher values indicate better alignment.
  - Adjusted Mutual Information (AMI): Measures mutual information between clustering results and true labels; higher values indicate clustering results closer to true labels.
  - Silhouette Score: Measures clustering consistency; higher values indicate better clustering.
  - Average Silhouette Width (ASW): Calculated using true annotations from original samples to evaluate cluster cohesion and separation.
  - These metrics can be used to compare k-means clustering derived labels with true annotations from original samples, thereby evaluating clustering effectiveness.

- **Classification metrics**:
  - Accuracy: Proportion of correctly classified samples to total samples.
  - Precision: Proportion of actual positives among predicted positives.
  - Recall: Proportion of correctly predicted positives among actual positives.
  - F1 Score: Harmonic mean of precision and recall, balancing the two.
  - AUC-ROC: Measures overall classification model performance; values closer to 1 indicate better model performance.
  - Class-Specific Accuracies: Accuracy for each class, calculated from the confusion matrix.
  - Confusion Matrix: A matrix comparing actual categories with predicted categories, helping identify which categories the model performs well or poorly on.

# scGPT

Comparing language and cell biology (where text is composed of words; similarly, cells are defined by genes), the applicability of foundation models in advancing cell biology and gene research is explored. Using emerging single-cell sequencing data, a foundation model for single-cell biology, scGPT, is constructed based on a generative pre-trained Transformer across a repository of over 33 million cells. scGPT effectively distills key biological insights about genes and cells. Through further adaptation with transfer learning, scGPT can be optimized to achieve superior performance in diverse downstream applications, including cell type annotation, multi-batch integration, multi-omics integration, perturbation response prediction, and gene network inference.

## Pre-trained Gene Embeddings

After pre-training, UMAP (Uniform Manifold Approximation and Projection) is used to visualize scGPT cell embeddings on 10% of human cells from 33 million cells. Local regions and clusters of cell types are accurately represented by different colors. Considering that the dataset contains over 400 studies, this demonstrates the pre-training's superior ability to extract biological variation.

## scGPT Improves Cell Type Annotation Precision

- Fine-tune pre-trained scGPT for cell type annotation
  - A neural network classifier takes scGPT transformer output cell embeddings as input and outputs classification predictions for cell types.
  - The entire model is trained with cross-entropy on reference datasets with expert annotations — fine-tuning
  - Then used to predict cell types on held-out query data partitions — validation
- Extensive experiments on different datasets were conducted to evaluate scGPT's performance in cell type annotation.
  - First, scGPT was used to predict cell types in the human pancreas dataset, considering 2 metrics:
    - Predictions
    - Confusion matrix
    - Embedding heatmap
  - Next, the model was tested on a Multiple Sclerosis (MS) disease dataset.
    - The model was fine-tuned on a reference partition of healthy human immune cells and evaluated based on predictions for MS condition cells.
    - Confusion matrix and heatmap were also considered as metrics.
  - Finally, fine-tuned scGPT was benchmarked against two other recent Transformer-based methods, TOSICA and scBERT, across three datasets.
    - Classification metrics include accuracy, precision, recall, and F1.

## scGPT Predicts Unseen Genetic Perturbation Responses

Recent advances in sequencing and gene editing technologies have greatly facilitated large-scale perturbation experiments, enabling characterization of cellular responses to various genetic perturbations. This approach holds great promise for revealing new gene interactions and advancing regenerative medicine. However, the vast combinatorial space of potential genetic perturbations quickly exceeds the practical limits of experimental feasibility. To overcome this limitation, scGPT can be used to leverage knowledge gained from known experiments and infer them to predict unknown responses. By utilizing self-attention mechanisms at the gene dimension, complex interactions between perturbed genes and other genes' responses can be encoded. By leveraging this capability, scGPT can effectively learn from existing experimental data and accurately predict gene expression responses to unseen perturbations.

### Predicting Unseen Gene Perturbations

- **Perturbation Prediction Task Evaluation**
  - scGPT was evaluated using three Perturb-seq datasets from leukemia cell lines:
    - **Adamson Dataset**: Contains 87 single-gene perturbations
    - **Curated Replogle Dataset**: Contains 1,823 single-gene perturbations
    - **Norman Dataset**: Contains 131 single-gene perturbations and 105 two-gene perturbations
- **Evaluation Method**
  - To evaluate scGPT's perturbation prediction capability, the model was fine-tuned on a subset of perturbations to predict perturbed expression profiles given input control cell states and intervention genes.
  - The model was then tested on perturbations involving unseen genes.
- **Evaluation Metrics**
  - **Pearson delta metric**: Measures correlation between predicted and observed post-perturbation expression changes.
  - **Top 20 most significantly changed genes metric**: Expressed as $Pearson_{\delta}$ on differentially expressed genes.
- **Results**
  - Using annotations from the original study, perturbation conditions from the same functional group were found to cluster in neighboring regions.
  - Using Leiden clustering on predicted expression, these clusters showed high association with "dominant genes" in the perturbation combinations.
- **Examples**
  - **KLF1 gene-related circular cluster**: Indicates that data points in this cluster underwent combinatorial perturbations involving KLF1 and another gene (i.e., KLF1 + X).
  - **KLF1 and CNN1 clusters**: Validated that corresponding predicted expression was specifically high in these regions, consistent with expected results from CRISPRa (CRISPR-mediated transcriptional activation) Perturb-seq experiments.
  - **Dominant gene clusters**: Demonstrated scGPT's ability to reveal associations between perturbation combinations.

### In Silico Inverse Perturbation Prediction

- **scGPT's Reverse Perturbation Prediction Capability**
  - scGPT can predict the genetic perturbation source given a resulting cell state, which is called in silico inverse perturbation prediction.
  - An ideal predictive model for such reverse prediction could be used to infer important driver genes of lineage development or facilitate discovery of potential therapeutic gene targets.
  - A hypothetical application example might be predicting CRISPR target genes that affect cell recovery from disease states.
- To demonstrate the effectiveness of inverse perturbation prediction, a subset of the Norman dataset was used, focusing on perturbations involving 20 genes.
  - This combinatorial space consists of 210 single-gene or two-gene perturbation combinations.
  - scGPT was fine-tuned using 39 (18%) known perturbations (training group in the figure).
  - The model was then tested on query cells with unseen perturbation states, and scGPT successfully predicted the perturbation source (within top-ranked predictions), thereby generating the observed outcome.
- Specific examples:
  - scGPT listed CNN1 + MAPK1 gene perturbation as the best prediction for one test example.
  - Listed FOSB + UBASH3B gene perturbation as the second-best prediction for another test example.
- Overall performance:
  - scGPT identified 91.4% of relevant perturbations in top-1 predictions on average.
  - Identified 65.7% of correct perturbations in top-8 predictions, far outperforming GEARS and differential gene baselines.

## scGPT Supports Multi-batch and Multi-omics Integration

### Multi-batch scRNA-seq Integration

- **Challenges of Integrating Multiple scRNA-seq Datasets**
  - Integrating multiple scRNA-seq datasets from different batches presents unique challenges in simultaneously preserving biological variance in integrated data and eliminating technical batch effects.
  - To integrate sequencing samples, scGPT is fine-tuned in a self-supervised manner by learning to recover masked gene expression to obtain unified cell representations.
- **Comparison with Other Integration Methods**
  - In benchmark experiments, scGPT was compared with three popular integration methods: scVI, Seurat, and Harmony.
  - Evaluation was conducted on three integration datasets: COVID-19 (18 batches), peripheral blood mononuclear cells (PBMC) 10k (two batches), and perirhinal cortex (two batches) datasets.
- **PBMC 10k Dataset Performance**
  - In the PBMC 10k dataset, scGPT successfully separated all cell types.
  - scGPT's integration performance was further supported by its high biological conservation score, with an AvgBIO score of 0.821, 5-10% higher than comparison methods.
- **AvgBIO Score**
  - AvgBIO score aggregates three cell type clustering metrics: Normalized Mutual Information (NMIcell), Adjusted Rand Index (ARIcell), and Average Silhouette Width (ASWcell)
  - scGPT also showed considerable performance in integrating the PBMC 10k dataset even without fine-tuning, highlighting the generality of pre-training.
- **Perirhinal Cortex Dataset Performance**
  - In the perirhinal cortex dataset context, scGPT remains competitive relative to all other methods.
  - This finding highlights the transferability and robustness of features learned from whole-human datasets when applied to specific organs or tissues (such as the brain).
- **Other Metric Performance**
  - scGPT consistently achieves competitive scores on all integration metrics and demonstrates strong protection of biological signals.
  - Strategies were also formulated to accelerate the fine-tuning process for integration tasks, including freezing specific model layers and excluding non-expressed genes, while maintaining results comparable to the original method.

### Single-cell Multi-omics Integration

- **scMultiomic Dataset Integration**
  - Single-cell multi-omics (scMultiomic) data combines multiple perspectives of genetic regulation, such as epigenetic, transcriptomic, and translational activities. Aggregating cell representations while preserving biological signals presents unique challenges.
  - scGPT addresses this challenge by effectively extracting integrated cell embeddings from different omics datasets.
- **Comparison with Other Methods**
  - In the 10x Multiome PBMC dataset, scGPT was compared with two state-of-the-art methods, scGLUE and Seurat (v.4).
  - scGPT is the only method capable of successfully generating a distinct cluster for CD8+ naive cells.
  - On the bone marrow mononuclear cells (BMMC) paired gene expression and protein abundance dataset, scGPT was tested.
    - scGPT presents a more distinct cluster structure than Seurat (v.4), with AvgBIO scores improved by 9%.
    - scGPT can separate CD4+ naive T cells and CD4+ activated T cells into two distinct clusters, further confirming the model's ability to capture subtle differences between immune cell subgroups.
  - In mosaic data integration settings, ATAC and selected antigen analysis (ASAP) human PBMC datasets were used as examples.
    - scGPT demonstrated superior batch correction performance, particularly in B, myeloid, and natural killer (NK) cell groups.
    - scGPT demonstrated excellent cell type clustering performance and robustness across various benchmark biological conservation metrics.

## scGPT Reveals Gene Networks for Specific Cell States

GRN (Gene Regulatory Network) interactions between transcription factors, cofactors, enhancers, and target genes mediate important biological processes. Existing GRN inference methods typically rely on static gene expression correlations or pseudotime estimates as proxies for causal graphs. scGPT optimizes through a generative model of gene expression, implicitly encoding this relationship in its gene embeddings and attention maps. Therefore, a GRN inference workflow is proposed by probing pre-trained or fine-tuned scGPT embeddings and attention maps.

Gene embeddings construct a similarity network that infers gene-gene interactions at the dataset level. Attention maps further capture unique gene network activation patterns under different cell states.

- **Gene Token Embedding Grouping and Discrimination**
  - scGPT demonstrates its ability to group functionally related genes and distinguish functionally different genes through learned gene token embeddings.
  - In Figure 5a, gene embeddings from the pre-trained scGPT model were used to visualize the similarity network of Human Leukocyte Antigen (HLA) proteins as a sanity check.
  - In this zero-shot setting, the scGPT model successfully highlighted two clusters corresponding to clearly characterized HLA classes: HLA class I and HLA class II genes.
- **HLA Class Functions**
  - These classes encode antigen-presenting proteins that play different roles in immune contexts.
  - HLA class I proteins (encoded by genes such as HLA-A, HLA-C, and HLA-E) are recognized by CD8+ T cells and mediate cytotoxic effects.
  - HLA class II proteins (encoded by HLA-DRB1, HLA-DRA, and HLA-DPA1) are recognized by CD4+ T cells and trigger broader helper functions.
  - The scGPT model was fine-tuned on the "Immune Human" dataset, and CD gene networks specific to immune cell types present in this dataset were explored.
- **GRN Analysis**
  - For GRN analysis, the same fine-tuning strategy as the integration task was used.
  - The pre-trained scGPT model successfully identified gene groups encoding the T3 complex for T cell activation (CD3E, CD3D, and CD3G), CD79A and CD79B for B cell signaling, and CD8A and CD8B as HLA class I molecule co-receptors.
  - Additionally, the fine-tuned scGPT model highlighted the connection between CD36 and CD14.

GRN refers to Gene Regulatory Network, a network structure describing interactions between genes. In biology, gene regulatory networks describe interaction relationships between genes, transcription factors, miRNAs, and other gene regulatory elements, and how these interactions affect gene expression. Gene regulatory networks are very important for understanding the complexity of gene regulation and cellular function regulation mechanisms, as they can reveal key factors and pathways in gene regulatory networks and help researchers understand signal transduction and cell fate determination processes in organisms.

## Summary

`scGPT` and `GenePT` have some overlap in downstream tasks and metrics. Compared to `GenePT`, `scGPT`'s tasks are more complex, many of which are seen for the first time. Below is an analysis.

Gene perturbation analysis and in silico inverse perturbation prediction are common tasks in genomics and bioinformatics, used to understand gene functions and effects in cellular states and biological processes. The following is their general process and some evaluation metrics:

### Gene Perturbation Analysis:

- **Perturbation Data Collection**: Collect datasets containing gene perturbation experimental data, such as gene editing technologies (like CRISPR) induced gene knockouts, overexpression, or mutations.
- **Data Preprocessing**: Preprocess perturbation experimental data, including data cleaning, removing noise and outliers, etc.
- **Feature Extraction**: Extract features from gene perturbation data, which can include changes in gene expression levels, changes in protein interaction networks, etc.
- **Analysis and Interpretation**: Use statistical and machine learning techniques to analyze and interpret the effects of gene perturbation on biological processes, possibly identifying key regulatory genes, pathways, or biological processes.
- **Evaluation Metrics**: Common evaluation metrics include:
  - **Number of Differentially Expressed Genes (DEGs)**: Number of genes with significantly changed expression levels after perturbation.
  - **Enrichment Analysis**: Whether gene perturbation leads to significant enrichment of certain functional pathways.
  - **Network Topology Analysis**: Whether gene perturbation affects the structure and stability of gene regulatory networks.
  - **Biological Effect Assessment**: For specific biological processes or diseases, whether gene perturbation leads to expected biological effects.

### In Silico Inverse Perturbation Prediction:

- **Modeling**: Use machine learning or deep learning methods to build models that can predict the genetic perturbation causing given gene expression patterns from gene expression data.
- **Data Preparation**: Prepare training and test sets containing gene expression data and corresponding gene perturbation information.
- **Feature Engineering**: Perform feature engineering on gene expression data, possibly including dimensionality reduction, standardization, or feature extraction.
- **Model Training**: Train the inverse perturbation prediction model using the training set, adjusting model parameters to improve prediction performance.
- **Model Evaluation**: Evaluate model performance using the test set, typically using common regression or classification metrics.
- **Evaluation Metrics**: Common evaluation metrics include:
  - **Mean Squared Error (MSE)**: Average squared difference between predicted and true values.
  - **Correlation Coefficient**: Linear correlation between predicted and true values.
  - **Accuracy, Recall, F1 Score**: For classification tasks, evaluating the model's classification accuracy for gene perturbations.

In summary, gene perturbation analysis and in silico inverse perturbation prediction are important tools for understanding gene function and effects by analyzing and modeling gene expression data, and evaluation metrics can help assess model performance and predictive capability.

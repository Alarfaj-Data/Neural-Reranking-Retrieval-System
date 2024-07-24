# Neural Reranking Retrieval System
**Master's degree project**

**University of Glasgow (2022)**

# Abstract

Information retrieval (IR) systems have benefited from the recent advancements in neural networks, which can capture the context of words and have a better semantic understanding. One of the shortcomings of neural networks in IR is the cost in terms of efficiency and resources. This can be addressed by using the neural model as a re-ranker which can increase the overall effectiveness of the system without sacrificing efficiency.
In this project, we aim to discover the effect of re-ranking a candidate list of documents using a neural network model. We will also investigate models with different document representations. We will pair these representations with relevance feedback function to improve the results. In other words, we will develop a pipeline of technologies that includes a classical retrieval model, a neural re-ranker, and a pseudo relevance feedback mechanism.
This pipeline has proven to be effective as it outperformed the use of a single classical ranker, the addition of relevance feedback got us significantly better results.



# Project Details: 

+ **Dataset:** MS MARCO document dataset[^1]
+ **Classical retrieval Model:** BM25[^2]
+ **1st Neural Model:** BERT[^3]
+ **2nd Neural Model:** Longformer[^4]
+ **Pseudo Relevance Feedback Model (For Query Expansion):** Bo1 from DFR[^5]
+ **Main Cloud Service:** Google Colab
+ **Model training Cloud Service:** Lambda Labs[^6] (Google Colab was not suitable for finetuning models to large datasets)

# Research Questions

**• Does using the pipeline approach improve effectiveness when compared to classical ranking model?**

**• Does the pseudo relevance feedback improve the effectiveness?**

**• Is just re-ranking using the new expanded query better than retrieving a new candidate list?**

**• What effect does the number of tokens have in Longformer?**

**• Does using full document representation instead of just the first passage improve the results?**

Note: I answered these questions in the results section below.

# Data Assets
**• [Finetuned BERT](https://archive.org/download/finetuned-bert/Finetuned%20BERT.zip)** Model weights that can be loaded using Huggingface

**• [Finetuned Longformer](https://archive.org/download/finetuned-longformer/Finetuned%20Longformer.zip)** Model weights that can be loaded using Huggingface

**• [Preprocessed Training dataset](https://archive.org/download/train_dataset/train_dataset.rar)** In CSV Format

**• [Preprocessed Validation dataset](https://archive.org/download/val_dataset/val_dataset.rar)** In CSV Format

**• [Tokenized Training dataset (BERT)](https://archive.org/download/bert-train-dataset-tensor/BERT_train_dataset_tensor.rar)** In TensorDataset Format (.pt)

**• [Tokenized Validation dataset (BERT)](https://archive.org/download/bert-val-dataset-tensor/BERT_val_dataset_tensor.rar)** In TensorDataset Format (.pt)

**• [Tokenized Training dataset (Longformer)](https://archive.org/download/longformer-train-dataset-tensor/Longformer_train_dataset_tensor.rar)** In TensorDataset Format (.pt)

**• [Tokenized Validation dataset (Longformer)](https://archive.org/download/longformer-val-dataset-tensor/Longformer_val_dataset_tensor.rar)** In TensorDataset Format (.pt)



# Project structure
This project was broken down to 6 files for simplicity.

**1- Passage Collection Notebook**

In this notebook, I created another document collection composed of the first 500 tokens of each document, I called this <b>passage collection</b>. The reason behind this is BERT number of tokens limitation, because of this limitation BERT cannot accept more than 512 tokens. So, if we expand the query with tokens that appear in later passages in the document BERT may not benefit from them and it may hurt the results. (This idea was completely experimental)


**2- Indexing Component Notebook**

In this notebook, I created the inverted indices to be used by the classical retrieval model (BM25), and the pseudo relevance feedback model (Bo1). Typically, you need to create one inverted index for the document collection, but I decided to create 2 inverted indices in this project. One index is created using the document collection (to be used by BM25), the other index is created using the passage collection in each document (to be used by Bo1).


**3- BERT Finetuning Notebook**

In this notebook, I prepared the dataset and finetuned BERT, this includes:

1- Preparing the dataset for finetuning BERT. This dataset provides a single document that is deemed relevant for each query (this is my positive label), but for a classification task, we also need documents that are not relevant to the query (this will be the negative label). To address this, I sampled a random document from the collection for each query. This way we have the provided relevant document and the random sampled document (positive and negative). By the end of the preprocessing, I created a dataframe that is suitable for my needs, it had query id, document id, query text, document text, and the label (whether this document is relevant to the query or not). The final step was to save the dataframe as CSV to be used later in the project.

2- Feeding the preprocessed dataset to BERT tokenizer and created <b>TensorDataset</b> objects, these objects are to be fed BERT as input. I downloaded the these objects as pt files. I did this because I didn't want go through the tokenization process every time I experiment with new ideas.

3- Setting the training hyperparameters and starting training, in the end I saved the finetuned model. 


**4- BERT Reranker Notebook**

In this notebook, I loaded the finetuned model and constructed the retrieval pipeline, this includes:

1- Initializing BM25, BERT, and BERT tokenizer.

2- The pipeline goes like this. BM25 >> BERT >> Relevance feedback >> BERT


**5- Longformer Finetuning Notebook**

This notebook is similar to <b> BERT Finetuning Notebook </b>, but with some modification to account for the differences between the two models.


**6- Longformer Reranker Notebook**

This notebook is similar to <b> BERT Reranker Notebook </b>, the main difference is using Longformer instead of BERT in the pipeline.


# Results

Before discussing the results, we want to clarify the abbreviations used, FI and PI specify the inverted index used for query expansion, FI stands for full index and PI stands for passage index. The letter J in FIJ and PIJ stands for just re-rank, which means instead of retrieving a new candidate list using the expanded query, we use the expanded query to re-rank the initial candidate list. Based on the results shown in the table, we will try to answer the following research questions:

**•	Does using the pipeline approach improve effectiveness when compared to classical ranking model?**

Results have shown improvement compared to BM25. BERT outperformed BM25 in all metrics with significant improvement in precision, Longformer on the other hand did not have a significant improvement in any metric but it performed better overall. This improvement is due to the neural models’ ability to build contextual representations to get better matching results. BM25 on the other hand, relies on statical analysis, which cannot capture the semantics.

**•	Does pseudo relevance feedback improve effectiveness?**

According to the results, pseudo relevance feedback using the full index is not improving the effectiveness of BERT’s pipeline, in fact it got surprisingly worse. We were surprised because we assumed that PRF would improve the effectiveness of BERT like it is improving the effectiveness of classical models, but that assumption was incorrect. After investigation we produced two explanations for this; the first is the number of tokens that BERT can handle compared to the length of the document. What if the added terms from PRF do not come from the first passage? That would confuse BERT because these terms do not appear in the window that BERT can see. This was the reason we built a second inverted index that covers only the first passage of every document, while there is no significant improvement compared to BM25, it is performing better than PRF with full index. The second explanation is about the way that PRF works. PRF adds terms to the end of the original query ordered by score which may look random, while this has no effect on classical models, BERT and contextualized neural models expect a complete sentence that have meaning. To show an example, this is a query from the test dataset “what slows down the flow of blood” and this is the expanded query “what slows down the flow of blood flow slow blood narrow atherosclerosi doctor heart occur increas vessel”. The expanded query is clearly not a complete sentence, and words are added in a random order. We will try to investigate this issue further in future research. Longformer also did not perform well with PRF, results from full index were not reported in the table but it performed worse. When we used the passage index the results got better as shown in the table except for MRR.

**•	Is just re-ranking using the new expanded query better than retrieving a new candidate list?**

Results has shown that just re-ranking was better in in both BERT and Longformer. The reason behind that is we are not introducing new documents in the case of just re-rank, we are just re-ranking the list that was already retrieved, which is the same list that was used to expand the query. On the other, retrieving a new list means that BM25 will retrieve new documents based on the expanded query, those documents may have the elite (distinguishing) terms in later passages which BERT cannot see.

**•	What effect does the number of tokens have in Longformer?**

Results suggest that there is a point where increasing the number of tokens does not improve the results, using 2048 or even 4096 tokens did not improve the results when compared to 1024 tokens, this may be specific to this dataset, this can be investigated further in future research.

**•	Does using full document representation instead of just the first passage improve effectiveness?**

Based on the results, we do not see any benefit from using full document representations when compared to using first passage representation, the reason behind this can be that the elite terms are usually found in the introduction or the abstract which is usually the first passage of the document.

<table class="first-row first-col no-vband docx_tablegrid" style="width: 100%; margin-left: auto; margin-right: auto;"><colgroup><col style="width: 97.35pt;"><col style="width: 57.25pt;"><col style="width: 48.6pt;"><col style="width: 48.6pt;"><col style="width: 67.7pt;"><col style="width: 61.65pt;"><col style="width: 70.15pt;"></colgroup><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-left: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>Model</span></p></td><td style="width: 13.1%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>Number of tokens</span></p></td><td style="width: 10.68%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>P@10</span></p></td><td style="width: 10.68%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>R@100</span></p></td><td style="width: 14.78%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>MAP@100</span></p></td><td style="width: 13.5%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>MRR@10</span></p></td><td style="width: 15.32%; border-bottom: 0.5pt solid black; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>NDCG@10</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-left: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25</span></p></td><td style="width: 13.1%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>N/A</span></p></td><td style="width: 10.68%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.6116</span></p></td><td style="width: 10.68%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.393</span><span>7</span></p></td><td style="width: 14.78%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.2461</span></p></td><td style="width: 13.5%; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.872</span><span>2</span></p></td><td style="width: 15.32%; border-bottom: 0.5pt solid black; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.5400</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + BERT</span></p></td><td style="width: 13.1%; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>512</span></p></td><td style="width: 10.68%; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold; "><b>0.6814</b></span></p></td><td style="width: 10.68%; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.393</span><span>9</span></p></td><td style="width: 14.78%; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.260</span><span>7</span></p></td><td style="width: 13.5%; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.913</span><span>8</span></p></td><td style="width: 15.32%; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold;"><b>0.5858</b></span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + BERT + PRF (FI)</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>512</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.6488</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.4002</span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.2547</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.8236</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.542</span><span>3</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + BERT + PRF (PI)</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>512</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.6698</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.4270</span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.271</span><span>9</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.8527</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.575</span><span>9</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + BERT + PRF (PIJ)</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>512</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold;"><b>0.6884</b></span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold;"><b>0.4142</b></span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.265</span><span>5</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: 0.5pt solid black; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.856</span><span>1</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: 0.5pt solid black; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold;"><b>0.5909</b></span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: 0.5pt solid black; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + Longformer</span></p></td><td style="width: 13.1%; border-top: 0.5pt solid black; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>512</span></p></td><td style="width: 10.68%; border-top: 0.5pt solid black; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.6581</span></p></td><td style="width: 10.68%; border-top: 0.5pt solid black; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.3892</span></p></td><td style="width: 14.78%; border-top: 0.5pt solid black; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.2427</span></p></td><td style="width: 13.5%; border-top: 0.5pt solid black; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.8064</span></p></td><td style="width: 15.32%; border-top: 0.5pt solid black; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.5452</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + Longformer</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>1024</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.662</span><span>8</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.406</span><span>1</span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.259</span><span>6</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.891</span><span>7</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.573</span><span>7</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + Longformer</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>2048</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.662</span><span>8</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.4053</span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.258</span><span>8</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.9033</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.5748</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + Longformer</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>4096</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.662</span><span>8</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.4053</span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.258</span><span>8</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.9033</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.5748</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + Longformer + PRF (PI)</span></p></td><td style="width: 13.1%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>1024</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.6465</span></p></td><td style="width: 10.68%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold;"><b>0.4306</b></span></p></td><td style="width: 14.78%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.2677</span></p></td><td style="width: 13.5%; border-top: medium; border-bottom: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.7985</span></p></td><td style="width: 15.32%; border-top: medium; border-bottom: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.5524</span></p></td></tr><tr style="height: 8.5pt; text-align: center;"><td style="width: 21.98%; border-top: medium; border-left: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>BM25 + Longformer + PRF (PIJ)</span></p></td><td style="width: 13.1%; border-top: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>1024</span></p></td><td style="width: 10.68%; border-top: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.6767</span></p></td><td style="width: 10.68%; border-top: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span style="font-weight: bold;"><b>0.4181</b></span></p></td><td style="width: 14.78%; border-top: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.2670</span></p></td><td style="width: 13.5%; border-top: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.8521</span></p></td><td style="width: 15.32%; border-top: medium; border-right: medium; vertical-align: middle;"><p style="margin-top: 0pt; text-align: center;"><span>0.5762</span></p></td></tr></table><p class="docx_caption"><span style="font-weight: normal;">Results</span><span style="font-weight: normal;">, bold marks significant improvement p-value &lt;0.05 </span></p><p></p></article></section></div>


# Notes

<ul>
<li> Please refer to the dissertation document which includes more details.</li>
<li> Contact if you have any questions or corrections  <a href="https://linkedin.com/in/abdullahalarfaj" target="blank"><img src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="abdullahalarfaj" height="15" width="15" /></a></li>
</ul>


# References

[^1]:N. Craswell, B. Mitra, E. Yilmaz, D. Campos, and E. M. Voorhees, ‘Overview of the TREC 2019 deep learning track’, Mar. 18, 2020, arXiv: arXiv:2003.07820. Accessed: Aug. 14, 2022. [Online]. Available: http://arxiv.org/abs/2003.07820
[^2]:S. Robertson, S. Walker, S. Jones, M. M. Hancock-Beaulieu, and M. Gatford, ‘Okapi at TREC-3’, in Overview of the Third Text REtrieval Conference (TREC-3), Gaithersburg, MD: NIST, Jan. 1995, pp. 109–126. [Online]. Available: https://www.microsoft.com/en-us/research/publication/okapi-at-trec-3/
[^3]: J. Devlin, M.-W. Chang, K. Lee, and K. Toutanova, “BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding.” arXiv, May 24, 2019. Accessed: Aug. 14, 2022. [Online]. Available: http://arxiv.org/abs/1810.04805
[^4]: I. Beltagy, M. E. Peters, and A. Cohan, “Longformer: The Long-Document Transformer.” arXiv, Dec. 02, 2020. Accessed: Aug. 14, 2022. [Online]. Available: http://arxiv.org/abs/2004.05150
[^5]: G. Amati, “Probability models for information retrieval based on divergence from randomness,” Thesis (PhD), University of Glasgow, 2003. [Online]. Available: https://theses.gla.ac.uk/1570/
[^6]: [Lambdalabs](https://lambdalabs.com/)

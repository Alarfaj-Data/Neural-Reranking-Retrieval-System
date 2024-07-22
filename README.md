# Neural Reranking Retrieval System
Master's degree project (2022)

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
+ **Training Cloud Service:** Lambda Labs[^6]


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


# Notes

<ul>
<li> I was not planning to publish the codes when I wrote them, forgive me if they look messy.</li>
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

---
title: "Can we avoid repeated questions by identifying similar intent?"
description: "Inhibiting repeated questions can help avoid duplication of efforts"
layout: post
toc: true
comments: true
hide: false
search_exclude: false
image: /images/2021-10-03-quora-question-pairs/quora-logo.png
categories: [nlp]
author: "<a href='https://www.linkedin.com/in/abhijithragav/'>Abhijith Ragav</a>, <a href='https://www.linkedin.com/in/ajish-sekar/'>Ajish Sekar</a>, <a href='https://www.linkedin.com/in/naveen-1999/'>Naveen Narayanan</a>, <a href='https://www.linkedin.com/in/pranav-guruprasad-82697514a/'>Pranav Guruprasad</a>, <a href='https://www.linkedin.com/in/yash-jakhotiya/'>Yash Jakhotiya</a>"
---

_Course project for [CS 7641](https://mahdi-roozbahani.github.io/CS46417641-fall2021/)_

# Introduction/Background :

With public discussion forums becoming highly popular in recent times, especially with classes and work moving to an online setup, websites like Edstem, Piazza, Stack Overflow and Quora have a constant influx of a large volume of questions. Multiple questions with the same intent leads to redundancy, and waste of time, space and resources.

# Problem Definition :

The aim is to identify and flag questions with a high similarity index, and retain only canonical questions in order to make the work of administrative staff such as TAs/Professors easier in terms of answering repeated questions or questions of the same intent.

# Methods:

1. Dataset: The dataset that we will be using is the Quora Question Pairs dataset<sup>[1]</sup>. It consists of 404,290 pairs of questions. Each datapoint consists of a pairof questions and whether
or not they are similar.


2. Data preprocessing:
    - One augmentation method we plan to leverage is the transitive property of similarity and create more question pairs. Assuming a question is represented as 𝑄<sub>𝑖</sub>. If 𝑄<sub>1</sub> - 𝑄<sub>2</sub> are similar and 𝑄<sub>2</sub> - 𝑄<sub>3</sub> are similar, then 𝑄<sub>1</sub> - 𝑄<sub>3</sub> will also be similar.
    - Other preprocessing techniques we intend to use are,
        1. Noise Removal
        2. Removing stopwords and punctuation
        3. Tokenization
        4. Lemmatization and stemming


3. Training:
    - For each training iteration, we input questions in a pairwise fashion - 𝑄<sub>𝑖</sub>, 𝑄<sub>𝑗</sub>.
    - The model learns representations of building blocks of both sentences𝑄<sub>𝑖</sub>−>ν<sub>𝑖</sub>, 𝑄<sub>𝑗</sub>−>ν<sub>𝑗</sub>.
    - Next, these representations are concatenated 𝑣<sub>𝑐𝑜𝑛</sub> = 𝑐𝑜𝑛𝑐𝑎𝑡(ν<sub>𝑖</sub>, ν<sub>𝑗</sub>), and passed on to a feedforward neural network or a machine learning model 𝐹 (𝑣𝑐𝑜𝑛) that predicts whether two questions are similar or not.
    - ∀ questions 𝑄<sub>𝑖</sub> ε question bank 𝑄<sub>𝐵</sub>, we group them into clusters 𝑐<sub>_1_</sub>,...,𝑐<sub>_n_</sub> for efficient inference.


4. Inference:
    - For a query question 𝑄<sub>𝑞</sub> we identify the cluster 𝑐<sub>𝑖</sub> it belongs to.
    - For all candidate questions 𝑄<sub>𝑑</sub> belonging to cluster 𝑐<sub>𝑖</sub>, we find similarity 𝑠𝑖𝑚(𝑄<sub>𝑑</sub>, 𝑄<sub>𝑞</sub>). With clustering we avoid finding similarities with all questions in the question bank, making inference efficient.
    - If for any 𝑄<sub>𝑑</sub> ε 𝑐<sub>𝑖</sub>, if 𝑠𝑖𝑚(𝑄<sub>𝑑</sub>, 𝑄<sub>𝑞</sub>) > 𝑡ℎ𝑟𝑒𝑠ℎ𝑜𝑙𝑑, we flag that question as similar.
    - We also output 𝑡𝑜𝑝 − 𝑘 similar questions based on 𝑠𝑖𝑚(𝑄<sub>𝑑</sub>, 𝑄<sub>𝑞</sub>).


5. Models in consideration:
    - Potential models for learning representations
        - BERT-like models (RoBERTa, ALBERT, DeBERTa) 
        - GPT models
        - XLNet
    - Potential models for clustering
        - K-means
        - DBSCAN
        - GMM
    - Potential models for finding similarity
        - Feedforward neural network
        - Gradient boosted trees

![](https://yashjakhotiya.github.io/blog/images/2021-10-03-quora-question-pairs/flow_chart.png "Model in action") 



# Potential Results and Discussions:
We aim to confidently identify a representative question for a
query question input by a user, and further list the **_top-k_** most
relevant questions.

We also hope to gain insight by clustering similar questions and
analyze the reason for their similarity, which may help in
downstream tasks such as automatic question tagging, and
personalized recommendation of questions basedon the field of
interest.

# Proposed Timeline and Responsibilities :
![](https://yashjakhotiya.github.io/blog/images/2021-10-03-quora-question-pairs/timeline.png "Timeline") 


# References :
1. https://quoradata.quora.com/First-Quora-Dataset-Release-Question-Pairs
2. D. A. Prabowo and G. Budi Herwanto, "Duplicate Question Detection in Question Answer Website using Convolutional Neural Network," 2019 5th International Conference on Science and Technology (ICST), 2019, pp. 1-6, doi: 10.1109/ICST47872.2019.9166343.
3. Jacob Devlin, Ming-Wei Chang, Kenton Lee, Kristina
Toutanova, “BERT: Pre-training of Deep Bidirectional
Transformers for Language Understanding”, arXiv:1810.
4. Tianqi Chen andCarlos Guestrin.2016.XGBoost: A Scalable Tree Boosting System. Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining (KDD '16). Association for Computing Machinery, New York,NY,USA,785–794.DOI:https://doi.org/10.1145/2939672.785.
5. Reynolds D.(2009)GaussianMixture Models. In:Li S.Z.,Jain A. (eds) Encyclopedia of Biometrics. Springer, Boston, MA. https://doi.org/10.1007/978-0-387-73003-5_
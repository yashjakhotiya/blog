---
title: "Can we avoid repeated questions by identifying similar intent?"
description: "Inhibiting repeated questions can help avoid duplication of efforts"
layout: post
toc: true
comments: true
hide: false
search_exclude: false
image: /images/2021-10-03-quora-question-pairs/similar-questions-flow-chart.png
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

![WordCloud](https://user-images.githubusercontent.com/22400185/141877000-0663d2d9-2a99-4f59-b7ee-1cd367323457.png)
_Fig 1: Word Cloud depicting which words appear more frequently in the dataset_

![Histogram of character count](https://user-images.githubusercontent.com/22400185/141876903-478cd635-ae3a-45c8-a491-7f71b0c14011.jpeg)
_Fig 2: Histogram representing the number of characters in each question_

![Histogram of word count](https://user-images.githubusercontent.com/22400185/141876911-18ff1378-cb7a-411c-b82b-f13c571eddd2.jpeg)
_Fig 3: Histogram representing the number of words in each question_

![Word share](https://user-images.githubusercontent.com/22400185/141876917-4d315806-63bb-4a46-a7a4-c55c6acff8ac.jpeg)

_Fig 4: Plots representing the distribution of ratio of words shared between similar and dissimlar question_

2. Data Augmentation:
    - We exploit the transitive property of similarity to generate new datapoints.
       - If Question Q<sub>1</sub> is similar to Question Q<sub>2</sub> and Question Q<sub>2</sub> is similar to Question Q<sub>3</sub>, then we can infer that Q<sub>1</sub> is similar to Q<sub>3</sub>
       - If Question Q<sub>1</sub> is similar to Question Q<sub>2</sub> and Question Q<sub>2</sub> is not similar to Question Q<sub>3</sub>, then we can infer that Q<sub>1</sub> is not similar to Q<sub>3</sub>
     
     Through this method, we generated 25% more training datapoints.
3. Data preprocessing:
    - We used the following preprocessing techniques:
        -   Sample Sentence: "How can internet speed be increased by hacking through DNS?"
        1. Removing punctuation
            - After Punctuation Removal: "How can internet speed be increased by hacking through DNS"
        2. Converting characters to Lower Case
            - After conversion: "how can internet speed be increased by hacking through dns"
        3. Tokenization
            - Tokenization is the process of converting the sentence into a list of it's constituent words/phrases. It helps reduce the input to a finite set, making it easier to process.
            - After Tokenization: ['how', 'can', 'internet', 'speed', 'be', 'increased', 'by', 'hacking', 'through', 'dns']
        4. Stop Word Removal
            - The idea of Stop Word Removal is to remove commonly occuring words in the corpus.
            - After Stop Word Removal: ['internet', 'speed', 'increased', 'hacking', 'dns']
        5. POS Tagging
            - POS tagging refers to categorizing words in a corpus corresponding to a particular part of speech (eg: Noun, Verb, Adjective etc). This process helps improve the performance of the Lemmatizer.
            - After POS Tagging: [('internet', 'a'), ('speed', 'n'), ('increased', 'v'), ('hacking', 'v'), ('dns', 'n')]
        6. Lemmatization
            - Lemmatization is grouping together different inflected forms of a word so that they can be analyzed as a single item.
            - After Lemmatization: ['internet', 'speed', 'increase', 'hack', 'dns']

4. Training:
    - For each training iteration, we input questions in a pairwise fashion - 𝑄<sub>𝑖</sub>, 𝑄<sub>𝑗</sub>.
    - The model learns representations of building blocks of both sentences𝑄<sub>𝑖</sub>−>ν<sub>𝑖</sub>, 𝑄<sub>𝑗</sub>−>ν<sub>𝑗</sub>.
    - Next, these representations are concatenated 𝑣<sub>𝑐𝑜𝑛</sub> = 𝑐𝑜𝑛𝑐𝑎𝑡(ν<sub>𝑖</sub>, ν<sub>𝑗</sub>), and passed on to a feedforward neural network or a machine learning model 𝐹 (𝑣𝑐𝑜𝑛) that predicts whether two questions are similar or not.
    - ∀ questions 𝑄<sub>𝑖</sub> ε question bank 𝑄<sub>𝐵</sub>, we group them into clusters 𝑐<sub>_1_</sub>,...,𝑐<sub>_n_</sub> for efficient inference.

    **Supervised method: Classifying sentences as similar or not using  BERT-based model (Added for midpoint report)**
    - The tokenized version of the candidate sentence that we get from the preprocessing stage is first passed to a pre-trained BERT model.
    - The output of this pre-trained BERT model is the sentence-level embedding of the tokenized version - E<sub>1.
    - The same process is done on the second candidate sentence to retrieve its sentence-level embedding - E<sub>2.
    - The two embeddings are concatenated, and then passed to a feed-forward neural network which consists of a fully connected layer, a ReLu layer, another fully       connected layer, and finally a softmax layer in the order specified.
    - The softmax layer then outputs the probabilities of the pair of candidate questions being similar. Based on the probabilities we classify the questions as         similar or not similar.
    
    ![image](https://user-images.githubusercontent.com/39705529/141838152-b27025fc-becc-47b6-a20f-f6c7bb41e548.png)
    
     _Fig 5: BERT takes in sentences as inputs and gives sentence-level embeddings as the output_
    

5. Inference:
    - For a query question 𝑄<sub>𝑞</sub> we identify the cluster 𝑐<sub>𝑖</sub> it belongs to.
    - For all candidate questions 𝑄<sub>𝑑</sub> belonging to cluster 𝑐<sub>𝑖</sub>, we find similarity 𝑠𝑖𝑚(𝑄<sub>𝑑</sub>, 𝑄<sub>𝑞</sub>). With clustering we avoid finding similarities with all questions in the question bank, making inference efficient.
    - If for any 𝑄<sub>𝑑</sub> ε 𝑐<sub>𝑖</sub>, if 𝑠𝑖𝑚(𝑄<sub>𝑑</sub>, 𝑄<sub>𝑞</sub>) > 𝑡ℎ𝑟𝑒𝑠ℎ𝑜𝑙𝑑, we flag that question as similar.
    - We also output 𝑡𝑜𝑝 − 𝑘 similar questions based on 𝑠𝑖𝑚(𝑄<sub>𝑑</sub>, 𝑄<sub>𝑞</sub>).


6. Models in consideration:
    - Potential models for learning representations
        - BERT-like models (RoBERTa, ALBERT, DeBERTa) 
        - GPT models
        - XLNet
    - Potential models for clustering
        - K-means
        - DBSCAN
        - GMM
    - Potential models for finding similarity
        - Feedforward neural network (FFN)
        - Gradient boosted trees

![](https://yashjakhotiya.github.io/blog//images/2021-10-03-quora-question-pairs/similar-questions-flow-chart.png "Model in action") 



# Potential Results and Discussions:
We aim to confidently identify a representative question for a
query question input by a user, and further list the **_top-k_** most
relevant questions.

# Midpoint results and analysis

For this checkpoint, we completed the supervised aspect of our project and implemented a BERT-based model to classify pairs of questions as "similar" or "not similar". We conducted thorough experiments and ablation studies to find the optimal model, hyperparameters and data preprocessing & augmentation techniques that gives us the best results.
 
**Experiment 1 : Data preprocessing and augmentation choices**
    
Through the first set of experiments, we try to observe the effects of data preprocessing and data augmentation on the performance of the model. We also apply three different types of models on a sample dataset of 1000 data points from the entire original dataset. The results of these are seen in the tables below:

| Data Preprocessing  | Data Augmentation  | Model | Train accuracy | Train F1 score | Test accuracy | Test F1 score |
|---|---|---|---|---|---|---|
| No  |  No |  Baseline | 0.57  | 0.61  | 0.42  | 0.58  |
| Yes  | No  | Baseline | 0.61  | 0.61  | 0.57  | 0.62 |
| Yes  | Yes  | Baseline| 0.6  |  0.62 |  0.61 |  0.6 |


| Data Preprocessing  | Data Augmentation  | Model | Train accuracy | Train F1 score | Test accuracy | Test F1 score |
|---|---|---|---|---|---|---|
| No  |  No |  Static BERT |  0.66  | 0.61  | 0.57  | 0.53  |
| Yes  | No  | Static BERT | 0.67  |0.62  | 0.59  | 0.57 |
| Yes  | Yes  | Static BERT| 0.66  |  0.62 |  0.59 |  0.55 |
    
| Data Preprocessing  | Data Augmentation  | Model | Train accuracy | Train F1 score | Test accuracy | Test F1 score |
|---|---|---|---|---|---|---|
| No  |  No |  Fine-tuned BERT |  0.98  | 0.97  | 0.57  | 0.41  |
| Yes  | No  | Fine-tuned BERT | 0.97  |0.97  | 0.58  | 0.5 |
| Yes  | Yes  | Fine-tuned BERT| 0.97  |  0.96 |  0.64 |  0.52 |
    
Here, the baseline model is nothing but the pre-trained BERT-based model, and the embeddings of the sentences obtained from this model are compared using a cosine similarity. This provided us a good baseline result and starting point, which we tried to improve upon.

For the other two models, the dataset was then split into training and testing data with a ratio of 80:20 respectively. The models were trained over this data for 5 epochs with a learning rate of 0.0001 and batch size of 8. The architecture used for these models in this experiment : ***BERT -> FFN -> Sigmoid layer -> FFN -> Softmax layer***. In the Static BERT model the pre-trained BERT model is kept static and the remaining layers are trained, whereas in the Fine-tuned BERT, all the layers are trained and the pre-trained BERT-based model is fine-tuned on our dataset.

**Inference** : As we can see from the table, applying data preprocessing techniques (mentioned in section 2) and data augmentation techniques increases the accuracy on the test data. This empirically backs our decision to do data preprocessing and augmentation.

**Experiment 2 : Model selection**

Through the second set of experiments, we tried to gain insight on which model would work best for our problem. As a part of this, we ran our baseline, Static BERT, and Fine-tuned BERT on a sample dataset of 10000 datapoints randomly chosen from the original dataset. For the Static BERT and Fine-tuned BERT models we again split the data into train and test with a 80:20 ratio, and ran them for 10 epochs with a learning rate of 0.0001 and batch size 32. The architecture used for these 2 models in this experiment : ***BERT -> FFN -> PRelu -> FFN -> PRelu -> FFN -> Softmax Layer***. The results are seen in the table below:

| Data Preprocessing  | Data Augmentation  | Model | Train accuracy | Train F1 score | Test accuracy | Test F1 score |
|---|---|---|---|---|---|---|
| Yes  |  Yes |  Baseline |  0.59  | 0.6  | 0.6  | 0.61  |
| Yes  | Yes  | Static BERT | 0.7  |0.63  | 0.68  | 0.61 |
| Yes  | Yes  | Fine-tuned BERT| 0.99  |  0.98 |  0.71 |  0.62 |

**Inference** : As it can be seen in the table, the Fine-tuned BERT model produces the best accuracy results on the data as compared to the baseline and Static-BERT. This again empirically backs our decision to choose a pre-trained BERT model and fine-tune it on our dataset. 

**Experiment 3 : Final model**
    
Once we chose the model as the Fine-tuned BERT model based on the results from experiment 2, we did some hyperparamter tuning to pick the best combination of hyperparameters for performance. We found that a batch size of 128 and a training over 5 epochs gave the best results for our Fine-tuned BERT model. For this experiment we used 100,000 data points from the original dataset. The results for this experiment are shown below:

| Data Preprocessing  | Data Augmentation  | Model | Train accuracy | Train F1 score | Test accuracy | Test F1 score |
|---|---|---|---|---|---|---|
| Yes  |  Yes |  Fine-tuned BERT |  0.97  | 0.96  | 0.76  | 0.69  |

We also tried adding a dropout layer to observe if any significant overfitting was occurring. However, this did not result in any change in the results. Thus, we can conclude that our model does not overfit. 

**Future experiments:**    
    
Experiment 3 was the final set of experiments done for the supervised part of our project before the midpoint deadline.
We also hope to gain insight by clustering similar questions and
analyze the reason for their similarity, which may help in
downstream tasks such as automatic question tagging, and
personalized recommendation of questions based on the field of
interest. This will be the unsupervised portion of our project.

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

---
title: "On Continual Learning"
description: "This is work in progress"
layout: post
toc: true
comments: true
hide: false
search_exclude: false
image: /images/2020-12-20-container-runtimes/runc_in_docker.png
categories: [continual learning, lifelong learning]
author: "<a href='https://www.linkedin.com/in/yash-jakhotiya/'>Yash Jakhotiya</a>"
---

# Why care? The problem and my personal encounter with it
In a past internship in a large company's private cloud department, I was tasked to train a model that would predict which on-call engineer an infrastuctural alert should go to, with features that ranged from time of the day to the alert's natural language description. Fair enough. There goes two months of feature engineering, adapting and evaluating state-of-the-art models, hyperparameter tuning, finalising a two-stage network and deployment, and we have a module that performs acceptably well. All stakeholders are happy. Pushed to prod! 

Does the story end there? Nope. You see, a large dynamic private cloud has new alerts added every week. Can we feed in the new alerts and expect our module to spit something out? Yes. Can we assume that our on-call engineer roles, our classes, will always be the same? Probably not. Can we assume these new alerts are from an identical distribution as the old ones? Definitely not. (You'd be surprised at the number of new and innovative problems that arise in a complex cloud environment!)

How do you make sure in this case, that your network learns to correctly classify these new alerts? A naive strategy would be to retrain the model on the whole (old + new) alerts data. This might work if -
i. the network training time is bearable
ii. the addition of new data doesn't quickly result in a storage problem, especially if new data is available as a regular stream
iii. you'll always have access to old data without any legal or privacy concerns.

However, more of than you'd expect, one of the above assumptions gets violated and you are forced to think how to go about including newer data in your model. Another naive strategy would be to fine-tune the old model on newer data, and expect it to perform well on everything. In this case, you are in for another suprise - of how quickly your model learns to forget older data. Eventually, this leads us to ...

# The problem of catastrophic forgetting
As early as in 1989, McCloskey and Cohen[3] identified a problem - when trained sequentially on new data, classes and tasks, a neural network's performance degrades at previously learned concepts. They named this 'catastrophic interference', now commonly known as catastrophic forgetting. This usually happens when a new task overrides previously learned weights, interfering with performance on past tasks. Abraham and Robins[4] referred to this as the 'stability-plasticity dilemma', where a model too _stable_ won't be able to learn new tasks and a model too _plastic_ will have it's weights too changed to forget older ones. The research that goes into addressing catastrophic forgetting


# References
[1]: @article{DBLP:journals/corr/abs-2007-00487,
  author    = {Timoth{\'{e}}e Lesort},
  title     = {Continual Learning: Tackling Catastrophic Forgetting in Deep Neural
               Networks with Replay Processes},
  journal   = {CoRR},
  volume    = {abs/2007.00487},
  year      = {2020},
  url       = {https://arxiv.org/abs/2007.00487},
  eprinttype = {arXiv},
  eprint    = {2007.00487},
  timestamp = {Mon, 06 Jul 2020 15:26:01 +0200},
  biburl    = {https://dblp.org/rec/journals/corr/abs-2007-00487.bib},
  bibsource = {dblp computer science bibliography, https://dblp.org}
}

[2]: @book{10.5555/3276462,
author = {Chen, Zhiyuan and Liu, Bing and Brachman, Ronald and Stone, Peter and Rossi, Francesca},
title = {Lifelong Machine Learning},
year = {2018},
isbn = {1681733021},
publisher = {Morgan &amp; Claypool Publishers},
edition = {2nd},
abstract = {Lifelong Machine Learning, Second Edition is an introduction to an advanced machine
learning paradigm that continuously learns by accumulating past knowledge that it
then uses in future learning and problem solving. In contrast, the current dominant
machine learning paradigm learns in isolation: given a training dataset, it runs a
machine learning algorithm on the dataset to produce a model that is then used in
its intended application. It makes no attempt to retain the learned knowledge and
use it in subsequent learning. Unlike this isolated system, humans learn effectively
with only a few examples precisely because our learning is very knowledge-driven:
the knowledge learned in the past helps us learn new things with little data or effort.
Lifelong learning aims to emulate this capability, because without it, an AI system
cannot be considered truly intelligent. Research in lifelong learning has developed
significantly in the relatively short time since the first edition of this book was
published. The purpose of this second edition is to expand the definition of lifelong
learning, update the content of several chapters, and add a new chapter about continual
learning in deep neural networkswhich has been actively researched over the past two
or three years. A few chapters have also been reorganized to make each of them more
coherent for the reader. Moreover, the authors want to propose a unified framework
for the research area. Currently, there are several research topics in machine learning
that are closely related to lifelong learningmost notably, multi-task learning, transfer
learning, and meta-learningbecause they also employ the idea of knowledge sharing
and transfer. This book brings all these topics under one roof and discusses their
similarities and differences. Its goal is to introduce this emerging machine learning
paradigm and present a comprehensive survey and review of the important research results
and latest ideas in the area. This book is thus suitable for students, researchers,
and practitioners who are interested in machine learning, data mining, natural language
processing, or pattern recognition. Lecturers can readily use the book for courses
in any of these related fields.}
}

[3]: McCloskey, M. and Cohen, N. J. (1989). Catastrophic interference in connectionist networks: The
sequential learning problem. In Psychology of learning and motivation, volume 24, pages 109–165.
Elsevier.

[4]: Abraham WC, Robins A. Memory retention--the synaptic stability versus plasticity dilemma. Trends Neurosci. 2005;28(2):73-78. doi:10.1016/j.tins.2004.12.003
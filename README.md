## Abstract

The goal of entity resolution is to determine whether two
entities are the same (e.g. whether George Smith and Smith George refers to the
same person). A fundamental issue is to derive an appropriate similarity
predicate (a.k.a similarity function). In the last few decades, many similarity
predicates are exploited, such as Jaccard coefficient, cosine similarity, BM25,
language model, etc. While we agreed that all existing similarity predicates are
well defined and effective, we observed that none of them can consistently
outperform the others in all situations. Choosing which similarity predicate to
use is usually being treated as an empirical question by evaluating a particular
entity resolution task using a number of different similarity predicates. This is
not efficient and the result obtained is not portable. As a result, we are
curious about whether we can combine all similarity predicates together to form a
committee so that we do not need to worry about choosing which of them to use,
meanwhile we can obtain an even better result. In this paper, we focus on this
issue and propose a generic framework to formulate a similarity predicate
committee. To the best of our knowledge, this is the first piece of work about
similarity predicate committee. However, formulating an effective similarity
predicate committee is not trivial. We need to model at least two elements,
namely, confidences of similarity predicates and reliabilities of attributes. In
this paper, we explain how they affect the effectiveness of similarity predicate
committee and how we model them on the fly dynamically. We apply our proposed
work to solve some entity resolution problems. We report all our findings in this
paper. It is worth to note that the proposed framework is
unsupervised.


## Introduction

Entity resolution is also known as record linkage (O.Benjelloun_TR05),
object identification or name disambiguation (S.Tejada_KDD02), data cleaning
(H.Do_VLDB02), approximate joins (S.Guha_VLDB04} and fuzzy matching
(R.Ananthakrishna_VLDB02). A typical entity resolution problem is to
identify records that belong to the same entity (e.g. whether George Smith and
George Smath refers to the same person) (W.E.Winkler_TR06). Entity
resolution is especially important in this Internet age because numerous online
applications require entity resolution, such as name disambiguation in digital
libraries, searching a person in a social network (such as facebook), etc. Many
different techniques can be found in the existing work to tackle this problem
(e.g.
(.E.Winkler_TR06,Gusfield_CUP97,Jaro_JASA84,Winkler_USBC99,Sarawagi_Kirpal_SIGMOD04,Gravano_Ipeirotis_VLDB01,Chandel_Hassanzadeh_SIGMOD07).
Regardless of which techniques we use, a fundamental issue is to define an
appropriate similarity predicate (a.k.a. similarity function) to measure the
similarities among records.



Since (rnandez_Stolfo_SIGMOD95)introduced the entity resolution problem
into our community, many similarity predicates were exploited (e.g.
(Sarawagi_Kirpal_SIGMOD04,Gravano_Ipeirotis_VLDB01,Chandel_Hassanzadeh_SIGMOD07,Cohen_SIGMOD98)).
While we agreed that most, if not all, of the existing similarity predicates
(e.g. Jaccard coefficient, cosine similarity, BM25, language model, Hidden Markov
Model, etc) are well defined and effective, we observed that none of them can
consistently outperform the others in all situations. Each of them has their own
strengths and weaknesses. Yet, we do not have any systematic approach to help us
to choose which similarity predicate to use in practice. Choosing which of them
to use is usually being treated as an empirical question by evaluating a
particular entity resolution task using a number of different similarity
predicates or simply by the so-call ``rule of trump method''. This is not very
efficient and the result obtained is usually not portable.



In this paper, we are {\it not} trying to propose any new similarity predicate as
we do not lack of it; instead, we are trying to propose a generic framework that
can {\it combine a set of similarity predicates to form a committee}. By doing
so, we do not need to worry about choosing which of them to use. In addition, we
believe that we should be able to achieve an even better entity resolution result
by means of similarity predicate committee. The reason is that, if a decision
requires expert's judgement, then the decision made by $N$ different experts is
usually better than any one of them if their decisions are combined properly. In
a similarity predicate committee, each similarity predicate can be regarded as an
expert. Our task is to determine whether a set of records refers to the same
entity. To the best of our knowledge, this is the first piece of work about
similarity predicate committee. We try to apply it to solve the entity resolution
problem. Choosing entity resolution problem as our application domain is that it
is one of the most important problems in our community (especially in this
information overwhelming century). It is certainly possible to implement our
framework in some other domains, such as information retrieval, classification,
nearest neighbor search, etc.


In this paper, we will show that in order to formulate an effective similarity
predicate committee, we need to consider at least two elements, namely,
confidences of similarity predicates and reliabilities of attributes; otherwise,
the result obtained by a similarity predicate committee may not be as good as any
of the single similarity predicates. However, modelling these two elements is not
trivial which poses some new challenges:


### Challenge 1 (Confidences of Similarity Predicates)
One of the simplest
ways to formulate a similarity predicate committee is to aggregate the results of
all similarity predicates in the committee. For example, assume we have three
similarity predicates: Jaccard coefficient (Sarawagi_Kirpal_SIGMOD04),
cosine similarity (Cohen_SIGMOD98) and edit distance
(Gravano_Ipeirotis_VLDB01). Given a query record $q$ and a set of records
$R$, assume that we have calculated the similarities between $q$ and each record
in $R$ by using the just mentioned three similarity predicates, which are shown
in Table 1. In Table 1, the values in the column
``Overall'' are computed by averaging their corresponding three similarity
values.According to Table 1, we say that (<img src="http://chart.googleapis.com/chart?cht=tx&chl= q" style="border:none;"> is more similar to $r_2$ than
<img src="http://chart.googleapis.com/chart?cht=tx&chl= r_1" style="border:none;"> because the overall similarity between <img src="http://chart.googleapis.com/chart?cht=tx&chl= q" style="border:none;"> and <img src="http://chart.googleapis.com/chart?cht=tx&chl= r_2" style="border:none;"> is higher).

![](https://github.com/soap117/Similarity-Predicate-Committee-for-Entity-Resolution/blob/master/table1.jpg)

Despite of this approach is intuitive, it may not always be able to solve the
entity resolution problem effectively. For example, {\it if} we have some prior
knowledge that the two similarity values computed by Jaccard coefficient is {\it
far more reliable} than the others, then we {\it may want to alter our
aforementioned decision and conclude that $q$ is more similar to $r_1$ than
$r_2$}. A different conclusion is obtained. In fact, associating a confidence
value to every member's decision in a committee is a very important topic. For
instance, a research in classifier committee \cite{G.Fung_ICDM06} shows that
associating a confidence value to each classifier in the committee can
significantly improve the effectiveness of a classifier committee. Moreover,
\cite{Chandel_Hassanzadeh_SIGMOD07} also shows that different similarity
predicate has different effectiveness in handling different type of error. Hence,
assigning a confidence value to each similarity predicate is necessary.



Unfortunately, even we theoretically know which similarity predicate is more
effective in handling which type of error, we can never anticipate the types of
errors may occur and how many errors would exist in a record. Assigning
confidences to the similarity predicates becomes very difficult. For example, in
Table \ref{table:ex2}, it is difficult to tell which similarity predicate is more
reliable by simply looking at those numbers. In this paper, we try to solve
this problem by using a consensus world model \cite{J.Li_PODS09}.
### Challenge 2
### Challenge 3
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/soap117/Similarity-Predicate-Committee-for-Entity-Resolution/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

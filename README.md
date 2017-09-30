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

Entity resolution is also known as record linkage (Benjel loun et al. 2005), object identification or name disambigua tion(Tejada, Knoblock, and Minton 2002). A typical entity resolution problem is to identify records that belong to the same entity (e.g., whether George Smith and Smith George refers to the same person) (Winkler 2006). However, regard less of which techniques we use to tackle this problem, a fundamental issue is to define an appropriate similarity pred icate (a.k.a. similarity function) to measure the similarities among records.
Since (Hernandez and Stolfo 1995) introduced the entity resolution problem into our community, many similar ity predicates were exploited (e.g., (Chandel et al. 2007;Gravano et al. 2001; Sarawagi and Kirpal 2004)). While we agree that most of the existing similarity predicates are well defined and effective, we observed that none of them can consistently outperform the others in all situations (Lin et al. 2017). Choosing which of them to use is usually being treated as an empirical question by evaluating a particular entity resolution task.

![](https://github.com/soap117/Similarity-Predicate-Committee-for-Entity-Resolution/blob/master/table1.jpg)

In this paper, we are not trying to propose any new similarity predicate as we do not lack of it; instead, we are trying
to propose a generic framework that can combine a set of
similarity predicates to form a committee so that we do not
need to worry about choosing which one of them to use. We
can achieve an even better entity resolution result as the idea
is if a decision requires expert’s judgement, then the decision made by N different experts is usually better than any
one of them if their decisions are combined properly (Orrite
et al. 2008). Obviously, each similarity predicate can be regarded as an expert in our research, and they can be used to
formulate a similarity predicate committee.
One of the simplest ways to formulate a similarity predicate committee is to aggregate the results of all similarity
predicates in the committee. For example, assume we have
three similarity predicates: Jaccard coefficient (Sarawagi
and Kirpal 2004), cosine similarity (Cohen 1998) and edit
distance (Gravano et al. 2001). Given a query record q and
a set of records R, assume that we have calculated the similarities between q and each record in R by using the just
mentioned three similarity predicates, which are shown in
Table 1. In Table 1, the values in the column “Overall” are
computed by averaging their corresponding three similarity
values.1 According to Table 1, we say that q is more similar
to r2 than r1 because the overall similarity between q and r2
is higher.
Despite this approach is intuitive, it may not always be
able to solve the entity resolution problem effectively. For
example, if we have some prior knowledge that the two similarity values computed by Jaccard coefficient is far more reliable than the others, then we may want to alter our aforementioned decision and conclude that q is more similar to
r1 than r2. A different conclusion is obtained. Unfortunately, even we theoretically know which similarity predicate is more effective in handling which type of error, we can
never anticipate the types of errors may occur and how many
errors would exist in a record. Assigning confidences to the
similarity predicates becomes very difficult. For example,
in Table 1, it is difficult to tell which similarity predicate is
more reliable by simply looking at those numbers.
In addition, we also need to incorporate the confidence
of each similarity predicate in a similarity predicate committee to help us tackle the entity resolution problem more
effectively. In this paper, we show that this problem can be
modeled as a classical 0-1 integer programming optimization problem which maximizes the confidences of the ranking of records based on different similarity predicates. The
rest of the paper is organized as follows – Section formally
defines our problem; Section briefly discusses some related
work; Section presents our proposed work; Section conducts evaluation; Section concludes this paper.
##  Problem Definition
![](https://github.com/soap117/Similarity-Predicate-Committee-for-Entity-Resolution/blob/master/table2.png)

Table 2 shows the notations used in this paper. Let <img src="http://chart.googleapis.com/chart?cht=tx&chl=R" style="border:none;"> be a
relation which contains <img src="http://chart.googleapis.com/chart?cht=tx&chl=|R|" style="border:none;"> records. Let <img src="http://chart.googleapis.com/chart?cht=tx&chl=A" style="border:none;"> be a set of attributes in <img src="http://chart.googleapis.com/chart?cht=tx&chl=R" style="border:none;">. Let
<img src="http://chart.googleapis.com/chart?cht=tx&chl=r" style="border:none;"> be a record in <img src="http://chart.googleapis.com/chart?cht=tx&chl=R" style="border:none;"> and <img src="http://chart.googleapis.com/chart?cht=tx&chl=a" style="border:none;"> be an attribute in <img src="http://chart.googleapis.com/chart?cht=tx&chl=A" style="border:none;">. We use <img src="http://chart.googleapis.com/chart?cht=tx&chl=r.a" style="border:none;"> to denote the
value of attribute <img src="http://chart.googleapis.com/chart?cht=tx&chl=a" style="border:none;"> in record $r$. Let $q$ be a query record and <img src="http://chart.googleapis.com/chart?cht=tx&chl=q.a" style="border:none;"> be the
value of attribute <img src="http://chart.googleapis.com/chart?cht=tx&chl=a" style="border:none;"> in <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;">. In this paper, we assume all attributes contain
string values.



Let <img src="http://chart.googleapis.com/chart?cht=tx&chl=T_{r.a}" style="border:none;"> and <img src="http://chart.googleapis.com/chart?cht=tx&chl=T_{q.a}" style="border:none;"> be two sets of tokens in <img src="http://chart.googleapis.com/chart?cht=tx&chl=T_{r.a}" style="border:none;"> and <img src="http://chart.googleapis.com/chart?cht=tx&chl=T_{q.a}" style="border:none;">,
respectively. Unless otherwise specified, we refer to substrings of a string as
tokens in a generic sense such that they can be words, q-grams
s (Gravano et al. 2001), v-grams (Li, Wang, and Yang 2007), etc. For example,
suppose <img src="http://chart.googleapis.com/chart?cht=tx&chl={George}" style="border:none;">, then <img src="http://chart.googleapis.com/chart?cht=tx&chl=T_{r.a}=\{Geo,eor,org,rge\}" style="border:none;"> for
<img src="http://chart.googleapis.com/chart?cht=tx&chl=q=3" style="border:none;"> in q-grams.


Given a query record <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;">, our goal is to resolve the records in <img src="http://chart.googleapis.com/chart?cht=tx&chl=R" style="border:none;"> that are
relevant to <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;">. There are lots of methods in the existing literatures to solve
this problem. In general, all methods require us to compute the similarities
between <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;"> and each <img src="http://chart.googleapis.com/chart?cht=tx&chl=r\in{R}$" style="border:none;"> by using some similarity predicates. After that,
we can determine which of the records should be returned based on their
similarities with <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;">. Let <img src="http://chart.googleapis.com/chart?cht=tx&chl=sim_{f}(q,r)" style="border:none;"> be the similarity value between <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;">
and <img src="http://chart.googleapis.com/chart?cht=tx&chl=r" style="border:none;"> by using a similarity predicate <img src="http://chart.googleapis.com/chart?cht=tx&chl=f" style="border:none;">. 

Let <img src="http://chart.googleapis.com/chart?cht=tx&chl=F" style="border:none;"> be a set of all similarity
predicates. We will have a brief review of the existing similarity predicates in
Section Related. Let <img src="http://chart.googleapis.com/chart?cht=tx&chl=sim_{f}(q.a,r.a)" style="border:none;"> be the similarity value
between <img src="http://chart.googleapis.com/chart?cht=tx&chl=q.a" style="border:none;"> and <img src="http://chart.googleapis.com/chart?cht=tx&chl=r.a" style="border:none;"> that is computed by <img src="http://chart.googleapis.com/chart?cht=tx&chl=f" style="border:none;">. Since each record may have $m$
attributes, in the existing work, one of the most common ways to obtain the
overall similarity between <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;"> and <img src="http://chart.googleapis.com/chart?cht=tx&chl=r" style="border:none;"> <img src="http://chart.googleapis.com/chart?cht=tx&chl=sim_{f}(q,r)" style="border:none;"> is to average
<img src="http://chart.googleapis.com/chart?cht=tx&chl=sim_{f}(q.a,r.a),\forall{a}\in{A}" style="border:none;">, i.e. <img src="http://chart.googleapis.com/chart?cht=tx&chl=sim_{f}(q,r)=\frac{1}{|A|}\sum_{\forall{a}\in{A}}sim_{f}(q.a,r.a)" style="border:none;">. Yet, in this paper, we
will take a different approach in modeling this element. In this paper, we are
trying to solve the following problem:
### Probelm

Given a relation <img src="http://chart.googleapis.com/chart?cht=tx&chl=R" style="border:none;">, a query record <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;"> and a set of similarity predicates <img src="http://chart.googleapis.com/chart?cht=tx&chl=F" style="border:none;">,
our goal is to rank the records in $R$ according to <img src="http://chart.googleapis.com/chart?cht=tx&chl=q" style="border:none;"> by 
properly combining the decisions of the similarity predicates in <img src="http://chart.googleapis.com/chart?cht=tx&chl=F" style="border:none;">.






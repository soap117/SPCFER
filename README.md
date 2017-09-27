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
modeled as a classical 0-1 integer programming optimization problem which maximizes the confidences of the ranking of records based on different similarity predicates. The
rest of the paper is organized as follows – Section formally
defines our problem; Section briefly discusses some related
work; Section presents our proposed work; Section conducts evaluation; Section concludes this paper.
##  Problem Definition

Table \ref{table:sym} shows the notations used in this paper. Let $R$ be a
relation which contains $|R|$ records. Let $A$ be a set of attributes in $R$. Let
$r$ be a record in $R$ and $a$ be an attribute in $A$. We use $r.a$ to denote the
value of attribute $a$ in record $r$. Let $q$ be a query record and $q.a$ be the
value of attribute $a$ in $q$. In this paper, we assume all attributes contain
string values.



Let $T_{r.a}$ and $T_{q.a}$ be two sets of tokens in $r.a$ and $q.a$,
respectively. Unless otherwise specified, we refer to substrings of a string as
tokens in a generic sense such that they can be words, q-grams
\cite{Gravano_Ipeirotis_VLDB01}, v-grams \cite{Li_Wang_VlDB07}, etc. For example,
suppose $r.a = {\tt George}$, then $T_{r.a} = \{{\tt Geo, eor, org, rge}\}$ for
$q=3$ in q-grams.


Given a query record $q$, our goal is to resolve the records in $R$ that are
relevant to $q$. There are lots of methods in the existing literatures to solve
this problem. In general, all methods require us to compute the similarities
between $q$ and each $r \in R$ by using some similarity predicates. After that,
we can determine which of the records should be returned based on their
similarities with $q$. Let $sim_{f}(q, r)$ be the similarity value between $q$
and $r$ by using a similarity predicate $f$. For example, $f$ can be Jaccard
coefficient predicate \cite{Sarawagi_Kirpal_SIGMOD04}, edit distance predicate
\cite{Gravano_Ipeirotis_VLDB01}, language model predicate
\cite{Chandel_Hassanzadeh_SIGMOD07}, etc. Let $F$ be a set of all similarity
predicates. We will have a brief review of the existing similarity predicates in
Section \ref{sec:related}. Let $sim_{f}(q.a, r.a)$ be the similarity value
between $q.a$ and $r.a$ that is computed by $f$. Since each record may have $m$
attributes, in the existing work, one of the most common ways to obtain the
overall similarity between $q$ and $r$ $sim_{f}(q, r)$ is to average
$sim_{f}(q.a, r.a), \forall a \in A$, i.e. $sim_{f}(q, r) =
\frac{1}{|A|}\sum_{\forall a \in A} sim_{f}(q.a, r.a)$. Yet, in this paper, we
will take a different approach in modeling this element. In this paper, we are
trying to solve the following problem:



\begin{problem}
\label{prob}
Given a relation $R$, a query record $q$ and a set of similarity predicates $F$,
our goal is to rank the records in $R$ according to $q$ by 
properly combining the decisions of the similarity predicates in $F$.
\end{problem}



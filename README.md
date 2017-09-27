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


### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
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

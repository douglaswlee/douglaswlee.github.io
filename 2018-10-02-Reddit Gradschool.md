---
layout: post
title: Still waiting to hear...
---

The internet can be a pretty [toxic, nasty place](https://www.vice.com/en_us/article/8xxymb/here-are-reddits-whiniest-most-low-key-toxic-subreddit) (*Warning: NSFW language), especially when like-minded toxic, nasty people get together to share their toxic, nasty thoughts. Of course, seeking out virtual interaction with like-minded people is [sometimes good](https://arstechnica.com/information-technology/2017/05/these-are-the-online-communities-we-will-never-forget/). And sometimes the much-maligned "front page of the internet" can [be good and do good](https://gizmodo.com/reddit-is-helping-some-people-deal-with-their-mental-he-1825364592).

These were some of the thoughts I had as I was planning out my fourth project at Metis, which emphasized Natural Language Processing (NLP) and unsupervised learning. Going through the vast data and resources from the [Stanford Network Analysis Project (SNAP)](https://snap.stanford.edu/data/index.html), I came across [a study](https://snap.stanford.edu/data/web-RedditPizzaRequests.html) about [free pizza requests](https://snap.stanford.edu/data/web-RedditPizzaRequests.html) made on [r/Random_Acts_of_Pizza](https://www.reddit.com/r/Random_Acts_Of_Pizza/), a subreddit where users make such requests of other users. Some of these are successful while some are not, and with results of each request documented, the group conducting the study was able to build a classificaiton model to shed light on features leading to success of actually receiving a free pizza from another user.

I decided to attempt a similar project using a different, advice-driven subreddit -- [r/gradamissions](https://www.reddit.com/r/gradadmissions/), a subreddit for graduate school candidates to obtain information and share experiences about the application process. My main objectives for this project were:

* To understand which topics best describe user-submitted questions to this subreddit using Topic Modeling, and
* To explain how these topics (and characteristics of the submitted questions) contribute to incidence of “useful feedback” using a Logistic Regression model.

Here, "useful feedback" can be any collection of comments responding to a submitted question where other users sufficiently address the concerns of the original poster (OP). This may be indicated by the OP simply thanking or otherwise expressing gratitude to other users for their responses. Ultimately, the goal of this effort would be to help r/gradadmissions users in structuring their questions to receive the best possible feedback and encourage future visits and contributions to discussions.

## The Data

A document for my corpus is a given user-submitted question to r/gradadmissions and any supporting text entered by the OP. Let's take a look at an example document below:

![alt text](../assets/img/document_example.png)

Here, we have a question from a reddit user named SFSUer (the OP), who is kind of an overachiever and wants to know about receiving funding for a Master's degree in Engineering. From the above, you can see that I'm treating as a document the combination of the question in the title of the submission and the body of text in the box below. In the comments in response to this question, our OP has returned to follow up but hasn't exactly thanked any other commenter at this point, so I would label this document as one where "useful feedback" has not been received.

I collected ~1000 documents like the above using [PRAW](https://praw.readthedocs.io/en/latest/), the **P**ython **R**eddit **A**PI **W**rapper, scraping user-submitted questions (or submission in PRAW parlance) and associated comments into separate data structures linked by a common submission ID. The collected documents cover a period from March 7, 2018 through May 30, 2018. For each submssion, we can also use PRAW to pull any needed information about each OP that we need as well.

Before performing any analysis, it should be noted that not every collected document is actually a question. Taking the path of least resistance, I filtered out submissions that were not explicitly questions using punctuation -- essentially looking for the presence of question marks in each document text. I also removed questions which had only been posted within 24 hours of collection, eventually leaving me with 829 documents (and their ~6000 associated comments) to analyze.

## Topic modeling: What are grad school candidates even asking?

With my complete corpus now set, I first need to transform the collected documents (the user-submitted questions to r/gradadmissions) into vectors of weighted tokens (in this case word tokens) -- that is, we map each document to all words in the corpus according to some metric capturing the frequency for each given word in the document. The metric used here is *term frequency-inverse document frequency*, or *tf-idf*, which ultimately measures the tradeoff between the frequency of occurrence of a given word in a document against its frequency of occurrence across the entire corpus. From this construction, tf-idf measures the [importance](https://www.kdnuggets.com/2018/08/wtf-tf-idf.html) of a given word to a particular document, and is suited for analyzing documents covering related subject matter.

Using scikit-learn's `TfidfVectorizer`, one can transform a set of text documents into a *document-term matrix*, where each row is a vector of weighted tokens (words) corresponding to an individual document (question). `TfidfVectorizer` may be tuned according to the following parameters to modify the document-term matrix output:

* `token_pattern`: denoting what text constitues a word (a string of at least two consecutive alphanumeric characters)
* `stop_words`: specifying a list of words to ignore -- usually commonly occurring words that provide little to no information about a document (such as most pronouns) 
* `ngram_range`: defining the minimum and maximum length of a sequence of words to evaluate (for this case only single words, and not multi-word phrases)
* `min_df`, `max_df`: specifying the minimum and maximum document frequencies for a given word to be included in the matrix -- especially rare or common words are thus ignored as uninformative.




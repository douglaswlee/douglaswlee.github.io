---
layout: post
title: Still waiting to hear...
---

The internet can be a pretty [toxic, nasty place](https://www.vice.com/en_us/article/8xxymb/here-are-reddits-whiniest-most-low-key-toxic-subreddit) (*Warning: NSFW language*), especially when like-minded toxic, nasty people get together to share their toxic, nasty thoughts. Of course, seeking out virtual interaction with like-minded people is [sometimes good](https://arstechnica.com/information-technology/2017/05/these-are-the-online-communities-we-will-never-forget/). And sometimes the much-maligned "front page of the internet" can [be good and do good](https://gizmodo.com/reddit-is-helping-some-people-deal-with-their-mental-he-1825364592).

These were some of the thoughts I had as I was planning out my fourth project at Metis, which emphasized Natural Language Processing (NLP) and unsupervised learning. Going through the vast data and resources from the [Stanford Network Analysis Project (SNAP)](https://snap.stanford.edu/data/index.html), I came across [a study](https://snap.stanford.edu/data/web-RedditPizzaRequests.html) about [free pizza requests](https://snap.stanford.edu/data/web-RedditPizzaRequests.html) made on [r/Random_Acts_of_Pizza](https://www.reddit.com/r/Random_Acts_Of_Pizza/), a subreddit where users make such requests of other users. Some of these are successful while some are not, and with results of each request documented, the group conducting the study was able to build a classificaiton model to shed light on features leading to success of actually receiving a free pizza from another user.

I decided to attempt a similar project using a different, advice-driven subreddit that happens to be close to my heart (or the darkness of it, sorry not sorry) -- [r/gradamissions](https://www.reddit.com/r/gradadmissions/), a subreddit for graduate school candidates to obtain information and share experiences about the application process. My main objectives for this project were:

* To understand which topics best describe user-submitted questions to this subreddit using Topic Modeling, and
* To explain how these topics (and characteristics of the submitted questions) contribute to incidence of “useful feedback” using a Logistic Regression model.

Here, "useful feedback" can be any collection of comments responding to a submitted question where other users sufficiently address the concerns of the original poster (OP). This may be indicated by the OP simply thanking or otherwise expressing gratitude to other users for their responses. Ultimately, the goal of this effort would be to help r/gradadmissions users in structuring their questions to receive the best possible feedback and encourage future visits and contributions to discussions.

## The Data

A document for my corpus is a given user-submitted question to r/gradadmissions and any supporting text entered by the OP. Let's take a look at an example document below:

![alt text](../assets/img/document_example.png)

Here, we have a question from a reddit user named SFSUer (the OP), who is kind of an overachiever and wants to know about receiving funding for a Master's degree in Engineering. From the above, you can see that I'm treating as a document the combination of the question in the title of the submission and the body of text in the box below. In the comments in response to this question, our OP has returned to follow up but hasn't exactly thanked any other commenter at this point, so I would label this document as one where "useful feedback" has not been received.

I collected ~1000 documents like the above using [PRAW](https://praw.readthedocs.io/en/latest/), the **P**ython **R**eddit **A**PI **W**rapper, scraping user-submitted questions (or submission in PRAW parlance) and associated comments into separate data structures linked by a common submission ID. The collected documents cover a period from March 7, 2018 through May 30, 2018. For each submssion, we can also use PRAW to pull any needed information about each OP that we need as well.

Before performing any analysis, it should be noted that not every collected document is actually a question. Taking the path of least resistance, I filtered out submissions that were not explicitly questions using punctuation -- essentially looking for the presence of question marks in each document text. I also removed questions which had only been posted within 24 hours of collection, eventually leaving me with 829 documents (and their ~6000 associated comments) to analyze.

## Topic Modeling: What are grad school candidates even asking?

With my complete corpus now set, I first transformed the collected documents (the user-submitted questions to r/gradadmissions) into vectors of weighted tokens (in this case word tokens) -- that is, I mapped each document to all words in the corpus according to some metric capturing the frequency for each given word in the document -- what's referred to as a [Bag-of-Words model](https://en.wikipedia.org/wiki/Bag-of-words_model). The metric used here is *term frequency-inverse document frequency*, or *tf-idf*, which measures the tradeoff between the frequency of occurrence of a given word in a document against its frequency of occurrence across the entire corpus. From this construction, tf-idf measures the [importance](https://www.kdnuggets.com/2018/08/wtf-tf-idf.html) of a given word to a particular document.

Using scikit-learn's `TfidfVectorizer`, one can transform a set of text documents into a *document-term matrix*, where each row is a vector of weighted tokens (words) corresponding to an individual document (question). `TfidfVectorizer` may be tuned according to the following parameters to modify the document-term matrix output:

* `token_pattern`: denoting what text constitues a word (a string of at least two consecutive alphanumeric characters)
* `stop_words`: specifying a list of words to ignore -- usually commonly occurring words that provide little to no information about a document (such as most pronouns) 
* `ngram_range`: defining the minimum and maximum length of a sequence of words to evaluate (for this case only single words, and not multi-word phrases)
* `min_df`, `max_df`: specifying the minimum and maximum document frequencies for a given word to be included in the matrix -- especially rare or common words are thus ignored as uninformative.

I tuned the last three sets of the above parameters in tandem with using [Non-Negative Matrix Factorization](http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.NMF.html) (NMF) to extract a specified number of topics from the documents. NMF allows us to derive these topics from the document-term matrix, decomposing it into a pair of non-negative matrices, the document-topic and topic-term matrix. The former of these assigns a vector of weights to each document basically reflecting the intensity of the topic in the document, while the latter assigns a vector of weights to each topic basically reflecting the intensity of the association of a given word to the topic. Consequently, for some chosen number of topics, one may obtain the top words for each topic -- for example:

```
Topic #0: gpa, major, semester, ve, grades, years, classes, overall, undergrad, low
Topic #1: school, accepted, hear, applied, faculty, worth, schools, haven, little, state
Topic #2: program, master, field, phd program, psychology, email, applied, post, accepted, research
Topic #3: research, experience, lab, research experience, good, really, summer, doing, project, related
Topic #4: just, really, want, like, feel, don, ve, lot, good, people
Topic #5: phd, ms, thesis, phd program, phd programs, experience, want, ve, apply, undergrad
Topic #6: university, state, list, universities, applying, programs, ms, strong, currently, different
Topic #7: cs, math, classes, major, courses, semester, ms, degree, level, really
Topic #8: year, semester, time, summer, got, applying, grades, applications, did, apply
Topic #9: grad, grad school, undergrad, want, getting, don, school, advice, think, ll
Topic #10: programs, applying, ve, getting, phd programs, looking, interested, sure, apply, fall
Topic #11: time, job, work, need, field, going, doing, ve, working, does
Topic #12: work, years, like, know, college, working, great, students, help, good
Topic #13: think, professor, don, got, did, letter, know, good, recommendation, professors
Topic #14: schools, gre, good, apply, scores, know, field, looking, student, experience
Topic #15: offer, interview, accepted, decision, funding, email, received, tuition, choice, got
Topic #16: graduate, graduate school, undergrad, admissions, undergraduate, classes, department, background, student, degree
Topic #17: application, college, admissions, background, undergraduate, just, applied, letters, ask, process
Topic #18: masters, degree, years, applied, looking, experience, applying, accepted, help, working
Topic #19: science, courses, engineering, course, computer, computer science, want, fall, master, summer
```

From above there is at least one topic (Topic #4) with a whole lot of nothing and at least two other topics (Topics #2 and #5, for instance) that seem redundant -- which might lead one to continue playing around with different lists of `stop_words`, values for `min_df` and `max_df`, different ranges for `ngram_range`, and of course, different numbers of topics in an effort to get topics which are clearer and more specific. (Note: This process can be extremely time and sanity-consuming, and worst of all, a little bit too manual.)

Ultimately, I was able to obtain a relatively resolved set of topics by considering only unigrams (single words) and without having to aggressively tune `stop_words` and `min_df` and `max_df` too much. Below are the 15 topics I ultimately settled upon, along with the top words associated with each topic which helped with naming each topic:

![alt text](../assets/img/Topics.png)

You can observe that there are some redundant topics -- attempts to reduce the number of topics (to say, 10-12 topics) tended to result in these same redundancies appearing while losing other topics entirely. For the most part, nothing particularly distinguished these cases from each other, except for the two Admissions topics. The topic "Admissions1" might emphasize pending admissions decisions more, while "Admissions2" seemed to focus more on decisions already processed.

## Visualizing the Topics

As previously mentioned, the document-topic matrix output by NMF represents each document as a weighted vector of the topics essentially describing how much content a given document contains for each topic. I chose to assign each document to a topic according to the topic with maximum weight for the document. Doing so gets us part of the way towards observing the distribution of topics across the documents and the relationships amongst the documents.

In order to really explore the documents and the topic distribution visually, we need to be able to take the 15-dimension representation from the document-topic matrix to two dimensions. Out of many such [dimensionality reduction techniques](https://www.analyticsvidhya.com/blog/2018/08/dimensionality-reduction-techniques-python/), I chose t-Distributed Stochastic Neighbor Embedding (t-SNE) to represent my topic model. t-SNE is a method especially [suited for high-dimensional data visualization](https://lvdmaaten.github.io/tsne/) which effectively captures the similarity between given points in its initial higher-dimensional space and then attempts to maintain the similarity in lower dimensions.

I adapted some code from a [tutorial](https://shuaiw.github.io/2016/12/22/topic-modeling-and-tsne-visualzation.html) for creating a t-SNE visualization of a topic model in [Bokeh](https://bokeh.pydata.org/en/latest/) to create the below visual representation of my documents and topics.

![alt text](../assets/img/tSNE_GradSchool.png)

We can see that documents with similar maximally-weighted topic matter tend to be to close to one another as a result of the t-SNE output. Topics pertaining to the admissions process before applications are sent out (GRE, Grades, Letter of Recommendation) seem to cluster together, with the same being true for topics focused on the post-application submission process (Interview, Offer, the two Admissions topics). I don't quite know what to make of the centrality of the Research topic, but on the other hand it is a rather critical element to the decision to attend grad school.

## OK, but what about "Useful Feedback"



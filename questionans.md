---
layout: default
---
# Question Answering Bot - V 1.0

This is an implementation of some [Natural Language Proccesing](https://en.wikipedia.org/wiki/Natural_language_processing) algorithms with the objective of answering questions about information that is readed from a corpus of documents. It was implemented using nltk in Python. 

It was made as an activity for the Harvard's Course [CS50’s Introduction to Artificial Intelligence with python](https://cs50.harvard.edu/ai/2020/). 

The bot performs two tasks: document retrieval and passage retrieval. This version, the 1.0, uses _simple_ strategies and metrics to get that, I will be working to add some features soon, such as looking for synonyms of query words, or lemmatizing to handle different forms of the same word.

## Bot Structure

* First, the system loads a corpus of documents (it should be .txt file(s) in a folder)
* Then, based on the Query, it chooses the best document in the corpus. This choice is made by [term frequency–inverse document frequency](https://en.wikipedia.org/wiki/Tf%E2%80%93idf). It uses nltk’s word_tokenize to achieve this.
* The passage retrieval is made using two index: inverse document frequency and query term density. 

## Some Examples

Using the [The Beatles' post on wikipedia](https://en.wikipedia.org/wiki/The_Beatles) as corpus, let's try some querys: 

```
$ python python questions.py beatles/
Query: when was let it be released?
On 8 May 1970, Let It Be was released.

$ python python questions.py beatles/
Query: when does ringo starr join the band?
Already contemplating Best's dismissal,[41] the Beatles replaced him in mid-August with Ringo Starr, who left Rory Storm and the Hurricanes to join them.

$ python python questions.py beatles/
Query: when was the first live performance in US? 
They gave their first live US television performance two days later on The Ed Sullivan Show, watched by approximately 73 million viewers in over 23 million households,[92] or 34 percent of the American population.
```

This version _needs_ some direct questions, because it answers by choosing the more related phrased in the text. However, it could happen that the more related phrase is not the best answer. However, this methodology could be useful to customer services if the corpus is opportunely prepared!


## How to use it

Look in my [repo](https://github.com/Marouxet/cs50/tree/06-Questions), to download the files and run the bot. 

Yo need to have in the same folder:

* A folder with the corpus (one or more .txt files)
* The file questions.py


Then, run:


```
$  python questions.py [foldername/]
```


It works!

[back](./)

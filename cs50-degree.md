---
layout: default
---
# Degree of separation between two actors

This is an implementation of a [breadth-first-search](https://en.wikipedia.org/wiki/Breadth-first_search) algorithm that looks in a local copy of the [IMDB](https://www.imdb.com/) database to find the shortest path between two actors by choosing a sequence of movies that connects them. 

It was made as an activity for the Harvard's Course [CS50’s Introduction to Artificial Intelligence with python](https://cs50.harvard.edu/ai/2020/). 

## The idea (taken from [here](https://cs50.harvard.edu/ai/2020/projects/0/degrees/))

According to the [Six Degrees of Kevin Bacon game](https://en.wikipedia.org/wiki/Six_Degrees_of_Kevin_Bacon), anyone in the Hollywood film industry can be connected to Kevin Bacon within six steps, where each step consists of finding a film that two actors both starred in.

In this problem, we’re interested in finding the shortest path between any two actors by choosing a sequence of movies that connects them. For example, the shortest path between Jennifer Lawrence and Tom Hanks is 2: Jennifer Lawrence is connected to Kevin Bacon by both starring in “X-Men: First Class,” and Kevin Bacon is connected to Tom Hanks by both starring in “Apollo 13.”
 

## Implementation

Look in my [repo](https://github.com/Marouxet/cs50/tree/degrees), download the files and run 

```
$ python degrees.py
```


[back](./)

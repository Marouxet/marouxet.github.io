---
layout: default
---
# Minesweeper

This is an implementation of the minesweeper Game, where a IA algorithm can play by itself. The system uses information from cells already discovered and generate _knowledge_ using  [proposicional logic](https://en.wikipedia.org/wiki/Propositional_calculus) algorithm. It was made as an activity for the Harvard's Course [CS50’s Introduction to Artificial Intelligence with python](https://cs50.harvard.edu/ai/2020/). 

![Board](https://cs50.harvard.edu/ai/2020/projects/1/minesweeper/images/game.png)


## Implementation

There are two main files in this project: runner.py and minesweeper.py. Look in my [repo](https://github.com/Marouxet/cs50/tree/01-minesweeper) to download the files. 

runner.py has been implemented by CS50'S staff. It contains all of the code to run the graphical interface for the game.

minesweeper.py was made by my, and it contains all of the logic for playing the game, and for making optimal moves. 
IA inferes secure movement (if it is able to do that, based on the knowledge base). It there are not secure movement, IA picks a random cell.


Download the files and run  

```
$ python runner.py
```

[back](./)

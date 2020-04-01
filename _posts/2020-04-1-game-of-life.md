---
title: 'John Conway's Game of Life'
date: 2020-04-01
permalink: /posts/2020/04/gameoflife/
tags:
  - R
  - Fortran
  - cellular automata
  - for fun
---

John Conway's famous Game of Life is a "zero player game", as he calls it. Lots of literature exist explaining the intricacies that can emerge from its very simple rules. Citing Wikipedia:

 > 1. Any live cell with fewer than two live neighbours dies, as if by underpopulation.
 > 2. Any live cell with two or three live neighbours lives on to the next generation.
 > 3. Any live cell with more than three live neighbours dies, as if by overpopulation.
 > 4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

I've been handling lattice plots lately, and given that the data-plotting part was already worked out I thought it could be fun to try and recreate the Game of Life. After two coffees and accidentally skipping an online class, I got [my code](https://github.com/malmriv/gameoflife) to work! Here's a square 128x128 lattice generated using the code and plotted using an R script:

![](https://github.com/malmriv/gameoflife/blob/master/animation.gif?raw=true)

 Some of the most famous structures can be seen, especially gliders! I think it could be fun to add options like starting the grid with well-known self replicating structures, but that's for another day. I think it would also be cool to recreate [Langton's ant](https://en.wikipedia.org/wiki/Langton%27s_ant), which has an even simpler set of rules and is probably more interesting computationally.

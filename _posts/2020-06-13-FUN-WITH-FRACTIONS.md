---
title: Fun With Fractions
date: 2020-06-13 08:00:00
layout: page
published: true
---

### A Simple Program to Prove Countability of Rationals
This was a small experiment to enumerate all rational numbers of form ***x/y***
utilizing a set of piecewise equations as a proof that rationals are countable. 
I programmed this in Ruby for ease of implementation. This includes a fraction class 
and simplification utilizing Euclid's GCD algorithm. Some of the theoretical backing 
came from StackExchange, and is cited in the readme for the project.  

The theoretical backing relies on the use of a breadth-first traversal of the Calkin-Wilf
tree utilizing Stern's diatomic sequence to enumerate the rationals. An example Calkin-Wilf tree can be seen below (image from Wikipedia). 

![calkin-wilf](https://upload.wikimedia.org/wikipedia/commons/6/62/Calkin-Wilf_tree.svg){:class="img-responsive"}

The nth rational is therfore generated using the definition of the __fusc()__ function, with the rational generated as ```fusc(n)/fusc(n+1)```. As such, the sequence forms the numerator and denominator as specified by the Calkin-Wilf tree. The program can convert between both rational numbers and their associated labels via command-line arguments.

The project itself can be found here:
[Fractions!](https://github.com/jrichards15/ruby-fractions)

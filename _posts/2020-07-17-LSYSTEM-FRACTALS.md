---
title: L-System Fractal Generator
date: 2020-07-17 05:00:00
layout: page
published: true
---

## Using L-System Grammars to Represent Recursive Patterns
### L-System Description
Essentially, this program generate strings that adhere to forms dictated by a specified grammar. As a slight (and perhaps inaccurate) brief on l-systems: 

L-systems are described as a 3-tuple: ```G = (V, omega, P)```. **V** represents the alphabet, or set of symbols containing all terminals and nonterminals utilized in the grammar. **Omega** is the starting point for the L-system, which is iterated over to produce the result. Finally, **P** is the set of rules (productions) that dictate how the system will behave given the number of symbols. A system with one production per non-terminal is deterministic, as it has a set outcome for all iterations. The code in this repo supports both deterministic and non-deterministic (stochastic) systems. See [here](https://en.wikipedia.org/wiki/L-system) for more information on L-systems.

### Lsystems.py Overview
This project began as a simple exploration into the PyCairo graphics library and practice with simple Python operations. The core generation of the l-systems utilizes an iterative approach to replace the string character-by-character with their predefined replacement.

For this project, the production rules were handled by classes ```LProduction``` and ```LProbabilisticProduction```, which both contain the rules for a specified non-terminal. The primary difference is the ```P``` field. For the standard production, this is merely a string literal. However, for the stochastic representation, this is represented as a sorted list of 2-tuples, containing a string literal and a floating point value in order to represent the probability of a given production being selected.

The systems themselves contain functions to check the given alphabet, find the production rules for a given nonterminal in said alphabet, and step the given axiom forward one iteration. They are relatively straightforward to operate and produce decent results. It is worth noting that the strings they operate on grow exponentially, leading to significant slowdowns for larger numbers of iterations.

The GitHub repo for this project can be found [here.](https://github.com/jrichards15/lsystems-python)

### Draw.py Overview
As a bonus, I decided to render the results of the l-system generation utilizing the PyCairo module, which provided a robust framework for drawing each figure. There are a few classes contained within this file. 

The ```DrawSystem``` class provides a subclass for the previously-discussed ```LSystem``` class, which allows it to operate upon the same data model while adding functionality. It utilizes the superclass attributes to generate the L-system to be drawn, then utilizes the PyCairo library to render them. It also takes in parameters for line thickness, length, and image dimensions. As such, they are fully configurable. The number of iteration the l-system will be run for is also specified. Finally, it takes in a list of production rules to specify how each character be represented graphically.

The ```DrawProduction``` class is used to represent these production rules, with each character in the alphabet corresponding to a certain operation. Since the images are rendered utilizing a custom-programmed turtle, each character can take a certain operation, defined in the ```InstructionEnum``` class, which specifies all supported operations for the turtle. The turtle code is contained within the ```ProductionTurtle``` class, which tracks its (x,y) position and its current angle. It also contains the rules for each instruction in the ```InstructionEnum```, and interprets them into PyCairo operations. 

Finally, the ```ShaderTracker``` class is a bit vestigial, as I have decided to leave it somewhat unfinished. As it currently stands, it takes in a list of hex color code values and converts them to RGB values (limited between float values 0.0 and 1.0 instead of 0 and 255), then keeps track of the current brush color until the next time ```next_color()``` is called. It is employed in a few of the ```DrawSystem``` functions, including ```draw_color(...)``` and ```draw_subtractive_color(...)```.

Drawing results in the creation of a results directory and a .png file. The images are black-and-white and contain the centered rendering of the given l-system. There is support for drawing multiple iterations of the same l-system in the function ```draw_multiple()```. I am working on colorizing these images, though I'm losing interest in the project and will pick it up again in a few weeks. The color functions rely on overlaying multiple iterations in order to show progression, though this sometimes results in blur and unsightly overlap. I attempted to address this in ```draw_subtractive_color``` to mixed results, which I will eventually go back and fix.

As of the current version, only the deterministic L-systems are supported due to the inheritance of the ```LSystem``` class rather than the ```ProbabilisticLSystem``` class. Perhaps I will go back and include a fix for this in the next version. For now, I have decided to leave it as-is. 

### Example Outputs
Here are a few examples of the results:
![dragon-curve](/assets/img/dragoncurve.png){:class="img-responsive"}
![sierpinski-triangle](/assets/img/sierpinski.png){:class="img-responsive"}
![arrowhead-curve](/assets/img/arrowhead.png){:class="img-responsive"}

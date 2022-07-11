---
title: "VLSI"
excerpt: "An optimization solver for the VLSI problem."
header:
teaser: "/assets/images/portfolio/transistor-boxes.png"
last_modified_at: 2022-07-11
--- 

# VLSI solver

![image](/assets/images/VLSI/out-35.png)

## Problem description

VLSI (Very Large Scale Integration) refers to the trend of integrating circuits into silicon chips. A typical example is the smartphone.

The modern trend of  shrinking transistor sizes, allowing engineers to fit more and more transistors into the same area of silicon, has pushed the integration of more and more functions of cellphone circuitry into a single silicon die (i.e. plate).

This enabled  the modern cellphone to mature into a powerful tool that shrank from the size of  a large brick-sized unit to a device small enough to comfortably carry in a pocket  or purse, with a video camera, touchscreen, and other advanced features.

As the  combinatorial decision and optimization expert, the student is assigned to design  the VLSI of the circuits defining their electrical device: given a fixed-width plate and a list of rectangular circuits, decide how to place them on the plate so that the length of the final device is minimized (improving its portability).

Consider two variants of the problem. In the first, each circuit must be placed in a fixed orientation with respect to the others. This means that, an n × m circuit cannot be positioned as an m × n circuit in the silicon plate. In the second case, the rotation is allowed, which means that an n × m circuit can be positioned either as it is or as m × n.

## Project

We developed a solver for the [VLSI](https://en.wikipedia.org/wiki/Very_Large_Scale_Integration) problem using three different technologies:
- Constraint Programming (CP)
- Satisfiable Modulo Theory (SMT)
- Linear Programming (LP)

The project was developed for the "Combinatorial Decision Making and Optimization" course at the Artificial Intelligence master's degree at the University of Bologna. You can find the assignment [here](assignment.pdf), in particular we chose **Problem 1**.

The task is to find a solution to 40 instances of a VLSI problem. The input is the number of circuits *n*, the width of the board *w*, and a list of circuits, in particular for every circuit we have the width *x* and the height *y*.

In particular, the goal is to place all the circuits in the board, respecting the width constraint while minimizing the height of the plate *h*.

Thus, the output requires the height *h*, and, for every circuit, its coordinates in the board *(p_x, p_y)*.

The problem had to be resolved in two ways: in the first one we could not rotate the rectangles, while in the second one we could rotate them.

## Installation

First, you shall clone this repo on your machine and move to the project directory:
```shell
git clone https://github.com/francescoolivo/VLSI.git
cd VLSI
```

Now create a conda environment and activate it:
```shell
conda env create -f environment.yml
conda activate VLSI
```

If you want to call the environment with a different name, you just have to change the environment name in the first line of the `environment.yml` file.

## Performances

This is a comparison among the performances of the solvers, without allowing rotation:

![image](/assets/images/VLSI/solvers.png)

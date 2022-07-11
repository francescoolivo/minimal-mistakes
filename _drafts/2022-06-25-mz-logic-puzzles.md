---
title: "Building a constraint solver for logic puzzles"
categories:
- "AI"
---

Hello there! Today we are going to investigate a problem which is not related to the usual data-science articles. Instead, we will use constraint programming (CP) to solve logic puzzles that the Bocconi University assigns during the Italian national math competition.

For those of you who do not know what constraint programming is, CP is a field of AI and of Operative Research. A CP solver is a software designed to take in input the model of a problem and a set of constraint, and, by using efficient search trees, find the solutions that satisfy the constraints.

Many Bocconi's logic puzzles are particularly suited for this particular approach. The provided code was developed by me and Yuri Noviello, as an assignment for a college course.

## The software

The software we decided to use for this task is Minizinc: it is a constraint programming oriented language, which allows to provide problems and constraint in a quite simple way.
Nonetheless, it provides very little documentation, so, for example, finding examples of programs dealing with strings was almost impossible, thus we approached many problems in an "algebraic" way.

Another possible option for this kind of problems is Prolog, which is less suited but may be used in this way. It's likely that we will develop the Prolog version too, and in the case I will append it to this article.

## The problems

Bocconi's puzzles are quite known in Italy: the university is known for organizing the main Italian math competition, which every year brings to Milan many kids. I had the chance to go when I was twelve and ranked 77th, which was a great result for me. Ironically, I would have ranked much higher if a didn't make a mistake in dividing a number by 2. Nice.

Nonetheless, these puzzles require math and logic skills, and can be quite complex.

You can find the dataset we used [here](https://giochimatematici.unibocconi.it/images/autunno/2021/practiceq.pdf). They are in Italian only, but I will translate the puzzles I will solve.

So, no further ado, let's get to the code!

### Problem 1

*Which is the smallest number (positive integer) of four digits, all even and different, that is divisible by 11? A number cannot start with the 0 digit.*

To solve this problem, we will look for a list of digits which satisfy our constraints. Let's list them all

1. We aim to minimize a number
2. The number is a positive integer
3. The number has four digits
4. The digits are all even
5. The digits are all different
6. The number is divisible by 11
7. The first digit is different from 0.

Now let's get to the code.

```Minizinc
include "globals.mzn";

% Defining the variables
array [1..4] of var int: x;
var int: n;

% Setting the constraints
% First of all let's define teh number n as a function of the digits
constraint (n = x[1]*1000 +x[2]*100 + x[3]*10 + x[4]);

% Constraint 3: dealing with digits and not any number
constraint forall(i in 1..4)( x[i] >=0 /\ x[i] <=9);

% Constraint 4: the digits are even
constraint forall(i in 1..4)( x[i] mod 2 =0);

% Constraint 5: the digits are all diffferent
constraint alldifferent(x);

% Constraint 6: n is divisible by 11
constraint (n mod 11 = 0); 

% Constraint 7: the first digit is different from 0
constraint (x[1] >0 );

% Solving by minimizing n
solve minimize n;

output ["The result is \(n)\n",];

```

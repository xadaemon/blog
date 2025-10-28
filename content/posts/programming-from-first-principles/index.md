+++
title = "Programming from first principles"
date = "2025-10-27"
draft = true
tags = ["First Principles","Fundamentals","Programming"]
layout = "post"
+++

# So you want to learn programming on a deeper level

How about learning it at a deeper level? Come with me and learn why programming languages exist, how the most famous ones compare
and how we know what our computers can and cannot do.

# What you should already know

I wrote this from a presumption that you know no programming or know very little, but that you know your maths somewhat well,
you should know what a variable is, and your basic arithmetic. If you know programming read on as this might be enlightening
for you too.

## 1 - Computing and turing machines

In this post I will explain turing machines, how they work and how it relates to programming

First let's explain what a CPU fundamentally does and how, let's imagine our CPU, it has boxes on it's "desk" called registers they
can hold a number each, and it has access to a much larger number of boxes called main memory, but if it wants to access the number
in one of these boxes it has to ask for the box, and copy the number to one of the boxes in the desk, our cpu speaks a language
that we call the instruction set. Examples of this are things like the ARM cpu in your phone and the intel/amd x86_64 cpu in your pc
this language they speak is made up of numbers that have meaning assigned to them, i.e if we decide that our cpu understands the
number 80 as telling it to move a value from box A to box B.

Now lets look at a turing machine, in it's simplest form the machine has a tape of infinite length, a head and an alphabet, the head
will look at a cell in the tape and it can set a new value that we call the state and move the tape left or right or halt the machine,
can you see the similarity?

Let's play pen and paper turing machine to cement the ideas.

First the alphabet of our machine
| Symbol | Instruction | Next state |
| -- | -- | -- |
| P0 | Puts `0` moves right | R0 |
| R0 | Move the tape right | P1 |
| P1 | Puts `1` moves right | R1 |
| R1 | Same as R0 | P0 |

If you then initialize your machine with it's state preset to `R0` you will have an infinite string of 0 &lt;blank&gt; 1 ... repeating
take a piece of paper and validate that this holds, a machine capable of storing, reading values and changing it's internal state is
said to be turing complete.

A processor is little more than a turing machine with a large alphabet and a more complex storage scheme.

---
layout: post
title: Understanding the Metal [Part 0]
description: Intro to a new series that will follow my progress through understanding how cpu's and languages are architected and implemented.
img: understand_metal.png # Add image post (optional)
tags: [CPU, Hardware Architecture] # add tag
---

## Intro

There have been many times since I started my programming journey where it's been made obvious to me how significant the hardware is to the programs we write. Being that all code is essentially a sequence of data transformations, the most important part of writing something that runs efficiently is having an understanding of what that data is, and precisely what we want to do with it. To figure out the best way to transform the data, one would imagine that knowing what is physically possible on a given piece of hardware would be a useful bit of information.

I have a general sense of things that helpful and harmful to this endeavor, but I'm nowhere near the level I'd like to be at. I stumbled upon an amazing [website](https://www.nand2tetris.org/)/course called nand2tetris that starts from simple NAND gates and builds up chip by chip to a working simulated computer. This and a few other resources will comprise the bulk of the info I plan on consuming in this space. I will be working on this in parallel to the game engine series where I will be thinking about Intel's architecture specifically, and will apply what I'm learning building my own architecture to problems I face there. 

The motivation for building my own architecture from primitive blocks is to help build intuition for hardware architecture in a general, and really understand the consequences of each instruction I generate from code. Part of building intuition for a subject for me is teaching. Trying to go through the process of teaching someone else what I'm learning forces me to understand it at a deeper level, deep enough to explain WHY something is the way it is not just that it IS. 

Hopefully documenting this process will be useful to someone some day, I don't claim to be any sort of expert in this stuff, my goal is that the documenting the steps I take to get the understanding I'm after will help someone if they're going down the same path. 

## The Only "Actual" Hardware We'll talk about

Despite the entire point of this activity being to understand how the hardware works, I don't actually want to be working with hardware or worrying about things in the domain of electrical engineering. I want to understand a small set of basic concepts that can get me to the NAND gate, and from there I will be doing everything in hardware simulations, and conceptual diagrams. I've tried implementing some of this stuff on breadboards and it gets out hand VERY quickly.

### Voltage 

**Motivation:** At it's lowest level, the data in computer systems is expressed with a series of 1's and 0's, but that's still essentially a software level concept. Computers don't move around slips of paper with 1's and 0's written on them. So what is going on in the real world that we call 0 or 1?

Computers are electrical machines, so the fundamental unit is electricity, and we can make things happen by manipulating the flow of the current. So when we talk about binary (0 and 1), the physical correlation is a **low voltage = 0** and a **high voltage = 1**. Someone could give you a more formal definition of voltage, but for what I care about I just think about it as how much "juice" or "power" is coming across a wire.

*Little to no current flowing means the lightbulb won't turn on (0)*

![Low Voltage](/assets/img/low_voltage.png) 

*Enough current flowing and the lightbulb will turn on (1)*

![High Voltage](/assets/img/high_voltage.png) 

Something to note is that 0 and 1 don't map directly to 0 volts and 1 volt, there is threshold where any voltage below a particular amount is "low" and any voltage above another amount is "high". I don't think this is ubiquitous across platforms but a recurring set of values I saw were anything under 0.7-0.8 volts is considered "low" and anything above ~2 volts is "high".

The logic we want to eventually get to is **bolean** logic, where we are doing operations on a space of 2 values. There are many ways to refer to these values, and they get used interchangeably to some extent.
- High / Low
- True / False
- Yes / No
- On / Off
- 1 / 0
- ...

### Transistors

*add more info on why we need this*

The transistor is the next thing I want to discuss. We know that we're working with an electrical system, and that we need to control the current in our system to get variations in voltage, but how do we do that? You can think of a transistor as essentially a current switch, given an input of some voltage, we can "turn on" the transistor and current can flow, otherwise current will not flow through. Let's go over a quick diagram then we'll go into an example to show why we want this.

![Transistor Off](/assets/img/transistor_base.png)

This is a basic diagram of a transistor, don't worry so much about the names, if we keep thinking about it in terms of a switch, the base is what turns the transistor "on" or "off", in the image below you can see that if we apply a bit of current through the base, we now allow a large current to pass over from the collector to the emitter. I think of it like pressing a button or flipping a switch, pressed/up the current flows, released/down the current does not flow. 

![Transistor On](/assets/img/transistor_on.png)

I suspect that many people are probably in the 'who cares?' camp at the moment, totally fair, this is certainly more info that you really need to get the level of understanding I was after. However, I have a hard time not digging down to the bottom of the well and a bit further just for good measure. So now you get to be subjected to it too :)

Let's get to the main use for this knowledge for my purposes before I lose whoever's left!

### Hardware NAND

what the heck is NAND 

So I mentioned earlier one of the main resources I was looking at for information being called nand2tetris, I also mentioned that that was going to be the place where I left the land of hardware and electrical engineering and turned towards hardware simulation. But why NAND? What is it, and why do we want it? The short story is, a NAND gate is a logical operator where given an input of two binary values we will get an output of 0 if BOTH values are 1 and 1 in any other case. With this we can build every other logical operation we want from a series of NAND gates which we will be doing. As we string more and more together we gain higher and higher levels of expressivity, which we can use to build complex systems like a CPU. 

As I mentioned NAND (short for Not And) is a boolean operator that operates on 0's and 1's. Since we are working in a boolean space there are a finite number of inputs and outputs, few enough where it's fairly trivial to write out the entire set of possibilities. This is called a truth table, and the one for NAND looks like this.   

![NAND truth](/assets/img/nand_truth.png)

We have our inputs A and B, and our output Out. For each permutation of possible values for A and B we can see what the output of the logical operation is. Again, note that the output is 0 or low if and only if both A and B are 1/high. This is the inverse of what AND would look like, where the output would be high only when both the inputs are high. That's why this operation is NOT AND, it's the output of AND but inverted.

We'll get into building other boolean operations shortly, but for now let's check off the last bit of hardware understanding and apply our knowledge of transistors before we leave that space.

### Hardware Layout

![NMOS NAND](/assets/img/nmos_nand.png)

Here is the basic diagram describing one possible hardware implementation of a NAND gate, this one is using NMOS transistors (the things colored in blue), I just think about them in the way I described transistors previously. At the top of the diagram we have a 5volt (or some other higher voltage input) which passes through a resistor (we don't really need to know a ton about resistors, basically the important part is that we drop the voltage from 5volts to something lower but still in the range we consider "high" or 1), there's a wire that goes out to the right which I've labeled out to keep in line with our previous naming. Then there are two resistors who base connections from the transistor diagram are labeled A and B respectively (these are our inputs). Finally, the last symbol stands for "ground" and is a potential termination point for the circuit. 

Let's go through some examples of inputs to the system to see how this correlates to the truth table.

![NMOS 00](/assets/img/nmos_0_0.png)

Our first example is for the resting state, both inputs are low/0 so current will not flow through them. Since there is nowhere else to go, the voltage flows through Out giving us high/1 on that line. You can see this correlates to the first line in the truth table 0 NAND 0 = 1. 

![NMOS 10](/assets/img/nmos_0_1.png)

I don't think this is actually how current works but for the sake of conceptualizing it's what I chose. If we imagine just one of the inputs is high, A in this case, we know that transistor will allow current through. However, since the other is still closed, the voltage still has nowhere to go in that direction, so it still has to leave through Out, again, resulting in a high output. You can see how this would also be the case if just B was high, so this takes care of the next two lines in the truth table 1 NAND 0 = 1, 0 NAND 1 = 1.

![NMOS 11](/assets/img/nmos_1_1.png) 

The final case occurs when both inputs are high, allowing current to flow through both. Now that this is the case, the electricity can flow through to the ground wire instead of going to Out. The reason this is the case is beyond what I'm interested in exploring, if you're interested you can explore further down the electrical engineering route. As a drastically oversimplified way of viewing it you can imagine the ground wire having a stronger "pull" on the positively charged electrons, causing them to go that direction rather than through Out. This gives us our final entry in the truth table 1 NAND 1 = 0.

## Where to now?

Ok, unless there's something I come across that seems important, I've personally gone deep enough to be happy taking NAND as a given unit that I can start using and not care about hardware and electricity so much. 

We will be moving on to discussing boolean logic in more depth, as well as other boolean functions, and start building a few of the other logical gates we will need to start making progress on the CPU. Until next time!
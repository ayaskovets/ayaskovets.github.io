---
layout: post
date: 2021-11-5
title: Evolution of interfaces
excerpt: In the most general sense one could think of an interface as a collection of ways a thing allows other things to interact with it. But how abstract this idea really is?
---

Let's try to figure out from bottom up how the simple idea of abstracting away the internal details of complex objects has evolved in the context of computer science and how much meaning the term 'interface' really has in this context.

## Electrons

In the beginning was an electron. I believe this is the best place to start since electrons are in a way the interface to electricity as a phenomenon. The mathematical model here is electrical field. Its the strength of the field that allows us predict the movement of electric charge. And along with electric charge the movement of energy and more importantly information.

Since the discovery of electron almost one and a half centuries ago electric current has become almost the one and only way to transmit and store information. On this level the interface consists of two operations:

- **capacitance** - to store electric charge and create **voltage**;
- **current** - to move the charge from one place to another.

There are many ways of manufacturing batteries, different types of wires, but that is really not important from the perspective of interfacing the electricity. The right point of view for now is that the laws of physics is the implementation detail. Or ultimately the Universe itself.

## Atoms

One level up are atomic-level structures and crystalline solids. Thanks to Pauli exclusion principle, fermions and in particular electrons can not occupy the same quantum state part of which is the potential energy that an electron has relative to the nucleus.

This means that in some configurations of atoms there would be electrons on energy levels high enough that relatively small external amounts of energy or thermal exitations would be able to detach electron away from the nucleus. The act of electron 'flying away from its temporary home' is move from valency band to conduction band. And it is crucial as it is the basic principle behind one of the versatile inventions of the 20st century - semiconductors.

Between these two levels could be an energy gap of a particular size. This size is how much the energy of the solid needs to be increased to make it conduct. Different materials can be manufactured so that this energy gap is of specific size and therefore make the simplest logical element switch for electric current. The obvious property of this good abstraction is:

- **switch** - do or do not allow **current** to flow based on some external state.

## Transistors

Semiconductors are fine by themselves but along with some engineering tricks they are used to manufacture more intricate devices - transistors.

The basic structure needed to make a transistor is PN-junction which occurs on the boundary of two types of semiconductors. These include:

- n-type. Manufactured by doping a crystal (silicon for ex. with four electrons in its valence band) with an element with a larger number of valence electrons (for silicon this number is five) which due to crystallic structure of the material makes the leftover electrons free (one that does not fit the four connections of the crystal). These electrons pass the charge when the crystal is connected to a battery;
- p-type. Here a crystal is doped with an element with a fewer valence electrons. This way the type of conductivity is via the free electron holes. These holes pass the charged electrons through when the semiconductor is connected to a battery.

Connecting anode to the p-type end of a semiconductor and catode to the n-type end (forward bias), we get a diode. Diode acts as a one-way conductor because the internal flow of electrons from n-type to p-type parts of the whole semiconductor creates an area around the PN-junction which acts as an insulator. This way, when anode is connected the n-type end and catode to the p-type end (reverse bias), only high-energy electrons are able to surpass the insulation layer thus requiring a large voltage.

Now things need to be only a bit more complex. We need to make a semiconductor out of three parts and it can be done in a couple of ways:

- n-type connected to p-type connected to n-type;
- p-type connected to p-type connected to p-type.

Both do not conduct electricity in neither forward nor reverse bias. To make this structure useful we need to add a third terminal which would either inject electrons to the middle part of the semiconductor or shrink the insulation area by changing voltage between the two PN-junctions. Thus the main function of a transistor is:

- **amplify** - regulate the amount of **current** or act like a switch depending on control **current** or **voltage**

## Logical circuits

This tier makes a bit larger leap through the levels of abstraction. The reason is that we are slowly moving away from the necessary physical devices towards the vast possibilities of an abstract design space. There are many ways of manufacturing logic gates either based on transistors or not, but the main goal of this operation is to create a set of binary logical elements which satisfies the criteria functional completeness. Inputs and outputs for these elements are represented by voltage.

The most basic devices are the unary NOT (inverter) and binary AND (conjunction) and OR (disjunction) and XOR (exclusive disjunction) gates. These represent the basic functions of binary logic and could be visualized via truth tables. To achieve functional completeness (which means having a set of gates that can be used to construct any binary function with any number of arguments) we need only a single gate. And none of the above is sufficient enough.

The bases for the binary logic in electric devices are NAND and NOR gates. One can proof that any binary function could be implemented in terms of one of these gates. And this is pretty much the main function of the logical circuits - being building blocks for more complex operations, so interface here is something like:

- **perform arbitrary operations** on binary numbers represented by distinct **voltage** levels using **voltage** or **current** as a controller

Some of the more specific applications of the logical gates include:

- flip-flops or latches - simplest stateful gate. Two inputs are used to manipulate the state bit on the output. The "stateful" here implies some clocking device that defines the possible frequency of the state transitions;
- counters and registers - a combination of flip-flops that represents a stored binary number which can be loaded or read;
- ALU (arithmetic logic unit) - general purpouse circuit that is designed to perform arithmetic operations on binary numbers.

## Processors, memory, networking, other hardware

There it is. Out of the previous beautiful abstractions something more general-purpouse could be created. Again, specific CPU architectures here are not important. The principle is using stateful logical circuits to store a fixed amount of information and having a computing circuit to read from and write into the memory. Everything else here is a matter of interpretation. 

- **store** information in binary form
- **manipulate** information in binary form

A specific CPU architecture exposes a set of instructions generic enough to express pretty much any computation. Instructions are represented by binary operation codes stored in memory. Basically this gives us a way to write a program and later execute it on a CPU or another hardware designed for different problems like displaying information or proxying it to other devices. There is also a lot more to unpack on this layer of abstraction by going deeper (c) into the specifics: memory management unit, caches, hardware parallelism etc.

The other hardware function that belongs to this section is networking. This is also a gargantuous topic that comes with its own abstraction layers like OSI, TCP/IP, wide variety of protocols. But to make the point of this discussion, we would only need a method of exchanging the information between processors. By providing a transmission medium like wires, air or optical fiber and a way of representing bits like different **voltage** levels, a modulated property of a electromagnetic wave, and photon detectors correspondingly, we unlock the next necessary part of the interface:

- **send** and **receive** information in binary

## General-purpouse computers, APIs

The next and the most recent level is programs. Having got the variety of the tools to manipulate binary numbers we can abstract further and finally implement specific ideas in digital space. I think it is better to treat compilers, interpreters or programming languages as an implementation detail and to better focus on inter-application communication.

Now that there are programs that have some intelligence on their own, their communication protocols are the interface themselves. Let's consider the main unit of work in the global network of computers currently - something like a web-server which is capable of handling a specific set of requests. Here the main decisions to make are the structure of a message passing between applications and the properties of the set of requests that the application could handle. The restrictions are that the message as well as the application interface has to be dynamic and flexible enough although that impacts performance a lot. It seems like the way to go for the modern apps is either binary format like protocol buffers along with gRPC or REST + JSON.

With all of that we can easily represent real-world actions by application-like actors and have basically achieved the complexity that allows to think about computations declaratively. The pinnacle of all the ladder of abstractions above is something like:

- **perform arbitrary actions** in an abstract space that also could be mappped onto the real world

## ...AI?

Well, actually it seems like the structure of application interfaces at the moment is still too rigid to become an implementation detail.

Maybe we will come together and develop a protocol generic enough to allow to easily integrate different independent applications without them knowing about each other. Maybe we will, as well as with many other hard tasks, employ neural networks to pass the responsibility for the answer onto the statistics. Maybe even the current solutions will proof to be sufficient enough.

Or maybe we will just discover a way to make strong AI. In any case, the greater goal of a good interface is to be transparent, understandable and obvious for all its consumers: for hardware, for software. And for us.

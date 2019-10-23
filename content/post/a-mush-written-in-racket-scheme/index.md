---
date: "2013-09-03 08:47:03"
title: "A MUSH Written in Racket (Scheme)"
---
I've been working on a project for the past couple days to learn the Racket programming language.  Racket is based on Scheme, which is in turn based on Lisp.  Racket includes a compiler that will produce a native binary on Mac OSX, Linux, or Windows as well.

The project I'm working on is a simple <a href="http://en.wikipedia.org/wiki/MUSH">MUSH</a> (Multi-User Shared Hallucination).  The code is on my <a href="https://github.com/vaelen/vaelen-mush">GitHub</a> page if you want to take a look at it.  It doesn't do much yet, but I'll keep adding to it as I have time to do so.  The initial code base is around 260 lines of code and it allows players to chat with each other, look at the room they are in, and see who is online.  It also contains the necessary data structures to allow users to move from one room to another and to have objects in their inventories, but the code to make use of those structures hasn't been written yet.

Part of my goal with this project is to have a simple in-game environment for creating rooms, objects, and commands on-the-fly.  The codebase still needs a long term storage mechanism before it will be useful though, as everything is currently stored in memory.

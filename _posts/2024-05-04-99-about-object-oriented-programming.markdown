---
layout: post
title:  "About Object Oriented Programming"
date:   2024-05-04 14:18:50 +0200
categories: oop
---
I have been working with Object Oriented Programming for a couple of years. During this time, I have seen many concepts, approaches and principles that could be useful for programmers. This blog post aims to collect these ideas.

# 99 Bottles of OOP

- abstraction is a tool, prepares for future change, but adds indirection which makes the code harder to understand
- What does this thing mean in the context?
    - What is a beer in the context of thr 99 bottles song? - It is a thing you drink, a beverage
- Does the code indicates the core concepts, challenges that it solves?
- Collect questions a bout the domain, write some business description about the problem
-  be careful with red herrings (obvious abstractions), they can be misleading
-  introducing an abstraction too early can prevent finding meaningful abstractions
    - wait as long as you can, you will have more information to create clear abstractions
- check if the abstraction is meaningful in the context
- when creating public API-s, consider how much knowledge is needed to call a method (parameters)
- - method names should be only one level higher than the concrete code
- create a table with the inputs and the the output, find a column, category name (what would the column name be?)
    - imagine more concrete examples to help you find a good method name
- don't use the same name to different concepts

- TDD: add the simplest test, add the simplest solution, add the simplest test that can ruin the code
    - Fake it, Obvious implementation, Triangulate (add multiple tests, which should be passed by creating an abstraction)
    - progress in small steps, take time, this will lead to good abstractions

-
- 
- avoid mirroring the code in the tests (echo-chamber)
- 
- Open/Closed principle, don't mix the flow of making the code open to a new requirement and implementing it
    - fix code smells until you can make the code open and then add
- Refactoring is changing the structure of the code, while keeping the behaviour the same (tests should pass after each change)
- Flocking rules
    - Find the things that are most alike, find the smallest difference, make the smallest change to remove the difference
- sometimes hard problems are only hard because the easy problems are not solved yet
- 

- some metrics, SLOC, ABC, FLOG - CC
- a code might have worse metrics, but can be more understandable if it names more useful abstractions
- 
- conditionals are OO code smells, they require the method to know about the parameter and supply behavior for the cases
    - a manageable OO app should consist of small purpose-built objects
    - a nice object should say, I do this, not I do this when and if (choosing what we want to do should happen in designated space)
- conditionals breed -> solution?
- we can remove conditionals by creating objects
- - we can replace conditionals with state/strategy patter (composition)
- we can replace conditionals with polymorphisim (inheritance)
- factory - should no which objects to create, to collaborate on solving a problem, conditionals are useful here(this is where conditionals go to die)
- 
- code is striving to be ignorant, ignorance mean less dependency
- 
- primitive obsession, we pass around primitives ans supply behaviour to the, instead of using domain objects
- OO lets us to model ideas, not just physical objects
    - Physical: Ticket, User | Ideas: Purchase, Refund, Discount
- methods should be named after what they mean (more abstract), classes should be named after what they are (more concrete)
- 
- we can't guess where performance issues will appear, we should measure performance and improve it base on data
- 
- data clumps are data appearing together often, methods, variables etc..,
    - this indicates a missing concept, extract to methods or classes

- Liskov substition
    - a class should be replacable by its subclasses (polymorphism)
    - a Subclass respects the common API to achive this
    - a class only know the interface, not the internals

- Dependency Inversion Principle
    - we should depend on an abstraction (parameter objects instead of hardcoded )
    - high level modules should not depend on low level modules (level is the abstractness, high level modules the top level of the piramid, rearly change, users interact with this)
    - abstaction should not depend on the details, details should depend on the abstraction
        - high classes get the objects they collaborate with as parametes, and the pass argument to them as well ?
        - a detail is chosen by the abstraction
- ISOLATE THE BEHAVIOUR THAT VARIES
- Law of Demeter (Principle of least knowledge)
    - an object should not reach trough their collaborators, and poke in their implementation, neighbours
    - tell dont ask, I tell you what I need (single method call), I don't ask (about your stuff and figure out what I need)
    - the collaborators mean a class (if each method in a chain retuns the same, it is okay, String, ActiveRecord, we only interacted with one class)
    - we can solve this with delagation, name the method what we want IN THE CONTEXT
    - makes testing easier, we dont have to mock a function that returns a mock and so on, easier test setup
    - 
- object creation should be moved to the edge of the app, down the stack, create it before
- coperate at the middle ?
- instance method should resist knowing concrete class names, expect factories
- 
- use wishful thinking, comment the code you wish you had and refactor until it works!
- 
- tests are useful for explaining the domain, each class should have its own tests
- if writing tests are hard, it means coupling is thigh
- for simple logic, we don't need direct tests (we should avoid echo chamber, tests closely resembling the original code, it creates a lock step change problem )
- integration tests prove the correctness of the collaboration between a group of objects
- integrationtests could masqurade as unittest if the tested class creates its own collaborators, does not use dependency injection, if it has its own story, it should have its own test
- we can skip unit tests for small classes(integral part of another class, invisible from the outside), they should be exercised in integration tests for the other class
- Dont put pattern names in the name of classes, concepts are important not the patterns
    - exception: if the pattern is intention revaling, we can include it (VerseFake, ApplicationController)
-
    -

## Resources
- https://dev.37signals.com/series/code-i-like/
- OOP books that give examples for clear abstractions
- Refactoring, Martin Fowler, Jay Fields for Ruby version
- gang of four - design patterns.

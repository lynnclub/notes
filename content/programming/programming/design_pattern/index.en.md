---
title: "Design Patterns"
weight: 4
type: "docs"
---

![Design Patterns](design_pattern.png)

Singleton Pattern: Class can only have one instance, provides global access point.
Simple Factory: Factory class decides which product class instance to create based on input parameters.
Factory Method: Defines interface for creating objects, lets subclasses decide which class to instantiate.
Abstract Factory: Creates families of related or dependent objects without specifying concrete classes.
Builder Pattern: Encapsulates complex object construction process, can construct step by step.
Prototype Pattern: Creates new instances through copying (clone), has performance advantages with multiple attributes, shared attributes inherited in same prototype layer, saves memory.

Adapter Pattern: Converts class method interface to another interface clients expect.
Bridge Pattern: Separates abstraction and implementation parts, allowing both to vary independently, converting inheritance relationship to association relationship.
Composite Pattern: Composes objects into tree structures to represent "part-whole" hierarchies.
Decorator Pattern: Dynamically adds new functionality to objects.
Facade Pattern: Provides unified method to access group of interfaces in subsystem.
Flyweight Pattern: Effectively supports large numbers of fine-grained objects through sharing techniques.
Proxy Pattern: Provides proxy for other objects to control access to that object.

Visitor Pattern: Adds new functionality to group of object elements without changing data structure.
Template Pattern: Defines algorithm structure while deferring some steps to subclass implementation.
Strategy Pattern: Defines series of algorithms, encapsulates them, and makes them interchangeable.
State Pattern: Allows object to change its behavior when its internal state changes.
Observer Pattern: One-to-many dependency relationship between objects.
Memento Pattern: Preserves object's internal state without breaking encapsulation.
Mediator Pattern: Uses mediator object to encapsulate series of object interactions.
Interpreter Pattern: Given language, defines representation of its grammar and defines interpreter.
Command Pattern: Encapsulates command request as object, allowing parameterization with different requests.
Chain of Responsibility Pattern: Decouples sender and receiver of requests, giving multiple objects chance to handle request.

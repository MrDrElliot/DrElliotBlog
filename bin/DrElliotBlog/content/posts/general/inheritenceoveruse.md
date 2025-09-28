+++
title = "Overuse of Inheritence."
date = 2024-08-16T13:46:48-05:00
tags = ['LifeLessons','Paradigms']
comments = true
+++

Inheritance in object-oriented programming **(OOP)** is a fundamental concept that is deeply embedded in the design of Unreal Engine, and in many cases, it's an elegant and powerful tool. However, like any tool, it can be overused, leading developers into a trap that can complicate development and maintenance in the long run.

### The Trap of Overusing Inheritance

Inheritance is often the go-to solution for sharing functionality across different classes. While it provides a straightforward way to extend and modify behavior, over-relying on it can lead to a rigid and tightly coupled codebase. This rigidity can make your codebase inflexible, harder to maintain, and challenging to extend. When you heavily depend on inheritance, you might find yourself in a situation where changing a base class can have far-reaching and unintended consequences across your entire project.

### The Modular Approach: A Look at the Gameplay Ability System

A prime example of avoiding the pitfalls of overusing inheritance is Unreal Engine's **Gameplay Ability System (GAS)**. GAS is designed to manage complex character abilities in a modular and decoupled way, which contrasts sharply with inheritance-heavy designs. In GAS, each ability is a self-contained unit that operates independently, without direct references to other abilities or game systems.


>People always try do that, the first thing they try is-
 "Where is the handle of the ability?", and "How I know the reference of the ability?".
-Middle


When developers first approach GAS, they often ask, "Where is the handle of the ability?" or "How do I know the reference of the ability?" This mindset reflects a more traditional, inheritance-based approach. However, GAS encourages a modular design where abilities are encapsulated and interacted with through predefined interfaces and events, not direct modifications. This decoupled approach is similar to **fragment design**, emphasizing composition over inheritance and ensuring that each part of the system can evolve independently without breaking other parts.

### Alternatives to Inheritance

To avoid the pitfalls of overusing inheritance, it's essential to explore and utilize other design paradigms. One such approach is **fragment design**, which I personally favor. This style emphasizes breaking down functionality into smaller, reusable components that can be composed together in flexible ways.

**Fragment design** allows you to create a more modular and decoupled system. In Unreal Engine, this can be achieved through the use of **Instanced UObjects** and **FInstancedStructs**. These can encapsulate specific pieces of functionality, which can then be attached to different actors or used across different systems without the need for a shared base class. This approach minimizes dependencies between classes and encourages composition over inheritance.

For larger systems, **UActorComponents** are another powerful tool in Unreal Engine. They can be individually placed on actors, each providing a specific piece of functionality. By utilizing components, you can avoid the pitfalls of deep inheritance hierarchies, creating actors that are flexible and easily extended without being tied to a common base class.

### The Problem with Data Handles/Injection

While fragment design offers a compelling alternative to inheritance, another approach that has gained popularity is the use of **data handles** or **dependency injection**. However, I believe that this approach can be even more problematic than inheritance. Data injection often leads to hidden dependencies and can make it difficult to trace how data flows through your system. This can create a codebase that is hard to debug and maintain, as the relationships between objects become obscured.

### The Role of Interfaces

Interfaces are another tool often used to reduce the reliance on inheritance by defining a contract that multiple classes can implement. However, they are not a silver bullet. Overuse of interfaces can lead to its own set of problems, particularly in Unreal Engine. Interfaces can make your code harder to understand, as you might end up with a proliferation of small, single-purpose interfaces that can complicate your class design.

While interfaces have their place, they should be used judiciously. Instead of relying heavily on interfaces, I recommend focusing on a fragment-style design that encourages composition over inheritance and interface implementation.

### The Long-Term Benefits

By limiting the use of inheritance and embracing modular, component-based designs, you can create a codebase that is more adaptable and easier to maintain. In the long run, this approach will save you time and effort as your project grows in scope and complexity. Unreal Engine provides a wealth of tools and patterns that support this style of development, from instanced objects and structs to actor components and interfaces.

In summary, while inheritance is a powerful feature of OOP and Unreal Engine, it's essential to be mindful of its limitations. By exploring other programming styles, such as fragment design and component-based architecture, you can build a more flexible and maintainable framework for your games. Avoid the trap of deep inheritance hierarchies and instead strive for a modular approach that will serve you well throughout the lifecycle of your projects.

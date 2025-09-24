+++
title = "About Lumina"
date = 2025-09-23T12:28:45-05:00
draft = false
+++

# Lumina Game Engine

Lumina is my ongoing game engine project written in C++. I’ve been working on it for about two years, and what started as a side experiment has turned into one of my biggest passions.

Originally, I was more interested in game development itself, making projects, prototypes, and ideas come to life. But over time, I found that what really grabbed me wasn’t the *games* — it was the *engines* behind them. Building the tools, systems, and architecture became far more rewarding than just shipping another game. 

So Lumina was born.  

---

## From Scratch (and Rewritten Again)

Like many, I began with inspiration from [TheCherno’s engine series](https://www.youtube.com/@TheCherno). The first version of Lumina followed a similar structure, but eventually, I realized it wasn’t what I wanted.  

I scrapped it and rebuilt the engine from the ground up, a painful but freeing process. That rewrite gave Lumina its current direction: a cleaner, more modular design that prioritizes learning, experimentation, and modern engine features.

---

## Key Features

- **C++ Reflection System**  
  Built using Libclang, enabling runtime type information, serialization, and editor integration.  

- **Fully integrated Vulkan renderer**  
  The renderer is one of the largest parts of any game engine, and I chose to invest my time in Vulkan.

- **Custom Editor**  
  Developed with ImGui, providing real-time control, debugging tools, and extensibility.  

- **Entity-Component System (ECS)**  
  A dedicated focus on ECS, designed not only for performance but also for clarity. ECS is often treated like magic, Lumina’s goal is to demystify it, making it practical and approachable.  

- **Tools & Extensibility**  
  From asset management to rendering abstractions, Lumina is gradually expanding into a full-fledged toolkit for engine developers and hobbyists alike.  

---

## Philosophy

Lumina isn’t just about being another engine, it’s about **learning, transparency, and accessibility**. Game engine development can feel intimidating, but the project aims to make concepts like ECS, reflection, and rendering pipelines easier to understand.  

I’m building it to be:  

- **Open Source** — available for anyone to study, fork, and use.  
- **Freely Licensed** — no paywalls or closed-off features.  
- **Collaborative** — pull requests and contributions are welcome.  

---

## Get Involved

The project is open source and actively evolving. If you’re curious about engine internals, or if you want to contribute, check it out:  

[Lumina on GitHub](https://github.com/MrDrElliot/LuminaEngine)

---

*Building games is fun, but building the tools that make them possible is where the real magic is.*

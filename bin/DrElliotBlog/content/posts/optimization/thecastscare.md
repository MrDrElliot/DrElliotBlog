+++

title = "The Cast Scare"
date = 2024-09-06
tags = ['UnrealEngine','Paradigms','Optimization']
comments = true
+++

Hello! Today I wanted to talk a little bit about casting, primarily Blueprint casting. The main reason I wanted to bring this up is because it's one of the most misunderstood and blown-out-of-proportion concepts in Unreal Engine. There's no shortage of YouTube videos with clickbait titles like "*Casting is so bad, NEVER use it in your games!*" (or some variation of that). This highlights a larger problem: people creating tutorials when they have no business doing so. But I digress.

## So what is casting? 

Casting is simply a way of converting one object type into another in Unreal Engine. In Blueprint, this is often done to access or interact with specific properties or methods on an object. For example, you might cast to a `PlayerCharacter` to access health or inventory functions from a generic object reference. 

In C++ terms, casting is essentially telling the engine, "I know this object is actually of a more specific type, so let me treat it as that type."

## Is casting expensive?

Let’s clear this up: casting itself is not expensive. The act of casting doesn’t add significant runtime cost to your game. The real performance cost comes from asset loading, not the casting process itself. When you cast to a type, if the asset (like a skeletal mesh, material, or Blueprint class) hasn’t been loaded yet, the engine will load it. That’s where you might see a performance hit, but it happens once—after that, the asset is loaded into memory, and casting has no additional cost.

A common misconception is the advice to "never cast on Tick." Let me be clear: casting on Tick is fine. It doesn’t negatively affect performance in any significant way. Want to know a secret? The engine itself casts on Tick, and even Epic’s own Lyra Starter Game casts on Tick. Why? Because it doesn’t matter. If you’re worrying about saving nanoseconds, you’re over-optimizing. Don't believe me? **PROFILE**.

So, relax—the world isn’t going to end if you cast on Tick.


This misunderstanding of casting has led to a lot of disinformation, causing new developers to shy away from it out of fear, or worse, use convoluted workarounds that don't actually solve the issue.

## What makes casting bad?

The main thing that gets casting a bad rap is the creation of **hard references**. A hard reference means that when one asset references another directly, Unreal has to load both assets into memory. This can create long dependency chains that cause your game or editor to stutter or hang, especially when many assets are involved.

However, here's the real kicker: **it's not the casting that causes the hard reference, it's the pin**. The "As XYZ" pin is what is causing the hard reference in your Blueprint, which is where the real problem lies. Many developers think they're avoiding hard references by using something like an interface or event dispatcher, but if they're still using the same reference pin, they're still creating the hard reference. The cast node itself is not the issue; it’s the context surrounding it.

## What makes casting good?

Now that we’ve cleared up some of the misconceptions, let’s talk about when casting is a good idea. When done right, casting is an effective tool to make your code cleaner and more efficient.

Here are some tips to keep your casting effective:
- **Cast to native classes whenever possible**. Native classes (C++ classes) have a lower overhead since they don't rely on Blueprint's reflection system, and don't contain any other hard references (a vast majority of the time).
- If you're working exclusively in Blueprint, **keep your base classes free of any asset references**. Avoid adding things like skeletal meshes, materials, or other large assets to the base class you're going to be casting to. This reduces the risk of inadvertently loading large assets unnecessarily.
  
  For example, if you cast to a `PlayerCharacter` class that has a skeletal mesh component attached to it, Unreal will load the mesh as soon as the class is referenced—even if you’re just trying to access a health variable. Avoid this by keeping base classes lightweight.

- You can also get a lot of these things without needing to cast to your player at all. For example, the On Overlap delegate that you commonly use in your Blueprints to receive a callback when your actor/component overlaps with something...
The *Other Actor* pin produces an actor reference. You can use **Get Component By Class* to anonymously find a component on your actor, without needing to cast to your character, or even any character.

## The Bigger Problem: Hard References

As mentioned earlier, hard references are a larger problem than casting itself. Creating long dependency chains between assets can bring the Unreal editor (or even your game) to a screeching halt. This is especially true if you’ve got multiple assets referencing one another indirectly through these chains. Managing these references is key to keeping your game performant, and it's something that requires thoughtful planning during development.

But that’s a discussion for another day.

---

In conclusion, casting has gotten a bad reputation because of misinformation. It's not inherently bad, nor is it expensive at runtime. The real issue lies in how casting can lead to hard references, especially when combined with asset-heavy classes. By understanding how casting works, and how to minimize hard references, you'll be in a much better position to use it efficiently in your projects.


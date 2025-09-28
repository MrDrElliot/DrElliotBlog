+++
title = "What Are Soft References?"
date = 2024-08-16T14:31:08-05:00
tags = ['Optimization','UnrealEngine','CPP','C++']
comments = true
+++

# What Are Soft References?

---

Soft references are a way to *reference* an object in Unreal Engine without forcing it to stay in memory. This can be incredibly useful in scenarios where you need to reference assets like textures, sounds, or even entire levels, but don’t want to load them into memory until absolutely necessary.

### Why Use Soft References?

Imagine your game has a massive library of assets. Loading everything at once would quickly consume your memory, leading to performance issues or, worse, crashing your game. This is where soft references come in handy. They allow you to keep a "pointer" to an asset, but the asset itself is not loaded into memory until you explicitly tell it to.

Think of soft references as an invitation to a party—you have the address (the reference), but you don’t actually show up at the party (load the asset) until you decide it’s time to go. This way, you avoid overcrowding (memory overload) and only arrive (load the asset) when it’s relevant to the gameplay.

### How Do Soft References Work?

Soft references in Unreal are often used with the `TSoftObjectPtr` and `TSoftClassPtr` templates. These templates allow you to reference an object or class without immediately loading it into memory. When you need the object, you simply resolve the reference, which loads the asset at that moment.

For example:
```cpp
TSoftObjectPtr<UTexture2D> SoftTexture;
```

In the code above, `SoftTexture` is a soft reference to a texture. The texture isn’t loaded until you call `SoftTexture.LoadSynchronous()`. Until then, it’s just a reference, taking up minimal memory.

### Synchronous Loading vs Asynchronous Loading

When it comes to loading assets referenced by soft references, you have two main options: **synchronous** loading and **asynchronous** loading.

#### Synchronous Loading

Synchronous loading means that the asset is loaded immediately, and the game waits until this process is complete before continuing. This is straightforward and ensures the asset is available right when you need it. However, it also means that the game will pause (or experience a hitch) while the asset loads, which can lead to noticeable frame drops or stuttering, especially if the asset is large or the game is running on hardware with limited resources.

In the earlier example:
```cpp
SoftTexture.LoadSynchronous();
```
This method blocks everything else until the texture is fully loaded, which is great for critical assets that must be available right away but can cause issues in scenarios where smooth gameplay is crucial.

#### Asynchronous Loading

Asynchronous loading, on the other hand, loads the asset in the background, allowing the game to continue running smoothly. This is particularly useful for assets that don’t need to be immediately available or for situations where you want to avoid disrupting the player’s experience.

To load an asset asynchronously, you typically use a function like:
```cpp
TSoftObjectPtr<UTexture2D> SoftTexture;
UAssetManager::GetStreamableManager().RequestAsyncLoad(SoftTexture.ToSoftObjectPath(), FStreamableDelegate::CreateWeakLambda(this, [SoftTexture]
{
    SoftTexture.LoadSynchronous(); /* Either .Get() or .LoadSynchronous(), the object is already loaded either way */
}));
```
With asynchronous loading, the game doesn't stop to wait for the asset to load, which helps maintain performance. However, this also means you need to handle the case where the asset isn’t ready when you need it. You might need to display a placeholder or delay an action until the asset is fully loaded.

### Issues with Synchronous and Asynchronous Loading

Both methods come with trade-offs:

- **Synchronous Loading**: The primary issue is the potential for hitches or pauses in gameplay, especially when loading large assets. It’s simple but can be disruptive if not used carefully.

- **Asynchronous Loading**: While it helps maintain smooth gameplay, it requires additional logic to handle the period where the asset is not yet available. If not managed well, this can lead to missing assets or awkward transitions in gameplay.

### When Should You Use Each?

The choice between synchronous and asynchronous loading depends on your game’s needs:

- Use **synchronous loading** for assets that are critical and need to be available immediately, but be mindful of potential hitches.
- Use **asynchronous loading** for assets that can load in the background without affecting the immediate gameplay experience, but ensure you have a plan for handling the absence of the asset during the load.

### When Soft References Might Not Be the Best Choice

While soft references are powerful, they aren’t always the optimal solution. Here are a few scenarios where soft references might not make sense:

1. **Frequently Used Assets**: If an asset is used frequently throughout the game, such as core UI elements, main character models, or constantly played sound effects, it might be more efficient to use a hard reference or preload the asset. The overhead of constantly resolving a soft reference might outweigh the memory savings.

2. **Critical Path Assets**: For assets that are essential to the game’s primary gameplay loop or are needed instantly during gameplay, soft references might introduce unwanted delays. In such cases, preloading or using a hard reference ensures the asset is always ready.

3. **Small Assets**: For small assets that don’t consume much memory, the benefit of using soft references diminishes. It might be simpler to keep these assets loaded rather than managing their load states through soft references.

4. **Assets Already Loaded by Other Means**: If an asset is already loaded into memory because it’s part of a larger loaded package (like a level or a character model), using a soft reference to the same asset might be redundant and unnecessary.

5. **Streaming Levels**: When using Unreal’s level streaming system, certain assets might be better managed through the level’s streaming setup rather than through soft references. Streaming levels handle the memory management for you, so adding another layer of soft reference management might complicate things unnecessarily.

### Conclusion

Soft references are a powerful tool in Unreal Engine that give you greater control over memory management and asset loading. They are ideal for managing memory in large games or for assets that aren’t always needed immediately. However, they are not a one-size-fits-all solution. For assets that are frequently used, critical to gameplay, or already managed by other systems, hard references or preloading might be a better choice.

By understanding when and how to use soft references, and recognizing when they might not be the best fit, you can optimize your game’s performance while keeping memory usage in check. This balanced approach is key to maintaining a smooth and efficient gaming experience.
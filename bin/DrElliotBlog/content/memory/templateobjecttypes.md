+++
title = "Templated Object Types"
date = 2024-08-16T14:31:08-05:00
draft = false
+++

---

Unreal Engine provides various templated object pointer types that help manage memory and references to `UObject` instances in a safe and efficient manner. These templated types are vital for ensuring the stability and performance of your Unreal Engine projects. In this post, we'll explore the most common templated object pointer types, explain what they do, and discuss when to use them.

### 1. **TObjectPtr<>**

**Purpose:** `TObjectPtr` is a smart pointer introduced in Unreal Engine 5, designed to replace raw pointers for referencing `UObject` types. It provides enhanced memory safety by automatically handling the lifetime of the referenced object. 
It was introduced in 5.0 and is now becoming the new norm, especially with the new `incremental garbage collection` being implemented in Unreal Engine 5.4.

**Use Case:** Use `TObjectPtr` when you want to ensure safe and efficient references to `UObject` instances, particularly when ownership semantics are important, and to be allow for reference tracking.

```cpp
TObjectPtr<UTexture2D> MyTexture;
```

### 2. **TWeakObjectPtr<>**

**Purpose:** `TWeakObjectPtr` is a non-owning smart pointer that references a `UObject` without preventing it from being garbage collected. It allows you to check if the object is still valid before accessing it.

**Use Case:** Use `TWeakObjectPtr` when you need a reference to a `UObject` that might be destroyed or unloaded, and you don't want to keep it alive unnecessarily.

```cpp
TWeakObjectPtr<AActor> WeakActorPtr;
```

### 3. **TSoftObjectPtr<>**

**Purpose:** `TSoftObjectPtr` is a smart pointer that holds a reference to a `UObject` asset by its path, without loading the asset into memory until it's explicitly needed. This is useful for optimizing memory usage.

**Use Case:** Use `TSoftObjectPtr` for assets that are not immediately needed, such as optional or large assets, where you want to load them only when required.

```cpp
TSoftObjectPtr<UTexture2D> SoftTexture;
```

### 4. **TSoftClassPtr<>**

**Purpose:** `TSoftClassPtr` is similar to `TSoftObjectPtr`, but it specifically references a class instead of an object. It holds the class by its path and loads it only when necessary.

**Use Case:** Use `TSoftClassPtr` when you want to reference a class that may not be needed immediately, such as a blueprint class or a specific gameplay actor class.

```cpp
TSoftClassPtr<AActor> SoftActorClass;
```

### 5. **TSubclassOf<>**

**Purpose:** `TSubclassOf` is a templated class used to store references to classes derived from a specified base class. It ensures that only valid subclasses are assigned to the variable, which is particularly useful for blueprint interactions.

**Use Case:** Use `TSubclassOf` when you need to reference a class type, especially when dealing with blueprints or dynamically spawning actors.

```cpp
TSubclassOf<AActor> ActorSubclass;
```

### 6. **TLazyObjectPtr<>**

**Purpose:** `TLazyObjectPtr` is designed to handle objects that might be unloaded and reloaded during gameplay. It maintains a reference to the object even if it is unloaded and can resolve it when the object is reloaded.

**Use Case:** Use `TLazyObjectPtr` when referencing objects that might be dynamically loaded or unloaded, such as with level streaming or large datasets.

```cpp
TLazyObjectPtr<AActor> LazyActorPtr;
```

### 7. **TScriptInterface<>**

**Purpose:** `TScriptInterface` is a templated type that holds a reference to an object that implements a specific interface. It allows interaction with the interface without knowing the underlying object type.

**Use Case:** Use `TScriptInterface` when you need to interact with different objects through a common interface, especially useful in blueprint scripting and interface-driven design.

```cpp
TScriptInterface<IMyInterface> InterfacePtr;
```

### Conclusion

Each of these templated object pointer types in Unreal Engine serves a specific purpose in managing references to `UObject` instances, ensuring safe memory management and optimal performance. Understanding when and how to use these types is essential for effective Unreal Engine development. By mastering these pointers, you can write more robust, efficient, and maintainable code.

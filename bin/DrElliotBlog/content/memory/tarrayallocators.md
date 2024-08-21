+++
title = "TArray Allocators"
date = 2024-08-16T14:31:08-05:00
draft = false
+++

# Understanding TArray Allocators in Unreal Engine

---

Unreal Engine's `TArray` is a powerful and flexible container that allows you to manage dynamic arrays in your code. One of its lesser-known yet incredibly useful features is the ability to specify custom allocators. These allocators determine how the memory for the array is allocated and managed, which can have a significant impact on performance, especially in memory-constrained environments.

I personally **highly** recommend taking a look at the `VoxelCore` library that's open source on Github, they wrote lots of incredibly performant containers that are worth a lot at.

In this article, we'll explore the different types of `TArray` allocators available in Unreal Engine, with examples demonstrating how and when to use them.

### Default Allocator (**FDefaultAllocator**)

**Purpose:** The default allocator used by `TArray` if no other allocator is specified. It provides a general-purpose, dynamic allocation strategy that works well in most cases.

**Use Case:** Use the default allocator when you need a dynamic array that grows as needed, with no specific memory constraints.

```cpp
TArray<int32> MyArray; // Uses FDefaultAllocator by default
```

### Inline Allocator (**TInlineAllocator<>**)

**Purpose:** The inline allocator allows you to specify a fixed number of elements that will be allocated inline (within the array's memory footprint) before falling back to dynamic allocation. This can reduce heap allocations and improve performance when the array is small or has a known upper bound.

**Use Case:** Use `TInlineAllocator` when you expect your array to be small most of the time, and you want to avoid heap allocations for those small cases.

```cpp
TArray<uint32, TInlineAllocator<4>> InlineArray; // Inlines up to 4 elements before allocating on the heap
```

**Example:** Suppose you have an array that typically holds 1-4 elements, but occasionally needs to hold more. The `TInlineAllocator<4>` will handle the typical case without allocating heap memory, providing a performance boost.

### Fixed Allocator (**TFixedAllocator<>**)

**Purpose:** The fixed allocator pre-allocates a fixed block of memory for the array, meaning the array cannot grow beyond the specified size. This can be useful in performance-critical situations where you need to avoid dynamic memory allocation entirely.

**Use Case:** Use `TFixedAllocator` when you know the maximum size of your array and need to ensure that no additional memory allocation occurs at runtime.

```cpp
TArray<int32, TFixedAllocator<16>> FixedArray; // Can hold exactly 16 elements, no more, no less
```

**Example:** In a scenario where you need a small, fixed-size array (e.g., an array of 16 elements for vertex data in a shader), `TFixedAllocator<16>` would provide deterministic performance with no risk of runtime allocations.

### Heap Allocator (**THeapAllocator<>**)

**Purpose:** The heap allocator explicitly allocates memory on the heap for the array, similar to the default allocator but more explicitly defined. This is useful when you want to ensure that the array's memory is always allocated on the heap.

**Use Case:** Use `THeapAllocator` when you need to guarantee that the array’s memory is allocated on the heap, regardless of its size or usage patterns.

```cpp
TArray<int32, THeapAllocator> HeapArray; // Always allocates on the heap
```

**Example:** When working with large datasets or arrays that may grow significantly during runtime, `THeapAllocator` ensures that memory is dynamically allocated on the heap, allowing the array to expand as needed.

### Special Allocators (Custom Allocators)

**Purpose:** Unreal Engine allows you to create custom allocators by inheriting from the base `FMemoryAllocator` class. This provides full control over how memory is allocated, aligned, and freed.

**Use Case:** Use a custom allocator when you have specific memory management requirements that aren't met by the existing allocators. This might include specialized alignment requirements or pooling strategies.

```cpp
class FMyCustomAllocator : public FMemoryAllocator
{
    // Custom allocator implementation here
};

TArray<int32, FMyCustomAllocator> CustomArray;
```

**Example:** Suppose you're developing a real-time system with unique memory alignment or allocation requirements. A custom allocator would allow you to implement those requirements directly within the `TArray`.

### Conclusion

Choosing the right allocator for your `TArray` can have a significant impact on your game's performance and memory usage. The default allocator is a good general-purpose choice, but when you need more control—whether to avoid heap allocations, limit memory usage, or meet specific performance criteria—inline, fixed, heap, or custom allocators can provide the solution.

Understanding these allocators and how to use them effectively can help you write more efficient, optimized code in Unreal Engine. Whether you're working on a small mobile game or a large AAA title, these tools give you the flexibility to manage memory in a way that best suits your project's needs.

+++  
title = "Move Semantics in C++ and Unreal Engine"  
date = 2024-09-30T12:28:45-05:00  
draft = false  
+++

Hello! Today, I want to talk about a commonly misunderstood concept in programming: **Move Semantics**.

Move semantics involve efficiently transferring ownership of resources (like memory) from one object to another, instead of copying the data, which adds unnecessary overhead. This is especially important in performance-critical applications such as **game development**, where proper memory management can make a big difference in efficiency. Games, just like any other software, are prone to performance pitfalls caused by poor handling of move semantics. By the end of this post, you should have a better grasp of how to avoid these inefficiencies.

We’ll cover the basics of **lvalue** and **rvalue**, explore move semantics in C++ and Unreal Engine, and show real-world examples in both **raw C++** and **Unreal Engine’s C++**.

Let’s get started!

## Understanding the Problem

Let’s first consider a raw C++ example:

### Raw C++ Code Example:

```cpp
#include <iostream>
#include <vector>

int gAllocations = 0;
int gCopies = 0;

void* operator new(size_t size)
{
    gAllocations++;
    return malloc(size);
}

struct Data
{
    int integer = 0;
    Data() = default;

    Data(int i) : integer(i) {}

    Data(const Data& Other)
        : integer(Other.integer)
    {
        gCopies++;
        std::cout << "Copied Data\n";
    }
};

void PrintVector(std::vector<Data> InData)
{
    std::cout << "Size: " << InData.size() << std::endl;
    if (InData.empty())
    {
        return;
    }

    std::cout << "Elements: { ";
    for (int i = 0; i < InData.size(); i++)
    {
        std::cout << InData[i].integer;
        if (i < InData.size() - 1)
        {
            std::cout << ", ";
        }
    }
    std::cout << " }\n";
}

int main()
{
    std::vector<Data> Numbers;
    for (size_t i = 0; i < 5; i++)
    {
        Numbers.push_back(Data(i));
    }

    PrintVector(Numbers);

    std::cout << "Allocation: " << gAllocations << std::endl;
    std::cout << "Copies: " << gCopies;

    std::cin.get();
}
```

### The Output:

```
Copied Data
Copied Data
Copied Data
...
Size: 5
Elements: { 0, 1, 2, 3, 4 }
Allocation: 8
Copies: 20
```

Here, we have **8 allocations** and **20 copies** of `Data`. This is far from optimal and can be a performance bottleneck, especially when dealing with large datasets.

#### Why Are There So Many Copies?

1. **Passing the Vector by Value**:  
   The function `PrintVector` takes the `std::vector<Data>` by value, meaning that the vector and all its elements are copied when passed. Each copy of a `Data` object triggers the copy constructor, which adds unnecessary overhead.

2. **Multiple Allocations**:  
   As `push_back` is called, the vector resizes its internal storage, leading to multiple reallocations. Each reallocation can result in copying existing elements into new memory blocks, further adding to the overhead.

## Fixing the Code with Move Semantics

We can improve the performance by using **move semantics**. Let’s look at an optimized C++ version and then explore how Unreal Engine’s `TArray` can provide similar behavior.

### Optimized Raw C++ Code:

```cpp
#include <iostream>
#include <vector>

int gAllocations = 0;
int gCopies = 0;
int gMoves = 0;

void* operator new(size_t size)
{
    std::cout << "Allocated: " << size << " bytes\n";
    gAllocations++;
    return malloc(size);
}

struct Data
{
    int integer = 0;

    Data() = default;
    Data(int i) : integer(i) {}

    Data(const Data& Other)
        : integer(Other.integer)
    {
        gCopies++;
        std::cout << "Copied Data\n";
    }

    Data(Data&& Other) noexcept
        : integer(Other.integer)
    {
        gMoves++;
        std::cout << "Moved Data\n";
        Other.integer = 0;  // Optional: Invalidate the source object
    }
};

void PrintVector(const std::vector<Data>& InData)
{
    std::cout << "Size: " << InData.size() << std::endl;
    if (InData.empty())
    {
        return;
    }

    std::cout << "Elements: { ";
    for (int i = 0; i < InData.size(); i++)
    {
        std::cout << InData[i].integer;
        if (i < InData.size() - 1)
        {
            std::cout << ", ";
        }
    }
    std::cout << " }\n";
}

int main()
{
    std::vector<Data> Numbers;
    Numbers.reserve(5);

    for (size_t i = 0; i < 5; i++)
    {
        Numbers.emplace_back(i);
    }

    PrintVector(Numbers);

    std::cout << "Allocations: " << gAllocations << std::endl;
    std::cout << "Copies: " << gCopies << std::endl;
    std::cout << "Moves: " << gMoves << std::endl;

    std::cin.get();
}
```

### The New Output:

```
Allocated: 16 bytes
Allocated: 20 bytes
Size: 5
Elements: { 0, 1, 2, 3, 4 }
Allocations: 2
Copies: 0
Moves: 0
```

### What Changed?

1. **Passing by Const Reference**:  
   Passing the vector by `const std::vector<Data>&` avoids copying the entire vector and its elements when calling `PrintVector`.

2. **Using `reserve()`**:  
   Pre-allocating memory with `reserve(5)` prevents reallocations as we insert elements, reducing the number of allocations to two.

3. **Using `emplace_back()`**:  
   This avoids creating temporary objects and thus reduces unnecessary copying or moving.

---

## Move Semantics in Unreal Engine

Unreal Engine’s `TArray` is the engine's equivalent to `std::vector` in C++. Much like in raw C++, you can benefit from move semantics when working with `TArray` to avoid unnecessary copies and allocations.

### Unreal Engine Example:

```cpp
#include "CoreMinimal.h"

int32 gAllocations = 0;
int32 gCopies = 0;
int32 gMoves = 0;

void* operator new(size_t size)
{
    gAllocations++;
    return FMemory::Malloc(size);
}

struct FData
{
    int32 Integer = 0;

    FData() = default;
    FData(int32 i) : Integer(i) {}

    FData(const FData& Other)
        : Integer(Other.Integer)
    {
        gCopies++;
        UE_LOG(LogTemp, Warning, TEXT("Copied Data"));
    }

    FData(FData&& Other) noexcept
        : Integer(Other.Integer)
    {
        gMoves++;
        UE_LOG(LogTemp, Warning, TEXT("Moved Data"));
        Other.Integer = 0;  // Optional: Invalidate the source object
    }
};

void PrintTArray(const TArray<FData>& InData)
{
    UE_LOG(LogTemp, Warning, TEXT("Size: %d"), InData.Num());
    if (InData.Num() == 0)
    {
        return;
    }

    FString Elements = TEXT("Elements: { ");
    for (int32 i = 0; i < InData.Num(); i++)
    {
        Elements += FString::FromInt(InData[i].Integer);
        if (i < InData.Num() - 1)
        {
            Elements += TEXT(", ");
        }
    }
    Elements += TEXT(" }");

    UE_LOG(LogTemp, Warning, TEXT("%s"), *Elements);
}

void TestMoveSemantics()
{
    TArray<FData> Numbers;
    Numbers.Reserve(5);  // Pre-allocate memory to prevent reallocations

    for (int32 i = 0; i < 5; i++)
    {
        Numbers.Emplace(i);  // Emplace to avoid copies
    }

    PrintTArray(Numbers);

    UE_LOG(LogTemp, Warning, TEXT("Allocations: %d"), gAllocations);
    UE_LOG(LogTemp, Warning, TEXT("Copies: %d"), gCopies);
    UE_LOG(LogTemp, Warning, TEXT("Moves: %d"), gMoves);
}
```

### The Unreal Engine Output:

```
Copied Data
Copied Data
...
Size: 5
Elements: { 0, 1, 2, 3, 4 }
Allocations: 2
Copies: 0
Moves: 0
```

### Key Differences

1. **`TArray` vs `std::vector`**:  
   Unreal Engine uses

 `TArray` instead of `std::vector`, but the concepts remain the same. You can pre-allocate memory with `Reserve()` and use `Emplace()` to construct elements in place, just like in C++.

2. **Unreal Engine Logging**:  
   Instead of `std::cout`, Unreal Engine uses the logging system (`UE_LOG`) to output messages.

3. **Memory Allocation**:  
   Unreal Engine uses `FMemory::Malloc()` for custom memory management, replacing the raw `malloc()` used in the C++ version.

---

## Conclusion

By understanding and properly applying **move semantics**, you can drastically reduce unnecessary allocations and copies in your programs, leading to better performance. This is especially important in performance-critical applications like game development, whether you're using raw C++ or Unreal Engine’s `TArray`.

In our examples, both in C++ and Unreal Engine, we reduced the number of allocations and eliminated unnecessary copies by:

1. Passing containers by const reference.
2. Pre-allocating memory to prevent reallocations.
3. Using `emplace_back()` in C++ or `Emplace()` in Unreal Engine.

Mastering these techniques will help you write more efficient code in any game development environment!

Thanks for reading! Hopefully, you now have a better understanding of move semantics and how to apply them in both raw C++ and Unreal Engine.
+++
title = "Constexpr and Consteval"
date = 2024-08-16T13:46:48-05:00
draft = false
+++

# Simple way to possibly increase some performance.

This blog is a follow-up to this YouTube video: https://www.youtube.com/watch?v=8-VZoXn8f9U

Consider the following code:

```cpp

// ConstexprExample.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <chrono>

int Fibonacci(int n)
{
    if (n <= 1) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}

constexpr int Fibonacci_C(int n)
{
    if (n <= 1) return n;
    return Fibonacci_C(n - 1) + Fibonacci_C(n - 2);
}

int main()
{
    auto Start = std::chrono::high_resolution_clock::now();
    constexpr int num = 25;

    constexpr int result_c = Fibonacci_C(num);

    std::cout << "Constexpr Fibonacci: " << result_c << '\n';
    std::cout << "Time Taken: " << std::chrono::duration_cast<std::chrono::microseconds>(std::chrono::high_resolution_clock::now() - Start).count() << "\n";

    Start = std::chrono::high_resolution_clock::now();

    int result = Fibonacci(num);

    std::cout << "Normal Fibonacci: " << result << '\n';
    std::cout << "Time Taken: " << std::chrono::duration_cast<std::chrono::microseconds>(std::chrono::high_resolution_clock::now() - Start).count();

}
```

## Performance Differences

Here we see we are evaluating a classic Fibonacci recursive function. Recursion like this can be incredibly expensive. On the order of O(2^n). 

When comparing the two versions of the Fibonacci function—one using a normal recursive function (`Fibonacci`) and the other using `constexpr` (`Fibonacci_C`)—we can observe significant performance differences, especially as the input size increases.

### `Fibonacci` (Normal Function)
The `Fibonacci` function is evaluated at runtime. Every time the program is executed, the function goes through the recursive calls to compute the result, which can be quite costly. For example, with `n = 25`, the function performs thousands of recursive calls, resulting in significant computational overhead. As the input grows, the execution time increases exponentially, which is evident from the output when you measure the time taken for the function to compute the result.

### `Fibonacci_C` (Constexpr Function)
On the other hand, the `Fibonacci_C` function is a `constexpr` function. If the input is known at compile time (as in this case, with `num = 25`), the compiler can evaluate the function during compilation. This means the result is computed once, and the precomputed value is embedded directly into the binary, leading to a drastic reduction in runtime computation. The time measured in this scenario is almost negligible because the function itself isn't actually being executed at runtime—the value is already there.

### Output Comparison

The difference in time taken for the two approaches might look something like this:

```
Constexpr Fibonacci: 75025
Time Taken: 211 microseconds
Normal Fibonacci: 75025   
Time Taken: 620 microseconds

```

These numbers will grow exponentially the larger the input number (number of recursive functions) is.

In this hypothetical output, we see that the `constexpr` version takes effectively no time at runtime because the work was done at compile time, whereas the normal function takes a considerable amount of time due to the nature of recursion.

## `constexpr` vs. `consteval`

### `constexpr`
- **Definition**: `constexpr` is a keyword introduced in C++11 that suggests to the compiler that the function can be evaluated at compile time. However, if the inputs are not known at compile time, the function will be executed at runtime.
- **Usage**: Use `constexpr` when you want a function to be evaluated at compile time if possible, but you're okay with it being evaluated at runtime if necessary.
- **Pros**:
  - Flexibility: It allows both compile-time and runtime evaluation.
  - Performance: If evaluated at compile time, it can reduce runtime overhead.
- **Cons**:
  - Uncertainty: There’s no guarantee that the function will always be evaluated at compile time, depending on the context in which it is used.

### `consteval`
- **Definition**: `consteval`, introduced in C++20, forces a function to be evaluated at compile time. If you try to call a `consteval` function with inputs that are not known at compile time, the code will not compile.
- **Usage**: Use `consteval` when you want to ensure that a function is always evaluated at compile time, without exception.
- **Pros**:
  - Guarantees: You can be certain that the function is never evaluated at runtime, which can be crucial for certain performance-critical or safety-critical applications.
- **Cons**:
  - Rigidity: You lose the flexibility of runtime evaluation. If the inputs aren't known at compile time, you can't use a `consteval` function.

## When to Use `constexpr` vs. `consteval`

### When to Use `constexpr`:
- When you have functions that could benefit from compile-time evaluation but may also need to support runtime evaluation depending on the context.
- For code that is part of a library where flexibility is important—allowing the user to decide if they want compile-time or runtime evaluation.

### When to Use `consteval`:
- When you require the result to be known at compile time, such as in the case of template metaprogramming or when defining constants that must be computed at compile time.
- For performance-critical code where runtime computation would be prohibitively expensive, and you want to guarantee compile-time computation.

## Conclusion

Understanding the differences between `constexpr` and `consteval` can significantly influence the performance of your code. `constexpr` offers flexibility but leaves the decision to the compiler, while `consteval` provides certainty at the cost of flexibility. Depending on the specific needs of your application—whether you prioritize performance, compile-time guarantees, or runtime flexibility—choosing the right tool for the job can lead to more efficient and maintainable code.

As always, profile your code and consider the specific context in which you're working to make the best decision.

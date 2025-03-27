+++
title = 'Fun with V-Tables'
date = 2025-03-27T12:34:23-05:00
draft = false
+++

# Fun with V-Tables!

---

* Have you ever wanted to access a function, but you couldn't because the function is private? :/
* Have you wanted to *replace* a function in a class?

Well, look no further! Today we're going to be talking about V-Table manipulation.  
Please note, this is purely for fun and can be dangerous to do in production environments. I don't recommend using it for actual runtime use, but it can be an interesting exercise in how memory manipulation works and how you can use it to your advantage.

---

## What is a V-Table?

A **Virtual Table** (or V-Table) is an essential concept in C++ for implementing **polymorphism**. It’s a data structure used to support dynamic dispatch for virtual functions. When you define a class with virtual functions, the compiler generates a V-Table for that class to keep track of the addresses of those virtual functions. At runtime, the V-Table is used to decide which function to call based on the type of the object, allowing polymorphism to work correctly.

### How does it work?

Here’s how a V-Table works:
- When a class has virtual functions, the compiler creates a special **V-Table** for it.
- Each entry in the V-Table points to a corresponding virtual function for that class.
- When a virtual function is called on an object, the program uses the V-Table to look up the function pointer, then calls the function based on the correct type.

In other words, when you have a base class with a virtual function, the V-Table is what makes sure that the right derived class function gets called at runtime, even if the actual object type is different from the type of the reference or pointer.

---

## Sniping the V-Table

Now, here’s where the fun begins. What if you wanted to **replace** a function in a class at runtime? Could you do that? With a little bit of memory manipulation, you can! Let's take a look at how we can do this in code:

We’ll modify the V-Table for an object and replace the address of the virtual function with one of our own choosing.

```cpp
#include <iostream>
#include <Windows.h>

class SomeBase
{
public:
    virtual const char* VirtualFunction()
    {
        return "Hello I'm virtual";
    }
};

const char* CustomFunction()
{
    return "Hello, I've been replaced!";
}

int main()
{
    SomeBase* obj = new SomeBase(); // Allocate on the heap instead of stack

    // Access the VTable by treating the object as a pointer to a pointer
    void** vtable = *(void***)obj;

    // Change memory protection to allow write access to the VTable
    DWORD oldProtect;
    VirtualProtect(&vtable[0], sizeof(void*), PAGE_EXECUTE_READWRITE, &oldProtect);

    // Replace the first entry in the VTable with our custom function
    vtable[0] = (void*)&CustomFunction;

    // Call the virtual function (it should now call our custom function)
    std::cout << obj->VirtualFunction() << std::endl;  // Expected output: "Hello, I've been replaced!"

    // Restore the original memory protection
    VirtualProtect(&vtable[0], sizeof(void*), oldProtect, &oldProtect);

    delete obj; // Clean up the heap-allocated object

    return 0;
}
```

### Explanation:
1. **V-Table Access**: We first access the V-Table for the object by treating the object as a pointer to a pointer (`void** vtable = *(void***)obj;`).
2. **Memory Protection**: To modify the V-Table, we use `VirtualProtect` to change the memory protection of the V-Table to `PAGE_EXECUTE_READWRITE`, allowing us to overwrite the function pointer.
3. **Replacing the Function Pointer**: We replace the first entry in the V-Table (which corresponds to the `VirtualFunction`) with our own custom function.
4. **Call the Function**: When we call `obj->VirtualFunction()`, it calls our custom function instead of the original virtual function.
5. **Restore Protection**: Finally, we restore the original memory protection using `VirtualProtect` again.

---

## Why Does This Work?

This works because C++ uses a V-Table to handle dynamic dispatch for virtual functions. By replacing the function pointer in the V-Table, we can redirect calls to our own function. This is a form of "sniping" the V-Table, where we modify an object's internal function dispatch table to point to a different function.

---

## A Word of Caution

While it’s fun to play around with V-Table manipulation, it's important to note that doing this in real applications can lead to unpredictable behavior, crashes, or memory corruption. This technique can easily break if the compiler or platform changes the way it handles V-Tables, and the object model may vary based on compiler settings, platform, or even optimization flags.

Here are a few things to consider:
- **Stability**: Replacing function pointers in the V-Table can destabilize the program, especially when you're dealing with polymorphic classes or objects with complex inheritance hierarchies.
- **Compatibility**: This code might break on different compilers or platforms, as not all compilers handle V-Tables the same way.
- **Safety**: Modifying V-Tables directly can result in **undefined behavior** and is generally discouraged in production code. Use this technique only for educational purposes or in very controlled environments.

---

## Conclusion

V-Table manipulation is a powerful, albeit dangerous, technique that allows you to modify the behavior of classes at runtime. Whether you want to replace a function for debugging, hacking, or learning purposes, this technique offers an interesting look under the hood of C++'s polymorphism.

That being said, **use this power wisely** and remember, there’s a fine line between hacking and hacking yourself into a corner!

Happy coding, and have fun with V-Tables!


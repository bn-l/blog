---
title: "Boxing vs unboxing & stack vs heap in csharp"
description: "Conversion from a 'reference type' to a value type and vice versa and what happens on a lower level with memory"
date: 2022-12-09
draft: false
slug: box-unbox-stack-heap-csharp
---

<span class="summary">**Summary:** Boxing converts a type from fixed memory size (stack memory) to a flexible memory version (on heap memory)—with all the potential benefits / detriments</summary>


Firstly the two major memory stores:

## Stack

- The fixed, **non-flexible** memory block
- Memory for this is allocated at compile time 
  - Parser can do this because stack types are fixed in size
- Works using <abbr title="Last In First Out">LIFO</abbr> 
- Has the "stack pointer": a register[^1] that stores the memory address of the last thing added to the stack. It's like a sliding indicator on a ruler. When more data is added it slides up to the last item. 
- Stack works with the concept of a "frame": A "frame" of data:
  1. A function puts its return address[^2] on the stack at the "stack pointer". Pointer is then incremented.
  2. The function then adds its parameters and local variables. Pointer is further incremented.
- Stores references to objects that are on the heap

## Heap

- The **flexible** part of the memory
- Garbage collected[^3] (unlike stack memory).
- The heap will grow upwards (towards high memory addresses) while the stack usually doesn't grow (downwards) ([diagram](https://archive.ph/S4AA7))
- Flexible in size because an object on the heap can grow 
  - Memory must be allocated, therefore, at runtime, because who knows how much memory the object might use.
- This where a boxed type, that was previously on the stack, is put

Two common types that are involved in boxing and unboxing:

## Value Type

- Contains an instance of the object itself (not just a reference, it's the real thing), so:
- Modifying one value type will not affect others (unlike references)
- A value type takes up a fixed amount of memory and is usually stored on stack[^4]. With some exceptions:
  - A variable inside a method: stack
  - Method parameter: stack
  - Declared as a class member: on the heap with its parent
  - Declared in struct: Wherever the struct is stored (stack or heap) [^5]
- Unboxing extracts a value type from a reference type
- Interface types are also stored on the stack and can be boxed
- Boxing converts this to a reference type that takes up a flexible amount of memory

## Reference Type

- Starts with a *reference* on the stack containing a memory address to somewhere on the heap where an object is stored
- Many of these references can point to a single object
  - So when modifying a reference type, you modify the underlying object other references might be also pointing to.
- When passed in as parameters, unlike value types, merely the reference will be given. With value types the whole value will be copied and passed as a parameter.

## More

- Boxing is implicit, unboxing in explicit:
  - I.e. boxing is something csharp does automatically, but you have to ask it to unbox.
- Boxing is computationally expensive:
  - A new object needs to be created
  - type needs to be casted (converted)

Note: Feel free to contact if you would like to make a suggestion

[^1]: Register: Special fast but small memory available to the CPU 

[^2]: Return address: A function or a **sub**routine is like a path going off the main line of code execution (routine). The return address is the point at which you took a detour. When you've finished your business you then know to go back to that location and continue on your journey. A function can also call another function and in this case you taking a detour on your detour. **Unwinding** is like going back along the branching detour(s) and picking up the thing you were working on before taking another path. You then go back to the main thread holding all the things you made on the detour(s) and dump them out on it. E.g. a recursive function. 
[^3]: In garbage collection languages, the language implementation takes care of managing memory (deleting where necessary)
[^4]: The memory needed for a value type on the stack can be calculated at compile time because it's a set amount. I.e. it's possible to block out the memory the thing needs because it's not going to change at runtime.
[^5]: A struct can be **constrained** to the stack by putting the word "ref" in front of it when declaring it. [For example](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct)
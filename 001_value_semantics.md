Value semantics is one of the core principles of C and C++ 

I would even go so far as to say that this is the most important part of C++ in terms of its design.

In the following discussion, I will ignore everything in the language that lets you bypass the value semantics. You have to learn the rules before you understand why and where thy can or should be bypassed. The less you bypass it, the more robust and efficient and idiomatic your program gets. 

---

**Types**

Every C++ type (Lets ignore void for now) is considered to be a string of bits of certain size.
Practically speaking, the granularity is usually at the byte level and a byte is defined as a certain number of bits ( typically 8 ). We will not go into the pedantics of uncommon and obsolete systems which use 6 bits per byte or other such quirks.

Every type has a certain set of operators defined on it, and most operators can be overloaded for all types. 
The few ones that you cannot override are:

- :: or the scope resolution operator
- ?: or the ternary operator
- dot operator ( this will change soon)

---

**Values** 

There are many academic CS discussions as to what exactly a value is - the simple way to think of it is that a value is an actual string of bits residing in physical memory or some hardware element like a register or even I/O port

Physical memory does not really have a notion of types - all it has is the ability to treat 1 or more bytes as a unit of computation. Our beloved x86_64 processors can treat a location in memory as 1, 2, 4, 8 byte integers (and even 128, 256, 512 as SIMD, but that's irrelevant here) 

In C++, there are two kinds of values : r-values and l-values

- l-values have a program visible address in memory where they live, and have a lifetime. l-values often have a name, but not always.
- r-values are temporary things that cease to exist past the expression in which they appear

```int x = 10; // x is l-value, 10 is r-value
int arr[10];
arr[2] = 1; // arr[2] is the l-value```

The predefined operators for all types work in the same way.
Aggregate objects like structs are values too.

```struct Point
{
    int x, y;
}```

This is treated as a chunk of memory that is >= sizeof(int) * 2

*[Note]*

C arrays are a historical glitch in the design of the language, and they break this rule of treating all values in the same way. For now ignore C based arrays, we will deal with them later and if possible rarely ever use them in our code. The semantics of C arrays are the biggest screw up of the language, that we have to live with today, because it's too late to change things. 

---

**Value semantics in assignment**

When a value is assigned to another, a bit identical copy is made from the source memory to the destination memory.  This is fundamental.

If you write :
```x = y;```
And x and y are the same type and the = operator is not overloaded, after this statement, it means that the memory at the place where x resides is bit for bit identical with that where y resides, upto sizeof(x) bytes.
There are no exceptions to this rule (except for the array snafu)

This is not true of any other mainstream language as far as I know (perhaps traditional Pascal )

Value semantics means that two names can never refer to the same block of memory (we will see how to bypass value semantics when really needed)

---

**Value semantics in function calls**

```int fn(int y)
{ 
    y++;
    return y; 
}

int x = 10;
int z = fn(x);```

When fn(x) is called, an independent copy of x is created, and that is the object called y inside of the function.

Once again there is no exception to this rule (except for C arrays - O GOD WHY!)

---

**The value of value semantics**

In almost all languages, the lifetime of an object depends on the lexical scoping of its name.
If a variable name is visible, the contents of that variable had better be accessible and alive in memory - this is common sense.

In C++ we have several scopes:
Global scope - the names of these are either visible in the file they are declared, or across the entire program (using extern)
Function and block scope - whenever you see a pair of braces {} - a new scope exists. Names defined within that {} are not visible outside of it.
Class/Struct scope - The names of members inside a class are visible inside its methods and they are also visible within the scope where a value of that class has its own name. Classes can define more classes within them and the same rules apply.

You can see that scopes form a tree hierarchy of visibility of names. 

Objects' memory is allocated when you declare them, and the memory is reclaimed when the object goes out of scope - obviously, if the name of a variable "expires", that memory is no longer reachable in the program, so it must be reclaimed.

Since scope is a tree, it should be clear that no variable in an intermediate scope ever dies before one in a deeper scope. Therefore, memory is always allocated and deallocated in a LIFO (last in first out) manner.

If y was declared  and x was declared, x has to die before y. This is unconditional.
LIFO structude automatically(pun intended) means that a stack can be used to store all the values in your program (except the outermost scope, we will ignore that for now)

A stack is the most efficient memory allocation strategy that can ever exist. Allocation is one increment instruction, deallocation is one decrement instruction. 

The LIFO lifetime of allocated variables also ensures that no memory is ever left hanging for even 1 cycle more than it needs to.

---

**Use value semantics everywhere**

Any data type or object you design must adhere to value semantics, like the rest of the language. 
You can see the extreme application of value semantics in pure functional languages, where all objects are immutable, and can only modified by creating a new  copy.

Some quite famous libraries like OpenCV break this fundamental paradigm of C++ 

```cv::Mat x = somehowCreateMat();
cv::Mat y = x;
// Now x and y refer to the same location in memory```

This is a horrible horrible mistake and unless you have some really really great reasons to, you should never make your objects behave like this in C++.

In the next post we will explore pointers, the first tool that lets you control value semantics in a consistent way, and start understanding "No sidewheels/ninja mode".

Later we will explore references, classes, move semantics, smart pointers  etc. and how we can preserve value semantics externally and keep our programs fast and robust, while still breaking the rules inside the implementations.

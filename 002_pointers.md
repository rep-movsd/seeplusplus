**Pointer semantics**

Life would be really simple if we could live with only values. The world works like that if you exclude language - there are things... and that's it. Functional languages "work" like this, but ironically are far removed from the real world.

In software, as opposed to the world, a given thing often needs to have multiple names - or rather, two or more names refer to the same thing. This requirement was referred to as "reference semantics" in older languages.

--

**The core problem**

Functions need to be able to modify their parameters in a way that's visible to the caller - let's follow the common sense reasoning why this came into being.
The first lessons programmers learned was to never repeat code - so instead we put repeating code into a thing called a procedure, and just used the name of the procedure when we needed that code to repeat.
There are three kinds of chunks of code that you can convert to a procedure and simplify a longer bot of code.

- The simplest kind is code that is context free like ```PRINT "HELLO"``` - that's easy - plain text substitution or macros can do that.
- The next kind is code that operates on one or more values and returns another value like ```sqrt()``` and so forth - this is similar to the defintion of function in math
- To make it universal, the third kind is needed, where a chunk of code can refers to many variables in its lexical scope, and you want to extract it into a reusable procedure. The procedure needs some way of being able to _refer_ to and modify the values in the lexical scope where it is invoked. To this end, reference parameters are the first solution.

In C however, everything is a value, and we're loath to give up that notion. 
If a parameter is passed to a function and the function has to modify it so the caller sees the change, then the function needs to know information privy to the caller - as to where that original variable exists in memory.

That "where something exists in memory" is nothing but the hardware concept of adresses. Addresses are nothing fancy, just think of them as an integer index into the entire adressable memory of the system.
C, being a "high level assembler" killed two birds with one stone, by using addresses - For one, you could do low level memory access, and then you could also pass addresses of values to functions, as values in turn and get reference like semantics without messing up the value semantics we had.

--

**Pointers**

A pointer is typically an integral typed value, which represents an address in memory. 
A pointer is always associated with another type - we say "Pointer to (type) T" or ```T*```.
A pointer itself is a value that has the similar semantics as say integers.
Since a pointer is a value type variable, it can itself be a type pointed to - so we can have pointer to pointer to T - ```T**``` and so on ad infinitum

C supports the following pointer operations:
- Dereference with the * operator: evaluate the contents at the address in the pointer as an r-value of type T
- Perform + and - operations on the address stored in the pointer variable.
- Retrieve the memory address of a variable with the & operator.

All pointers to data types have the same size - this is obvious since all pointers are memory addresses.
Even though pointers are like simple memory indices, pointers to different types may not be compatible.
This is because of hardware memory implementation limitations. 

For example (assuming 4 byte integers):
```
char x[] = {'H', 'a', 'h', 'a'}; //This variables value is at an address which is aligned to a level needed by char
int *px = (int *)&x;  // We just forced the compiler to let us treat that value as a 4 byte integer

// The following is undefined - we cant say if the hardware can read a 4 byte integer at that address
// Perhaps the address is not at an alignment that is OK for reading an int. A hardware exception (or TRAP) may occur
cout << *px << endl;  
```

You can always reinterpret cast any ```T*``` as a ```char*``` and you can also store any ```T*``` inside a ```void*``` 

C allows you to assign a ```void*``` to ```T*``` but C++ does not - you have to explicitly cast it.

--

**Iterators**

This is a good time to diverge into an iterators, briefly.
An iterator is the generalized version of what a pointer is - just like pointers, iterators support the following ops
- Dereference with the * operator: evaluate the contents that the iterator points to as an r-value of type T 
- Perform some arithmetic 
    - ForwardIterators support ++
    - BidirectionalIterators support ++ and --
    - RandomAccessIterators support + and -
    - OutputIterators "advance" when you dereference and write - they have no arithmetic

Every pointer is an iterator.

--

**Arrays and Pointers - The ugliest part of C**

C was written to be as close to the metal as possible. So they discarded all notions of runtime checking of any operation - whether it's integer overflow, accessing inacessible memory or dividing by 0, there is never any code generated to check this at runtime.

Almost all other languages do this, at a very, very high runtime cost.

Since we only wanted the fastest thing rather than the safest, C arrays have no size information tacked to them. A C array is just a chunk of memory. The compiler knows its size when it was declared, but for the most part ignores that in any generated code.
You can ask for an arrays byte size with ```sizeof(arr)``` and get the length with ```sizeof(arr)/sizeof(arr[0])```

Remember that ```sizeof``` is a compiletime operator - it's like a macro that evaluates to an actual constant.
As long as that array variable was in scope, ```sizeof``` works.


Now all this would be perfectly fine, except that if we need to pass arrays as parameters to functions, then we have a problem.
An array is only its contents, there is no size info, so if we pass those values to a function, the function cannot know where the array ends.
If it cannot know where it ends, you cannot keep any parameter in memory after the array parameter (including another array), so that seems like a big limitation. Remember that during runtime, all variables are only addresses offset from some base.

The "solution" was to say that when you have ```int arr[10];``` then when you write just ```arr```, the compiler treats it as ```&(arr[0])```.
Meaning that the name of an array when used outside of its declaration, _decays_ to a pointer to its first element.

Now we can happily pass any number of array and other parameters to a function and it's up to the programmer to ensure that they pass the size also along so we can iterate over that array safely.

The nasty side effect of this is that we lost value semantics, but also that it is not at all obvious.
If we saw the following code:
```
int fn(int a[10])
{
    int i;
    for(i = 0; i < 10; ++i)
    {
        printf("%d : %d\n", i, a[i]);
    }

    return a[9];
}
```

The common sense guy might think - Oh well, this function is constrained to accept only int arrays of length 10.

But then this code compiles without any complaint:

```
int main()
{
    int k[5] = {1, 2, 3, 4, 5};
    return fn(k);
}
```

```fn(k)``` looks like were passing some data as a value, and that data seems to be an array. But actually were not passing that array. We are passing a pointer to the start of the array, and so the nice world of value semantics is doomed.

We might have as well declared fn as ```int fn(int *a)``` and it would make no difference to the meaning of the program


The best advice to any C++ programmer is :
- Use a ```std::array``` rather than a C array
- If not, use a ```std::vector```
- If you do use a C array, never pass it as a parameter

There is just too much bad code and resulting grief from using C array parameters. Don't do it in C++!

--

**Valid uses of pointer semantics**

Alexander Stepanov says that the acid test for any language is whether you can write a totally generic ```swap``` function in it.
How's that for C++ love eh?

You can ponder that and check if your favorite language can do that simply, but in any case, in the C world, the only way to make a function mutate its arguments is by passing pointers to them.

There are two legit uses for pointers passed to functions : 
- Passing a pointer to a large object to avoid copying it, like a struct or a buffer (as long as you pass the size with it)
- Passing a pointer to something that the function modifies

If you follow a couple of simple rules when using pointers, you can mostly avoid the perils
- Never return a pointer from a function 
- Never pass a pointer to a function whose pointee type (and size) is unknown to the function

There are rationalizations for these recommendations:
- One must always make sure that a pointer does not outlive the thing it points to - passing a pointer out from a function does just that. Even if it is dynamically allocated memory, you just bought yourself potential trouble at a discount. If pointers are passed into functions but not out, then they always point to stuff that outlives them.
- Passing a pointer to a buffer/array is a prime candidate for messing up - as we discussed above. Pointers to primitive types and structs have some safety, as opposed to array pointers.


These rules are impossible to follow always in plain C, thus the language is inherently unsafe. 
However in C++ we need not get into this C cesspool unless we really need to.


References, smart pointers, templates and so on can let us avoid this nastiness, as we will explore in the forthcoming posts.


--


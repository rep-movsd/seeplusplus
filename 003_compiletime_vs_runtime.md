## Compiletime versus runtime

It's very important to realize what part of your code is instructions to your compiler and what part becomes instructions to the computer. 

A proper understanding of this is essential to reading and writing C++ code.

---

**Compile time**

- **Constants** - Any arithmetic operation involving constants gets converted to a constant by the compiler.

  ```int a = 512 * 1024;  // a gets 524288 ```

- **sizeof()** - sizeof evaluates only the type of the expression you pass it. It does not cause any code to be generated at runtime

  ```
  int x = sizeof(int);
  int *px = NULL;
  int y = sizeof(*px); // pf is NOT dereferenced at runtime
  ```

- **:: operator** - This is a merely a hint to the compiler as to which scope to look for a symbol in

- **\# preprocesor directive** - This happens way before the compiler even touches the code - text subsitution and arithmetic only

- **templates** : templates are 100% compiletime - when a template is instantiated, it serves as instructions to the compiler to generate code internally based on the template declaration.

- **lambdas** : Again these are syntactic sugar over function objects, so a lambda is instructions to the compiler to generate a class with an overloaded () operator. The body of lambdas is of course runtime code.

---

**Both Compile and Runtime**

- **The & reference operator** : When you use a reference to alias a variables name, the compiler generates no extra code for this. When you use a reference as a parameter or return value, it works as though you used a pointer and runtime semantics are involved
    ```
    int a = 1;
    int &b = a; // No code is actually generated here, b is just another name for a during compiletime
    ```
    

- **The dot and arrow operators** : The right hand side of the dot operator is a symbolic name that only exists at compile time. It does lead to code generation in the sense that the compiler calculates the addresses of member variables by adding a fixed offset, but it is irrelevant to the runtime state of the application. If the standard implements the proposal to allow overloading of this operator, it can lead to an actual function call at runtime. 
  The arrow operator works in a similar fashion to the dot operator, it may involve dynamic polymorphism, since the exact type of the object upon which it is being called can vary at runtime. The dot operator can also act the same way, if we use references. In the cases where the compiler can determine the type of the object at compiletime, extra code is not generated

- **The & addressof operator** : During runtime there are no variable names, so the meaning of & being "give me the address of a variable" is meaningless. But it still has the meaning "give me the address of this l-value". Obviously r-values may or may not be actually present in actual memory, whilst l-values always can be assumed to be in some storage (except if they are optimized away). So & is somewhat of a hybrid operator, tending to be more compiletime than runtime. 
  
- **constexpr** : constexpr is the new const - A constexpr expression or function is attempted to be evaluated at compile time, and if successful, it becomes a compile time hardcoded value. If the compiler finds that it cannot do the computation at compiletime, it adds a runtime code to do it.
  For example: 
  ```
  constexpr int sum(int x, int y)
  {
      return x+y;
  }


  int main(int argc, char **argv)
  {
      constexpr int i = sum(10, 10);
      int j = sum(argv[0][0], argv[0][1]);
      
      return i + j;
  }
  ```
  In the above code, i gets assigned the constant 20 directly while the j initialization causes the compiler to add a function call. If j was declared constexpr, the compielr complains that the value is not known at compile time.

- OO and dynamic dispatch - When you have static polymorphism, any method call is merely a plain function call. When it is dynamic, there are layers of indirection added for the code to determine the runtime type of the object. 



Almost everything else in C++ is runtime semantics.

---

When you look at a line of code, just ask yourself this question : 

_Can the meaning of this change based on the runtime state of the program at this point or not?_

If the answer is yes then its a runtime construct, otherwise it is a compiletime construct. Remember that any constant, constexpr variable or function is (and must be known) at compiletime.

The more stuff you do at compiletime (including code generation via macros or templates), the less you rely on work done at runtime and possibly make your program faster.

For example C++ ```std::sort()``` can always be faster than C ```qsort()``` because ```qsort()``` uses a runtime parameter as the comparision function, and that function is called umpteen zillion times. 

The C++ sort takes a comparison object, which can be a lambda/template/inline function, which inlines its code right within the sort template function. 

No unnecessary function calls.

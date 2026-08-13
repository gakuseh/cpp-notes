# lvalues and rvalues

> An lvalue (locator value) represents an object that occupies some identifiable
> location in memory (i.e. has an address).
>
> rvalues are defined by exclusion, by saying that every expression is either an
> lvalue or an rvalue. Therefore, from the above definition of lvalue, an rvalue
> is an expression that does not represent an object occupying some identifiable
> location in memory.
>
> \-
> https://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c/

```c++
int var;
var = 4;
```

`var` is an lvalue, because it is an object with an identifiable memory
location.

By contrast:

```c++
4 = var;       // ERROR!
(var + 1) = 4; // ERROR!
```

Neither `4` nor `(var + 1)` are objects with an identifiable memory location.
The code errors because `=` requires an `lvalue` on the left side.

Another example:

```c++
int globalVar = 20;
int foo() {
    return globalVar;
}

int main()
{
    foo() = 2; // error

    return 0;
}
```

`foo()` evaluates to `2`, which is an rvalue, so the code errors.

However, consider:

```c++
int globalvar = 20;

int& foo()
{
    return globalvar;
}

int main()
{
    foo() = 10;
    return 0;
}
```

`foo()` now returns a `int&`, i.e. a reference to a variable, namely to
`globalVar`. Thus, `foo()` evaluates to an lvalue, and so the line `foo() = 10`
is perfectly legal.

With the introduction of `const` keyword in ISO C, you can only modify
modifiable lvalues.

```c++
const int a = 10;
a = 10; // error, because a is not a modifiable lvalue.
```

> All lvalues that aren't arrays, functions or of incomplete types can be
> converted thus to rvalues. - Bendersky 2011
>
> ```c++
> int a = 1;     // a is an lvalue
> int b = 2;     // b is an lvalue
> int c = a + b; // + needs rvalues, so a and b are converted to rvalues
>                // and an rvalue is returned
> ```

rvalues will never be implicitly converted into lvalues, but is still possible:

```c++
int arr[] = {1, 2};
int* p = &arr[0];
*(p + 1) = 10;   // OK: p + 1 is an rvalue, but *(p + 1) is an lvalue
                 // This does not imply that p (or pointers) is an rvalue or lvalue (in fact it can be an lvalue or an rvalue)
                 // Instead explicitly the EXPRESSION p + 1 is an rvalue. (It's like how 3 + 1 is an rvalue, or a+1 is an rvalue if int a = 3).
                 // See also https://stackoverflow.com/questions/8450429/is-a-pointer-an-lvalue-or-rvalue
```

# lvalue references

Before C++11, there were only references. After C++11, the old term of
"references" now means "lvalue references".
(https://www.learncpp.com/cpp-tutorial/rvalue-references/)

E.g.

```c++
std::string name("Yotsuba");
std::string& ref = name;
```

`ref` is an lvalue reference; it refers to another lvalue. This is why you
cannot set a reference equal to a literal. lvalue references must hold an
lvalue. This is why the below code is not valid.

```c++
std::string& ref = std::string();
```

`std::string()` is an rvalue. We cannot store it in an lvalue reference.

**_HOWEVER_**, you can store an rvalue into a lvalue reference IF the lvalue
reference is constant. For example:

```c++
const std::string& name = std::string("Yotsuba");
```

Since the lvalue reference is const, you can't change the value that the
reference holds, so this will work. You can only read the rvalue the lvalue
reference refers to.

The above code may appear to be pretty useless, and may seem identical to

```c++
const std::string name = std::string("Yotsuba");
```

but this is more useful if `name` was a function parameter, and you wanted to
read values in by reference, and not copy the value.

# rvalue references

lvalue references could only be initialized with a reference to an lvalue. As a
complement to that, Rvalue references can only be initialized with a reference
to an rvalue.

That's pretty much it for the definition of rvalue. Rather, the importance of
rvalue comes from the implications of its definition. **By merely restricting
itself to be only bindable to rvalues, rvalue references become very useful, in
particular with [[move-semantics]]**.

TODO:

List examples from https://www.learncpp.com/cpp-tutorial/rvalue-references/

Put the move semantics example int he move semantics doc
https://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c/

Read the SO answer at
https://stackoverflow.com/questions/3106110/what-is-move-semantics

Watch this video eventually (helpful for RAII too \[which apparently also is
what makes rust rust ??]) https://www.youtube.com/watch?v=7Qgd9B1KuMQ

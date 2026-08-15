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

First, you can overload functions that can distinguish between rvalues and
lvalues

```c++
void fun(const int& lref) // l-value arguments will select this function
{
	std::cout << "l-value reference to const: " << lref << '\n';
}

void fun(int&& rref) // r-value arguments will select this function
{
	std::cout << "r-value reference: " << rref << '\n';
}

int main()
{
	int x{ 5 };
	fun(x); // l-value argument calls l-value version of function
	fun(5); // r-value argument calls r-value version of function

	return 0;
}
```

Output:

```
l-value reference to const: 5
r-value reference: 5
```

This is important for [[move-semantics]].

Once created rvalue references, are basically still lvalues. In the example
below, the variable `var` acts as an lvalue.

```c++
std::string&& var = std::string("Hello world");

std::string&& faulty = var; // Error: cannot bind lvalue to rvalue reference
```

This makes it, in most cases, practically identical to an lvalue reference or
just a normal variable.

Rather, the actual use of it is _semantic_. It emphasizes that you really care
about the value of an object, rather than the name.

For example, in the below example, we bind an lvalue, emphasizing that we care
about the name/variable, `text`, we are binding the lvalue to.

```c++
std::string text("yotsuba")

std::string& lvalRef = text;
```

On the other hand, in the below example, we turn the initial lvalue `text` into
an rvalue using `std::move`, which tells the `operator=` of string that we
really only care about the value, not the variable `text`. As a result, `dest`
will move the contents of `text` from `text` into `dest`, and we must not use
`text` again.

```c++
std::string text("Yotsuba");

std::string dest = std::move(text);
```

Rvalue references do not force anybody to move things. Instead, string's
`operator=` implementation just treats rvalues differently (compared to using
lvalues) because usually getting an rvalue means it's from `std::move` or from
an object that's already created.

# Function Templates

Templates are similar to generics in Java, and can be used for functions or
classes.

> The initial function template that is used to generate other functions is
> called the **primary template**, and the functions generated from the primary
> template are called **instantiated functions**.
>
> \- https://www.learncpp.com/cpp-tutorial/function-templates/

Example:

```cpp
template <typename T> // this is the template parameter declaration defining T as a type template parameter
T max(T x, T y) // this is the function template definition for max<T>
{
    return (x < y) ? y : x;
}
```

`typename` in general is interchangable with `class` when making templates,
although not in some
[more advanced scenarios](https://stackoverflow.com/questions/2023977/what-is-the-difference-between-typename-and-class-template-parameters).

`typename` was introduced after `class` to clarify you don't need to put a class
in the template. As a result, older code may use `class` instead of `typename`.

# Template argument deduction

Rather than

```cpp
std::cout << max<int>(1, 2) << '\n'; // specifying we want to call max<int>
```

the template argument can be deduced like so:

```cpp
std::cout << max<>(1, 2) << '\n';
std::cout << max(1, 2) << '\n';
```

Putting empty angle brackets will have the compiler consider only template
function overloads. Not putting empty angle brackets will have the compiler
consider both template function overloads and non-template function overloads
(e.g. `max(double, double)`). If both template function and non-template
function overloads are found, non-template overloads are chosen.

Example from learncpp.com:

```cpp
#include <iostream>

template <typename T>
T max(T x, T y)
{
    std::cout << "called max<int>(int, int)\n";
    return (x < y) ? y : x;
}

int max(int x, int y)
{
    std::cout << "called max(int, int)\n";
    return (x < y) ? y : x;
}

int main()
{
    std::cout << max<int>(1, 2) << '\n'; // calls max<int>(int, int)
    std::cout << max<>(1, 2) << '\n';    // deduces max<int>(int, int) (non-template functions not considered)
    std::cout << max(1, 2) << '\n';      // calls max(int, int)

    return 0;
}
```

# Template instantiation that don't compile, or shouldn't compile

Consider

```cpp
#include <iostream>
#include <string>

template <typename T>
T addOne(T x)
{
    return x + 1;
}

int main()
{
    std::string hello { "Hello, world!" };
    std::cout << addOne(hello) << '\n';

    return 0;
}
```

This will instantiate an `addOne` function that adds 1 to a string, like:

```cpp
template<>
std::string addOne<std::string>(std::string x)
{
    return x + 1;
}
```

But you can't add one to a string, so this code will not compile.

More nefariously is when an instantiated function shouldn't compile, but does.
For example, if you put a string literal (type `char*`), it will compile,
because you can add one to a pointer, but you may not actually want this:

```cpp
#include <iostream>

template <typename T>
T addOne(T x)
{
    return x + 1;
}

int main()
{
    std::cout << addOne("Hello, world!") << '\n';

    return 0;
}
```

Output:

```
ello, world!
```

One way to prevent this is with **concepts**, covered in the sections below, or
by explicitly preventing some instantiated functions from being created.

Following the previous example, you can prevent `char*` instantiated functions
to be created by setting them to `delete`:

```cpp
#include <iostream>
#include <string>

template <typename T>
T addOne(T x)
{
    return x + 1;
}

// Use function template specialization to tell the compiler that addOne(const char*) should emit a compilation error
// const char* will match a string literal
template <>
const char* addOne(const char* x) = delete;

int main()
{
    std::cout << addOne("Hello, world!") << '\n'; // compile error

    return 0;
}
```

# Non-type template parameters

Have the function change based on a `constexpr` value.

Example:

```cpp
#include <iostream>

template <int N> // declare a non-type template parameter of type int named N
void print()
{
    std::cout << N << '\n'; // use value of N here
}

int main()
{
    print<5>(); // 5 is our non-type template argument

    return 0;
}
```

Helpful when you want an argument to force the caller to use only `constexpr`
values. For example, tuples in C++ use `std::get<INDEX>(tuple)`, likely because
the size of tuples should be known at compile time, and thus the indicies should
also be known at compile time (if you want dynamic sizes, use a vector).

> As of C++20, function parameters cannot be constexpr.
>
> \- https://www.learncpp.com/cpp-tutorial/non-type-template-parameters/

although

> # Author’s note
>
> Having to use non-type template parameters to circumvent the restriction that
> function parameters can’t be constexpr isn’t great. There are quite a few
> different proposals being evaluated to help address situations like this. I
> expect that we might see a better solution to this in a future C++ language
> standard.
>
> \- https://www.learncpp.com/cpp-tutorial/non-type-template-parameters/

# Templates in multiple files

Because the compiler literally creates new definitions of templates every time
you use it, the template definition must be visible in the same file where you
use it.

This is why for template functions, their definitions are usually visible in the
header file where they are declared, while other normal functions usually have
their definitions elsewhere.

# Concepts

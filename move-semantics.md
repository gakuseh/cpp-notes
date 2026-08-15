# Move Semantics

Usually, when you assign one object to another, it copies the object:

```cpp
std::vector<int> vec {1, 2, 3};
std::vector<int> copy = vec; // Copy is a copy of vect
```

However, this may be inefficient, or in some cases, against the design of the
object. Instead, you may want to **move** the object.

To do this, you need to use `std::move`:

```cpp
#include <utility>

std::vector<int> vec {1, 2, 3};
std::vector<int> dest = std::move(vec);
```

The data of `vec` is moved into `dest`, and the `vec` object should no longer be
used. Use after move is undefined.

This eliminates any expensive copies. In addition, it is required for the design
of some objects, such as `unique_ptr`, where only one name can have access to a
pointer at any given time.

Move semantics works by using the `lvalue` and `rvalue` distinction. They have
at least one `operator=` that accepts rvalues (rather than lvalues) and have it
so the `operator=` moves the data from the other object. Objects may also define
an `operator=` that accepts lvalues and have that act as the standard copy
assignment operator, or don't by explicitly setting it to `delete` so no default
is created.

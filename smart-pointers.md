# Smart Pointer

Raw pointers have many problems, such as use after the memory has been
deallocated, or forgetting to deallocate the memory and resulting in a memory
leak.

Smart pointers are objects that wrap raw pointers and then provide extra safety
features.

Smart pointers may also be classified into **owning pointers**, or pointers that
will deallocate the data they point to once they go out of scope, and
**non-owning pointers**, pointers that don't. Raw pointers clearly can't be
classified in this way because they can't do any automatic deallocation once
they go out of scope.

In modern C++ (after C++11), there are two smart pointers: `unique_ptr` and
`shared_ptr`.

# `unique_ptr`

`unique_ptr`s is an "owning pointer". Once it goes out of scope, the resource it
points to is deallocated.

```cpp
#include <memory>

auto ptr = std::unique_ptr<std::string>(new std::string("Yotsuba"));

std::string* raw_pointer = ptr.get(); // .get() to get the wrapped pointer

std::string& name = *ptr; // you can deallocate to get value like normal pointer
                          // make sure to make it a reference, or else you will
                          // get a copy
```

You can also create unique_ptr from rvalue using `make_unique`.

```cpp
auto vec = std::vector<int>({1, 2, 3});

auto ptr = std::make_unique<std::vector<int>>(vec);
```

Or if you'd like to move from `vec` to the `pointer`:

```cpp
auto vec = std::vector<int>({1, 2, 3});

auto ptr = std::make_unique<std::vector<int>>(std::move(vec));
```

Because `unique_ptr` deallocates once it goes out of scope, it doesn't really
make sense for multiple `unique_ptr`s to point to the same object; if there were
multiple, which one actually deallocates the memory?

Thus, `unique_ptr`s must be moved, and must be used in conjunction with
`std::move`:

```cpp
auto ptr = std::unique_ptr<std::string>(new std::string("Yotsuba"));
auto dest = std::move(ptr);

// now only dest points to the string

// using ptr afterwards is undefined behavior
auto stuff = *ptr; // VERY BAD!!
```

Not using `std::move` would use the lvalue version of `unique_ptr`'s
`operator=`, but it is deleted, which results in a compile time error (which is
good)

When returning a `unique_ptr`, however, you don't need to use `std::move`

```cpp
std::unique_ptr<std::string> new_name() {
    auto ptr = std::unique_ptr<std::string>(new std::string("Yotsuba"));

    return ptr; // don't use std::move
}
```

# `shared_ptr`

> The `shared_ptr` is similar to `unique_ptr` except that `shared_ptr`s are
> copied rather than moved. The `shared_ptr`s for an object share ownership of
> an object; that object is destroyed when the last of its `shared_ptr`s is
> destroyed.
>
> \- Stroustrup, _A Tour of C++_ Third edition

Interface is quite similar to `unique_ptr`, and has a matching `make_shared`.

Basically reference counting garbage collection in C++.

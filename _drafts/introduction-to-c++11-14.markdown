---
layout: post
title:  "Introduction to some C++11/14 Features"
date:   2015-12-02 21:15:59
categories: C++
---

C++ is a Multi-Paradism programming language: functional, procedural, generic, object-oriented.

In C++11 standard, there are many new features.

- `auto`
- nullptr
- Range-based for loop
- New STL containers
- RAII (Smart Pointer)
- Lambda function
- Move Semantics && "Perfect Forwarding"
- Thread supporting library
- Atomic Type
- Variadic template (C++14)

Here let's come across some new features of C++11/14.

## auto

In C++98, we often have to write such code:

```cpp
std::vector<std::pair<int, std::string>> list;
for (std::vector<std::pair<int, std::string>>::iterator it = list.begin();
        it < list.end(); it++) {
    do_something(it);
}
```

In C++11 we can use `auto` to get the same result:

```cpp
std::vector<std::pair<int, std::string>> list;
for (auto it = list.begin(); it < list.end(); it++) {
    do_something(it);
}
```

We can also use **Range based for loop** to make it more terse:

```cpp
std::vector<std::pair<int, std::string>> list;
for (auto &&item : list {
    do_something(item);
}
```

## `nullptr`
```cpp
int foo(int i);

int foo(char* i);

foo(NULL);
foo(nullptr);
```


## STL containers
- `std::vector`
- `std::list`
- `std::map`, `std::set`
- `std::multimap`, `std::multiset`

## New STL containers
- `std::array`
- `std::forward_list`
- `std::unordered_map`, `std::unordered_set`
- `std::unordered_multimap`, `std::unordered_multiset`

## Lambda function
- `[](){}`
- `[capture list](args) -> return type {}`
  - [&]: reference
  - [=]: value
- Usable for algorithms
  - `std::copy_if()`, `std::find_if`
- Can be storaged as object

```cpp
std::vector<Foo> l;
auto it = find_if(l.begin(), l.end(), [] (const Foo& foo) {
    return foo.bar == 0;
});
```

## RAII
### C++98
```cpp
void foo(int i) {
  bar* b = new bar(i);
  b.xxx();
  b.yyy();
  ....
  delete b;
}
```
- Seems no problem?

## RAII
```cpp
void foo(int i) {
  bar* b = new bar(i);
  b.xxx();
  b.yyy();    // <= throw exception.
  ....
  delete b;   // XXX memory leaks
}
```
- Not exception safe
- That's why many C++ programs disable exception


## RAII
- RAII (Resource Acquisition is Initialization)
- Use `std::unique_ptr`
```cpp
void foo() {
  auto b = std::make_unique<bar>(i);
  b.xxx();
  b.yyy();    // <= throw exception.
  ....
  //delete b;   // no memory leaks
}
```

## RAII
- `std::unique_ptr`
- `std::shared_ptr`
    - Reference counting smart pointer
- Helpers
  - `std::make_unique<T>() // C++14
  - `std::make_shared<T>()


```cpp
void f(int i) {
    static std::mutex datalock;
    // C++98
    datalock.lock();
    xxxx   // may throw exception
    datalock.unlock();

    // C++11
    {
        std::lock_guard<std::mutex> lock(datalock);
        xxxxx // do some thing
    }

}

```

## Move Semantics
```cpp
class A {
public:
  A(size_t size): size_(size) {
    data = new int[size];
  }
  ~A() {
    delete data;
  }
  size_t size() const {
    return size_;
  };
private:
  size_t size_;
  int* data_;
}

A a;
A b = a; // what is done?
```
- Deep Copy
- Shallow Copy
  - We need override copy ctor

---
## Copy ctor
```cpp
class A {
public:
  A(size_t size): size_(size) {
    data = new int[size];
  }
  A(const A &a) {
    data = new int[a.size_];
    memcpy(data, a.data, a.size_ * sizeof(int));
  }
  ~A() {
    if (data)
      delete data;
  }
  size_t size() const {
    return size_;
  };
private:
  size_t size_;
  int* data_;
}

A a;
A b = a; // works well
```
---
## New problem
```cpp
A getObj(int size) {
  A a(size);
  a.xxx();
  a.yyy();
  return a;
}

A b = getObj(10);

```
- 1 ctor, 2 copy ctor
- too expensive


---
## Rvalue reference
- What is rvalue?
  - Has name
  - Has address

```cpp
int a = 42;
int b = 43;

a = b; // ok
a = a * b; // ok
a * b = a; // not ok, rvalue on left hand of =
Foo foo = getFoo(); // ok
```

- rvalue reference

---

## Move ctor
```cpp
A(A&& a) noexcept {
    size_ = a.size_;
    data_ = a.data_;
    a.data_ = nullptr;
    a.size_ = 0;
};
A(const A& a) {
    size_ = a.size_;
    memcpy(data_, a.data_, a.size_ * sizeof(int));
};
```
---
## New problem
- Ctor

```cpp
FooBar(Bar && bar, Foo &&foo) {
    xxxxxxxxxx;
}

FooBar(const Bar &bar, const Foobar &foo) {
}
```
- Do we need 4 combinations?
---
## Solution: reference coallapse
- T & && = T&
- Universal reference

```cpp
template<typename T, typename U>
Foo(T &&t, U &&u):T(t), U(u) {

}
```
- t & u: lvalue or rvalue?
---
## Move Semantics
- Helpers
  - `std::move()`
  - `std::forward()`

---
## Move Semantics
- `std::move`: cast lvalue to rvalue

```cpp
template <class _Tp>
typename remove_reference<_Tp>::type&& move(_Tp&& __t) {
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}
```

```cpp
- A b = a;           // call the A::operator=(const A& b);
- A b = getObj();    // call the A::operator=(A&& b);
- A b = std::move(a) // Call the A::operator=(A&& b);
```
---
## Move Semantics
- `std::forward()`: keep lvalue & rvalue

```cpp
template <class _Tp>
_Tp&& forward(typename std::remove_reference<_Tp>::type& __t) {
    return static_cast<_Tp&&>(__t);
}

template <typename T>
void foo(T&& arg) {
    bar(std::forward<T>(arg));
}
```

- if arg is A&&, T = A, Tp = A, forward A&&
- if arg is A&, T = A&, Tp = A&, forward A&
- Perfect forwarding!

## Thread supporting library
- `std::thread`
- `std::mutex`, `std::lock`, `std::lock_guard`
- `std::async()`, `std::future`, `std::promise`

```cpp
auto handle = std::async(f, args); // Return a future
auto result = handle.get();

```

## Atomic variables

- `std::atomic_int`
- `std::atomic<T>`

## Variadic template (C++14)
```cpp
template<typename T>
void print(T arg){
    std::cout << arg << std::endl;
}
template<typename T, typename ...Type>
void print(T arg, Type ...args) {
    std::cout << arg << std::endl;
    print(..args);
}
```

```cpp

template <typename... T>
inline Database::ResultSet Database::query(const std::string &sql, T... args) {
Database::PreparedStatement stmt(Database::instance().getConnection()
        ->prepareStatement(sql));
  return query_impl(std::move(stmt), 1, args...);
}

```

---
## References
- [C++ Reference](http://en.cppreference.com/w/)
- Effective C++
- [Effective Modern C++](http://shop.oreilly.com/product/0636920033707.do)
- [C++ Concurrency in Action](https://www.manning.com/books/c-plus-plus-concurrency-in-action)
- [C++ Core Guideline](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md)

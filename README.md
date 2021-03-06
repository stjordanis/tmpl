TMPL
====

TMPL ("temple") is a C++17 metaprogramming library that will eventually
replace the metaprogramming facilities in my other projects.
This is mostly done so that I have only a single codebase
to test and maintain, especially since this tends to be a
bug-prone part of code.

Parts of the design/syntax are inspired by boost::hana, a popular
metaprogramming library that likely is more robust than this library.

Compatibility
-------------
This library requires C++17 features (fold expressions, constexpr if,
constexpr lambdas, auto templates), so a recent compiler is required.
The library is being developed using gcc-7.1 and clang-5.0, so 
it should always work with at least these compilers.

Currently, GCC 7.1 seems to struggle with the constexpr-ness of the
"concepts" (see below/examples). Due to this, it is necessary to
break usage out into two parts:
```C++
constexpr bool Foo_has_bar = tmpl_has_member(Foo, bar); //assume that type "Foo" has a member "bar"
std::integral_constant<bool, Foo_has_bar> Baz;

//This fails to compile with my GCC, works fine with Clang 5.0:
std::integral_constant<bool, tmpl_has_member(Foo,bar)> BadBaz;
```
**UPDATE 2/13/18:** Thanks to some wonderful input from @manfredd,
the value_list has been re-worked and the library now compiles under
clang 5.0 and 6.0!


Installation
------------
The library is header-only, so it can be trivially "built"
and installed using CMake. The commands below will install
it into the default CMake installation location.
```bash
mkdir build
cd build
cmake ..
make install
```

Usage
-----
The easiest way to use tmpl is via CMake, and it can
be included in your project via
```CMake
find_package(tmpl)
target_link_libraries(YourExecutableOrLibrary PUBLIC tmpl::tmpl)
```
Simply include "tmpl.hpp", and all functionality will 
be available. All user-facing types and functions are 
contained in the "tmpl" namespace. An overview of 
functionality is given below.


Basic Types and Operations
--------------------------
The bread-and-butter of the library (so far) is the "type_list".
It is exactly what it sounds like: a list of types inside a template.

```C++
template<typename ...T>
struct type_list {};
```

The type_list can be manipulated as a value to perform
useful operations on the list:
```C++
type_list<int, double, char, myClass> List;

//Get head/tail of list
auto h = List.head(); // decltype(h) == type_list<int>
auto t = List.tail(); //decltype(t) == type_list<double, char, myClass>

// Concatenation
auto L2 = t | h; //decltype(L2) == type_list<double, char, myClass, int>

// Reversal
auto LR = reverse(List); //decltype(LR) == type_list<myClass, char, double, int>

//---------------------------------------------------
// Set operations
//---------------------------------------------------
type_list<int, double, int, double, int, double> ID_Repeated;
type_list<char, int, char, int, char, int> CI_Repeated;

auto ID_Set = make_set(ID_Repeated); //decltype(ID_Set) == type_list<int,double>
auto CI_Set = make_set(CI_Repeated); //decltype(CI_Set) == type_list<char,int>

auto ID_minus_CI = set_difference(ID_Set, CI_Set); // decltype(ID_minus_CI) == type_list<double>
auto CI_minus_ID = set_difference(CI_Set, ID_Set); // decltype(CI_minus_ID) == type_list<char>
auto sym_diff = symmetric_difference(ID_Set, CI_Set); //decltype(sym_diff) == type_list<double, char>
```

There is an alias for a type list with a single type, called
```Type```, which can be "unboxed" to yield a default-constructed
value of the contained type.
```C++
Type<int> T;
auto Tval = unbox(T); //decltype(Tval) == int
```

Concepts
--------
There are also facilities for creating powerful concepts to enforce
constraints on generic code. Inspired by boost::hana, I have included
an "is_valid" function to test whether a code section is valid.
Using this, other concepts can be built for more special-purpose constraints.

Since they are among the most common concepts, I have included macros
that test for the presence of (public) class members, typedefs,
nonstatic member functions, and static member functions.
See examples/concepts.cpp for usage and for an example of creating
a custom concept.

```C++
struct A {
    int a;
    using value_type = int;
};

struct B{};

// use macros to test for members and typedefs:
tmpl_has_member(A, a)

```

Macro Citizenship
=================
Since one of the major barriers to dependency management in C++ is the presence
of macros, this section aims to list the macros defined and used by the library.

Convenience Macros
------------------

- `NAMESPACE_TMPL_OPEN`: identical to `namespace tmpl {`, used to prevent editors
  from automatically reformatting and generating whitespace-only commits 
  based on different style preferences
- `NAMESPACE_TMPL_CLOSE`: the closing tag, `}`

Function-like Macros
--------------------
The following macros are intended to be used by the user like functions,
but they must be macros due to C++'s inability to provide a "mixin" feature
such as the one provided in the D language.

Provided by the Concepts module:
- `tmpl_has_member(TYPE,MEMBER)`: check if type `TYPE` has a member variable `MEMBER` without triggering a compilation error
- `tmpl_has_typedef(TYPE,TYPEDEF)`: check if `typename TYPE::TYPEDEF` is well-formed without triggering a compilation error
- `tmpl_has_nonstatic_member_function(TYPE,MEMBER)`: check if `TYPE` has a member function named `MEMBER` that
  *is not* a static member function, without triggering a compilation error. No checking of the function signature
  is currently done.
- `tmpl_has_static_member_function(TYPE,MEMBER)`: check if `TYPE` has a member function named `MEMBER` that
  *is* a static member function, without triggering a compilation error. No checking of the function signature
  is done.


TODO
====
- more concepts!
- real-life (useful) examples
    - topological sort of types with dependencies (useful for accumulators?)
- value_list math & reductions
    - cumsum
    - cumprod
- boolean functions
    - operator==(list A, list B)
    - any(list, f)
    - all(list, f)
- selection functions
    - select_if (return elements of type/value_list satisfying a boolean function)
    - indexing (operator\[\](std::integral_constant))

    

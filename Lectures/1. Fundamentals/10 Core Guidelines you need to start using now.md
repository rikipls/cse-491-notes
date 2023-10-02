[Video link](https://www.youtube.com/watch?v=XkDEzfpdcSg)

1. [[#[C.45 Don’t define a default constructor that only initializes data members; use in-class member initializers instead](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines c45-dont-define-a-default-constructor-that-only-initializes-data-members-use-in-class-member-initializers-instead)|C.45: Don’t define a default constructor that only initializes data members; use in-class member initializers instead]]
2. [[#[F.51 Where there is a choice, prefer default arguments over overloading](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines f51-where-there-is-a-choice-prefer-default-arguments-over-overloading)|F.51: Where there is a choice, prefer default arguments over overloading]]
3. [[#[C.47 Define and initialize member variables in the order of member declaration](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines c47-define-and-initialize-member-variables-in-the-order-of-member-declaration)|C.47: Define and initialize member variables in the order of member declaration]]
4. [[#[I.23 Keep the number of function arguments low](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines i23-keep-the-number-of-function-arguments-low)|I.23 Keep the number of function arguments low]]
5. [[#[ES.50 Don’t cast away `const`](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines es50-dont-cast-away-const)|ES.50 Don’t cast away `const`]]
6. [[#[I.11 Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines i11-never-transfer-ownership-by-a-raw-pointer-t-or-reference-t)|I.11 Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)]]
7. [[#[F.21 To return multiple “out” values, prefer returning a struct or tuple](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines f21-to-return-multiple-out-values-prefer-returning-a-struct-or-tuple)|F.21 To return multiple “out” values, prefer returning a struct or tuple]]
8. [[#[Enum.3 Prefer class enums over “plain” enums](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines enum3-prefer-class-enums-over-plain-enums)|Enum.3 Prefer class enums over “plain” enums]]
9. [[#[I.12 Declare a pointer that must not be null as `not_null`](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines i12-declare-a-pointer-that-must-not-be-null-as-not_null)|I.12 Declare a pointer that must not be null as `not_null`]]
10. [[#[ES.46 Avoid lossy (narrowing, truncating) arithmetic conversions](https //isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines es46-avoid-lossy-narrowing-truncating-arithmetic-conversions)|ES.46 Avoid lossy (narrowing, truncating) arithmetic conversions]]
### [C.45: Don’t define a default constructor that only initializes data members; use in-class member initializers instead](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c45-dont-define-a-default-constructor-that-only-initializes-data-members-use-in-class-member-initializers-instead)


##### Reason

Using in-class member initializers lets the compiler generate the function for you. The compiler-generated function can be more efficient.

##### Example, bad

```cpp
class X1 { // BAD: doesn't use member initializers
    string s;
    int i;
public:
    X1() :s{"default"}, i{1} { }
    // ...
};
```

##### Example

```cpp
class X2 {
    string s {"default"};
    int i {1};
public:
    // use compiler-generated default constructor
    // ...
};
```

##### Enforcement

(Simple) A default constructor should do more than just initialize member variables with constants.

### [F.51: Where there is a choice, prefer default arguments over overloading](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f51-where-there-is-a-choice-prefer-default-arguments-over-overloading)


##### Reason

Default arguments simply provide alternative interfaces to a single implementation. There is no guarantee that a set of overloaded functions all implement the same semantics. The use of default arguments can avoid code replication.

##### Note

There is a choice between using default argument and overloading when the alternatives are from a set of arguments of the same types. For example:

```cpp
void print(const string& s, format f = {});
```

as opposed to

```cpp
void print(const string& s);  // use default format
void print(const string& s, format f);
```

There is not a choice when a set of functions are used to do a semantically equivalent operation to a set of types. For example:

```cpp
void print(const char&);
void print(int);
void print(zstring);
```

##### See also

[Default arguments for virtual functions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-default-arg)

##### Enforcement

- Warn on an overload set where the overloads have a common prefix of parameters (e.g., `f(int)`, `f(int, const string&)`, `f(int, const string&, double)`). (Note: Review this enforcement if it’s too noisy in practice.)

### [C.47: Define and initialize member variables in the order of member declaration](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c47-define-and-initialize-member-variables-in-the-order-of-member-declaration)

##### Reason

To minimize confusion and errors. That is the order in which the initialization happens (independent of the order of member initializers).

##### Example, bad

```cpp
class Foo {
    int m1;
    int m2;
public:
    Foo(int x) :m2{x}, m1{++x} { }   // BAD: misleading initializer order
    // ...
};

Foo x(1); // surprise: x.m1 == x.m2 == 2
```

##### Enforcement

(Simple) A member initializer list should mention the members in the same order they are declared.

**See also**: [Discussion](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Sd-order)

### [I.23: Keep the number of function arguments low](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i23-keep-the-number-of-function-arguments-low)

##### Reason

Having many arguments opens opportunities for confusion. Passing lots of arguments is often costly compared to alternatives.

##### Discussion

The two most common reasons why functions have too many parameters are:

1. _Missing an abstraction._ There is an abstraction missing, so that a compound value is being passed as individual elements instead of as a single object that enforces an invariant. This not only expands the parameter list, but it leads to errors because the component values are no longer protected by an enforced invariant.
    
2. _Violating “one function, one responsibility.”_ The function is trying to do more than one job and should probably be refactored.
    

##### Example

The standard-library `merge()` is at the limit of what we can comfortably handle:

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator, class Compare>
OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                     InputIterator2 first2, InputIterator2 last2,
                     OutputIterator result, Compare comp);
```

Note that this is because of problem 1 above – missing abstraction. Instead of passing a range (abstraction), STL passed iterator pairs (unencapsulated component values).

Here, we have four template arguments and six function arguments. To simplify the most frequent and simplest uses, the comparison argument can be defaulted to `<`:

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                     InputIterator2 first2, InputIterator2 last2,
                     OutputIterator result);
```

This doesn’t reduce the total complexity, but it reduces the surface complexity presented to many users. To really reduce the number of arguments, we need to bundle the arguments into higher-level abstractions:

```cpp
template<class InputRange1, class InputRange2, class OutputIterator>
OutputIterator merge(InputRange1 r1, InputRange2 r2, OutputIterator result);
```

Grouping arguments into “bundles” is a general technique to reduce the number of arguments and to increase the opportunities for checking.

Alternatively, we could use a standard library concept to define the notion of three types that must be usable for merging:

```cpp
template<class In1, class In2, class Out>
  requires mergeable<In1, In2, Out>
Out merge(In1 r1, In2 r2, Out result);
```

##### Example

The safety Profiles recommend replacing

```cpp
void f(int* some_ints, int some_ints_length);  // BAD: C style, unsafe
```

with

```cpp
void f(gsl::span<int> some_ints);              // GOOD: safe, bounds-checked
```

Here, using an abstraction has safety and robustness benefits, and naturally also reduces the number of parameters.

##### Note

How many parameters are too many? Try to use fewer than four (4) parameters. There are functions that are best expressed with four individual parameters, but not many.

**Alternative**: Use better abstraction: Group arguments into meaningful objects and pass the objects (by value or by reference).

**Alternative**: Use default arguments or overloads to allow the most common forms of calls to be done with fewer arguments.

##### Enforcement

- Warn when a function declares two iterators (including pointers) of the same type instead of a range or a view.
- (Not enforceable) This is a philosophical guideline that is infeasible to check directly.

### [ES.50: Don’t cast away `const`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es50-dont-cast-away-const)

##### Reason

It makes a lie out of `const`. If the variable is actually declared `const`, modifying it results in undefined behavior.

##### Example, bad

```cpp
void f(const int& x)
{
    const_cast<int&>(x) = 42;   // BAD
}

static int i = 0;
static const int j = 0;

f(i); // silent side effect
f(j); // undefined behavior
```

##### Example

Sometimes, you might be tempted to resort to `const_cast` to avoid code duplication, such as when two accessor functions that differ only in `const`-ness have similar implementations. For example:

```cpp
class Bar;

class Foo {
public:
    // BAD, duplicates logic
    Bar& get_bar()
    {
        /* complex logic around getting a non-const reference to my_bar */
    }

    const Bar& get_bar() const
    {
        /* same complex logic around getting a const reference to my_bar */
    }
private:
    Bar my_bar;
};
```

Instead, prefer to share implementations. Normally, you can just have the non-`const` function call the `const` function. However, when there is complex logic this can lead to the following pattern that still resorts to a `const_cast`:

```cpp
class Foo {
public:
    // not great, non-const calls const version but resorts to const_cast
    Bar& get_bar()
    {
        return const_cast<Bar&>(static_cast<const Foo&>(*this).get_bar());
    }
    const Bar& get_bar() const
    {
        /* the complex logic around getting a const reference to my_bar */
    }
private:
    Bar my_bar;
};
```

Although this pattern is safe when applied correctly, because the caller must have had a non-`const` object to begin with, it’s not ideal because the safety is hard to enforce automatically as a checker rule.

Instead, prefer to put the common code in a common helper function – and make it a template so that it deduces `const`. This doesn’t use any `const_cast` at all:

```cpp
class Foo {
public:                         // good
          Bar& get_bar()       { return get_bar_impl(*this); }
    const Bar& get_bar() const { return get_bar_impl(*this); }
private:
    Bar my_bar;

    template<class T>           // good, deduces whether T is const or non-const
    static auto& get_bar_impl(T& t)
        { /* the complex logic around getting a possibly-const reference to my_bar */ }
};
```

Note: Don’t do large non-dependent work inside a template, which leads to code bloat. For example, a further improvement would be if all or part of `get_bar_impl` can be non-dependent and factored out into a common non-template function, for a potentially big reduction in code size.

##### Exception

You might need to cast away `const` when calling `const`-incorrect functions. Prefer to wrap such functions in inline `const`-correct wrappers to encapsulate the cast in one place.

##### Example

Sometimes, “cast away `const`” is to allow the updating of some transient information of an otherwise immutable object. Examples are caching, memoization, and precomputation. Such examples are often handled as well or better using `mutable` or an indirection than with a `const_cast`.

Consider keeping previously computed results around for a costly operation:

```cpp
int compute(int x); // compute a value for x; assume this to be costly

class Cache {   // some type implementing a cache for an int->int operation
public:
    pair<bool, int> find(int x) const;   // is there a value for x?
    void set(int x, int v);             // make y the value for x
    // ...
private:
    // ...
};

class X {
public:
    int get_val(int x)
    {
        auto p = cache.find(x);
        if (p.first) return p.second;
        int val = compute(x);
        cache.set(x, val); // insert value for x
        return val;
    }
    // ...
private:
    Cache cache;
};
```

Here, `get_val()` is logically constant, so we would like to make it a `const` member. To do this we still need to mutate `cache`, so people sometimes resort to a `const_cast`:

```cpp
class X {   // Suspicious solution based on casting
public:
    int get_val(int x) const
    {
        auto p = cache.find(x);
        if (p.first) return p.second;
        int val = compute(x);
        const_cast<Cache&>(cache).set(x, val);   // ugly
        return val;
    }
    // ...
private:
    Cache cache;
};
```

Fortunately, there is a better solution: State that `cache` is mutable even for a `const` object:

```cpp
class X {   // better solution
public:
    int get_val(int x) const
    {
        auto p = cache.find(x);
        if (p.first) return p.second;
        int val = compute(x);
        cache.set(x, val);
        return val;
    }
    // ...
private:
    mutable Cache cache;
};
```

An alternative solution would be to store a pointer to the `cache`:

```cpp
class X {   // OK, but slightly messier solution
public:
    int get_val(int x) const
    {
        auto p = cache->find(x);
        if (p.first) return p.second;
        int val = compute(x);
        cache->set(x, val);
        return val;
    }
    // ...
private:
    unique_ptr<Cache> cache;
};
```

That solution is the most flexible, but requires explicit construction and destruction of `*cache` (most likely in the constructor and destructor of `X`).

In any variant, we must guard against data races on the `cache` in multi-threaded code, possibly using a `std::mutex`.

##### Enforcement

- Flag `const_cast`s.
- This rule is part of the [type-safety profile](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Pro-type-constcast) for the related Profile.

### [I.11: Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i11-never-transfer-ownership-by-a-raw-pointer-t-or-reference-t)

##### Reason

If there is any doubt whether the caller or the callee owns an object, leaks or premature destruction will occur.

##### Example

Consider:

```cpp
X* compute(args)    // don't
{
    X* res = new X{};
    // ...
    return res;
}
```

Who deletes the returned `X`? The problem would be harder to spot if `compute` returned a reference. Consider returning the result by value (use move semantics if the result is large):

```cpp
vector<double> compute(args)  // good
{
    vector<double> res(10000);
    // ...
    return res;
}
```

**Alternative**: [Pass ownership](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-smartptrparam) using a “smart pointer”, such as `unique_ptr` (for exclusive ownership) and `shared_ptr` (for shared ownership). However, that is less elegant and often less efficient than returning the object itself, so use smart pointers only if reference semantics are needed.

**Alternative**: Sometimes older code can’t be modified because of ABI compatibility requirements or lack of resources. In that case, mark owning pointers using `owner` from the [guidelines support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library):

```cpp
owner<X*> compute(args)    // It is now clear that ownership is transferred
{
    owner<X*> res = new X{};
    // ...
    return res;
}
```

This tells analysis tools that `res` is an owner. That is, its value must be `delete`d or transferred to another owner, as is done here by the `return`.

`owner` is used similarly in the implementation of resource handles.

##### Note

Every object passed as a raw pointer (or iterator) is assumed to be owned by the caller, so that its lifetime is handled by the caller. Viewed another way: ownership transferring APIs are relatively rare compared to pointer-passing APIs, so the default is “no ownership transfer.”

**See also**: [Argument passing](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-conventional), [use of smart pointer arguments](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-smartptrparam), and [value return](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-value-return).

##### Enforcement

- (Simple) Warn on `delete` of a raw pointer that is not an `owner<T>`. Suggest use of standard-library resource handle or use of `owner<T>`.
- (Simple) Warn on failure to either `reset` or explicitly `delete` an `owner` pointer on every code path.
- (Simple) Warn if the return value of `new` or a function call with an `owner` return value is assigned to a raw pointer or non-`owner` reference.

### [F.21: To return multiple “out” values, prefer returning a struct or tuple](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f21-to-return-multiple-out-values-prefer-returning-a-struct-or-tuple)

##### Reason

A return value is self-documenting as an “output-only” value. Note that C++ does have multiple return values, by convention of using a `tuple` (including `pair`), possibly with the extra convenience of `tie` or structured bindings (C++17) at the call site. Prefer using a named struct where there are semantics to the returned value. Otherwise, a nameless `tuple` is useful in generic code.

##### Example

```cpp
// BAD: output-only parameter documented in a comment
int f(const string& input, /*output only*/ string& output_data)
{
    // ...
    output_data = something();
    return status;
}

// GOOD: self-documenting
tuple<int, string> f(const string& input)
{
    // ...
    return {status, something()};
}
```

C++98’s standard library already used this style, because a `pair` is like a two-element `tuple`. For example, given a `set<string> my_set`, consider:

```cpp
// C++98
result = my_set.insert("Hello");
if (result.second) do_something_with(result.first);    // workaround
```

With C++11 we can write this, putting the results directly in existing local variables:

```cpp
Sometype iter;                                // default initialize if we haven't already
Someothertype success;                        // used these variables for some other purpose

tie(iter, success) = my_set.insert("Hello");   // normal return value
if (success) do_something_with(iter);
```

With C++17 we are able to use “structured bindings” to declare and initialize the multiple variables:

```cpp
if (auto [ iter, success ] = my_set.insert("Hello"); success) do_something_with(iter);
```

##### Exception

Sometimes, we need to pass an object to a function to manipulate its state. In such cases, passing the object by reference [`T&`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-inout) is usually the right technique. Explicitly passing an in-out parameter back out again as a return value is often not necessary. For example:

```cpp
istream& operator>>(istream& in, string& s);    // much like std::operator>>()

for (string s; in >> s; ) {
    // do something with line
}
```

Here, both `s` and `in` are used as in-out parameters. We pass `in` by (non-`const`) reference to be able to manipulate its state. We pass `s` to avoid repeated allocations. By reusing `s` (passed by reference), we allocate new memory only when we need to expand `s`’s capacity. This technique is sometimes called the “caller-allocated out” pattern and is particularly useful for types, such as `string` and `vector`, that needs to do free store allocations.

To compare, if we passed out all values as return values, we would something like this:

```cpp
pair<istream&, string> get_string(istream& in)  // not recommended
{
    string s;
    in >> s;
    return {in, move(s)};
}

for (auto p = get_string(cin); p.first; ) {
    // do something with p.second
}
```

We consider that significantly less elegant with significantly less performance.

For a truly strict reading of this rule (F.21), the exception isn’t really an exception because it relies on in-out parameters, rather than the plain out parameters mentioned in the rule. However, we prefer to be explicit, rather than subtle.

##### Note

In many cases, it can be useful to return a specific, user-defined type. For example:

```cpp
struct Distance {
    int value;
    int unit = 1;   // 1 means meters
};

Distance d1 = measure(obj1);        // access d1.value and d1.unit
auto d2 = measure(obj2);            // access d2.value and d2.unit
auto [value, unit] = measure(obj3); // access value and unit; somewhat redundant
                                    // to people who know measure()
auto [x, y] = measure(obj4);        // don't; it's likely to be confusing
```

The overly-generic `pair` and `tuple` should be used only when the value returned represents independent entities rather than an abstraction.

Another example, use a specific type along the lines of `variant<T, error_code>`, rather than using the generic `tuple`.

##### Note

When the tuple to be returned is initialized from local variables that are expensive to copy, explicit `move` may be helpful to avoid copying:

```cpp
pair<LargeObject, LargeObject> f(const string& input)
{
    LargeObject large1 = g(input);
    LargeObject large2 = h(input);
    // ...
    return { move(large1), move(large2) }; // no copies
}
```

Alternatively,

```cpp
pair<LargeObject, LargeObject> f(const string& input)
{
    // ...
    return { g(input), h(input) }; // no copies, no moves
}
```

Note this is different from the `return move(...)` anti-pattern from [ES.56](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-move)

##### Enforcement

- Output parameters should be replaced by return values. An output parameter is one that the function writes to, invokes a non-`const` member function, or passes on as a non-`const`.

### [Enum.3: Prefer class enums over “plain” enums](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#enum3-prefer-class-enums-over-plain-enums)

##### Reason

To minimize surprises: traditional enums convert to int too readily.

##### Example

```cpp
void Print_color(int color);

enum Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
enum Product_info { red = 0, purple = 1, blue = 2 };

Web_color webby = Web_color::blue;

// Clearly at least one of these calls is buggy.
Print_color(webby);
Print_color(Product_info::blue);
```

Instead use an `enum class`:

```cpp
void Print_color(int color);

enum class Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
enum class Product_info { red = 0, purple = 1, blue = 2 };

Web_color webby = Web_color::blue;
Print_color(webby);  // Error: cannot convert Web_color to int.
Print_color(Product_info::red);  // Error: cannot convert Product_info to int.
```

##### Enforcement

(Simple) Warn on any non-class `enum` definition.

### [I.12: Declare a pointer that must not be null as `not_null`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i12-declare-a-pointer-that-must-not-be-null-as-not_null)

##### Reason

To help avoid dereferencing `nullptr` errors. To improve performance by avoiding redundant checks for `nullptr`.

##### Example

```cpp
int length(const char* p);            // it is not clear whether length(nullptr) is valid

length(nullptr);                      // OK?

int length(not_null<const char*> p);  // better: we can assume that p cannot be nullptr

int length(const char* p);            // we must assume that p can be nullptr
```

By stating the intent in source, implementers and tools can provide better diagnostics, such as finding some classes of errors through static analysis, and perform optimizations, such as removing branches and null tests.

##### Note

`not_null` is defined in the [guidelines support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library).

##### Note

The assumption that the pointer to `char` pointed to a C-style string (a zero-terminated string of characters) was still implicit, and a potential source of confusion and errors. Use `czstring` in preference to `const char*`.

```cpp
// we can assume that p cannot be nullptr
// we can assume that p points to a zero-terminated array of characters
int length(not_null<zstring> p);
```

Note: `length()` is, of course, `std::strlen()` in disguise.

##### Enforcement

- (Simple) ((Foundation)) If a function checks a pointer parameter against `nullptr` before access, on all control-flow paths, then warn it should be declared `not_null`.
- (Complex) If a function with pointer return value ensures it is not `nullptr` on all return paths, then warn the return type should be declared `not_null`.

### [ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es46-avoid-lossy-narrowing-truncating-arithmetic-conversions)

##### Reason

A narrowing conversion destroys information, often unexpectedly so.

##### Example, bad

A key example is basic narrowing:

```cpp
double d = 7.9;
int i = d;    // bad: narrowing: i becomes 7
i = (int) d;  // bad: we're going to claim this is still not explicit enough

void f(int x, long y, double d)
{
    char c1 = x;   // bad: narrowing
    char c2 = y;   // bad: narrowing
    char c3 = d;   // bad: narrowing
}
```

##### Note

The guidelines support library offers a `narrow_cast` operation for specifying that narrowing is acceptable and a `narrow` (“narrow if”) that throws an exception if a narrowing would throw away legal values:

```cpp
i = gsl::narrow_cast<int>(d);   // OK (you asked for it): narrowing: i becomes 7
i = gsl::narrow<int>(d);        // OK: throws narrowing_error
```

We also include lossy arithmetic casts, such as from a negative floating point type to an unsigned integral type:

```cpp
double d = -7.9;
unsigned u = 0;

u = d;                               // bad: narrowing
u = gsl::narrow_cast<unsigned>(d);   // OK (you asked for it): u becomes 4294967289
u = gsl::narrow<unsigned>(d);        // OK: throws narrowing_error
```

##### Note

This rule does not apply to [contextual conversions to bool](https://en.cppreference.com/w/cpp/language/implicit_conversion#Contextual_conversions):

```cpp
if (ptr) do_something(*ptr);   // OK: ptr is used as a condition
bool b = ptr;                  // bad: narrowing
```

##### Enforcement

A good analyzer can detect all narrowing conversions. However, flagging all narrowing conversions will lead to a lot of false positives. Suggestions:

- Flag all floating-point to integer conversions (maybe only `float`->`char` and `double`->`int`. Here be dragons! we need data).
- Flag all `long`->`char` (I suspect `int`->`char` is very common. Here be dragons! we need data).
- Consider narrowing conversions for function arguments especially suspect.
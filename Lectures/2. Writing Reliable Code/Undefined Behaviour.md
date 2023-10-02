[Video link](https://www.youtube.com/watch?v=NpL9YnxnOqM)

We have a list of words: Webster's Dictionary.
What about everything that's *not a word*? There's an infinite number of non-words, we can't list them all.

Similarly, we cannot definitively say what is **undefined behaviour**.

What happens when you read past the end of a `std::vector<T>`?
- Could return a perfectly valid `T`
- Might return something that's **not** a `T`
- Program may crash at runtime
- Read could be optimized out by the compiler

According to the C++ Standard
- Reading past the end of a vector is *undefined behavior*.
So any of those could, in theory, happen.

### Definitions from the C++ Standard

**Defined behavior**
- Code which has a clear or precise meaning
```cpp
int sum = 7 + 18;
printf("Welcome to CppCon 2021");
auto [first, second] = getPair();
```

**Implementation defined behaviour**
- Code which can have multiple meanings
- Compiler must consistently pick one and document the choice
```cpp
if (sizeof(int) < sizeof(long)) {}
```

**Unspecified Behaviour**
- Code which can have multiple meanings
- Compiler is allowed to choose one at random
	- Comparing string literals
```cpp
if ("abc" == "abc") {} 
```
May compare them as strings, but these are also `char const*`, so the compiler could just compare them as addresses and all the code answers is "do these two arrays live at the same address?"
Though not undefined behaviour, it's still important to watch out for code like this because it may not perform as expected.

***Undefined Behaviour (UB)***
- Code which has no meaning
	- Invoking the destructor of an object twice
	- Doing a bit-shift by a negative value
	- Converting a double to a float when the value is too large

Once you perform these operations, your code **no longer has any meaning.**

Is the following code UB?
```cpp
int * varA = nullptr; // 1. Fine
*varA = 17;           // 2. UB: dereferencing a nullptr

int varB;             // 3. Also fine, don't need to initialize
varA = &varB;         // 4. Address of varB is valid, also fine

std::cout << *varA;   // 5. Dereference is valid, but printing is UB 
std::cout << varB;    // 6. Printing is UB
```

Line 2: Dereferencing a null pointer is UB
Line 5 and 6: Accessing and printing an uninitialized value

Does this function have UB?
```cpp
template <typename T1, typename T2>
void doLessThanLessThan(T1 & x, T2 & y) {
	x << y;
}

doLessThanLessThan(250, 75); // UB: Bit shift of larger width than platform
doLessThanLessThan(std::cout, "Hello"); // Write to standard out is fine
```

A program is *correct* and guaranteed to operate as written
- only if the code is free of undefined behaviour

Guarantees made by the C++ Standard
- None, if you have any undefined behaviour

Responsibility pushed onto the programmer

Partial list of common C++ UB
- Access to an element of an `std::vector` past the end
- De-reference of a `nullptr`
- Use of an uninitialized variable
- Calling a pure virtual function from a constructor or destructor
- Use after free
- Casting a pointer to an incompatible type and then using the result
- Infinite loop without side effects
- Modifying a string literal or any other `const` object
- Failing to return a value from a non-void function
- Any race condition
- Integer divide by zero
- Signed integer overflor
- Accessing an inactive member of a union

### Undefined Behaviour is not an Error
- No overlap between UB and an error
- Something *defined* as an error is not UB
- UB is not something your code can test for

Code which produces an error at compile-time
- Missing semicolon or unbalanced curly-braces
- Method signature incompatible with the declaration
- No matching candidate found for function call
- Adding the values of two pointers

Code which produces an error at run-time
- `mystring.erase(10)` when the index is out of range

**An error is something you can test for and handle:** you can't handle UB because the code that handles UB runs after the UB occurs and is, therefore, also UB.

Undefined behaviour needs to be prevented, not handled. It is never acceptable.

Const-cast example
```cpp
std::string const value = "tiger"; // A
doThing8(value);

void doThing8(std::string const& input) {
	std::string & tmp = const_cast<std::string &>(input); // B
	tmp = "bear";  // C
}
```
A, B, and C on their own are all fine. But the error occurs when they're together:
- `const_cast` removes the "constness" of an object.
- Modifying `input` is UB **if** the passed argument was originally declared `const`
	- If `value` was not `const`, this would not be UB

Specializing a type trait which exists in the standard namespace is UB
```cpp
namespace std {
	template <>
	struct is_pointer<int> 
	: public std::true_type {}; // Defines a type trait as true
}

bool var2 = std::is_pointer<int>::value;
```
Undefined behaviour that happens at *compile-time*: if you have this code in any source-file, your entire program is undefined behaviour

#### Resolving Undefined Behaviour

- Tools to help locate UB in your code base
	- Address sanitizer
	- Memory sanitizer
	- Undefined behaviour sanitizer
	- Thread sanitizer

- Code reviews
	- Institute a policy which exclusively checks for UB
- Pay attention to compiler warnings
- Build your code with multiple compilers
- Test crazy corner-cases
- Treat undefined behaviour as a *critical bug*
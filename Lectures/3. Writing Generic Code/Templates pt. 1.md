[Video link](https://www.youtube.com/watch?v=XN319NYEOcE)

1. [[#Function Templates]]
2. [[#Class Templates]]
3. [[#Member Function Templates]]
4. [[#Alias Templates]]
5. [[#Variable Templates]]
6. [[#Lambda Templates]]
7. [[#Nomenclature]]
8. [[#Phases of compilation]]
9. [[#Template declarations and definitions]]
10. [[#The One Definition Rule]]
11. [[#Template Parameters]]
12. [[#Type parameters]]
13. [[#Non-type template parameters (NTTPs)]]
14. [[#Template-template parameters]]
15. [[#Default template arguments]]
16. [[#Specialization]]
17. [[#Template instantiation]]
18. [[#Implicit instantiation]]
19. [[#Explicit instantiation]]
20. [[#Explicit Specialization]]

![[Pasted image 20230930113354.png]]
Why have templates? Because we want to be able to write generic code.

### Function Templates
##### Recipes for making *functions*

```cpp
template<class T>
T const& min(T const& a, T const& b) {
	return (a < b) ? a : b;
}

template<class T>
void swap(T& a, T& b);

template<class RandomIt, class Compare>
void sort(RandomIt first, RandomIt last, Compare comp);
```

### Class Templates
#### Recipes for making *classes*

```cpp
template<class T, size_t N>
struct array
{...};

template<class T, class Alloc = allocator<T>>
class vector
{...};

template<class Key, class Val,
	class Compare = less<Key>,
	class Allocator = allocator<pair<const Key, T>>>
 
class map
{...};
```

### Member Function Templates
#### Recipes for making *member functions*

```cpp
template<class T, class Alloc = allocator<T>>
class vector
{
	public:
	...
	using iterator = ...;
	using const_iterator = ...;
	...

	template<class InputIter>
	iterator insert(const_iterator pos, InputIter first, InputIter last) {...}
	...
};
```

### Alias Templates
#### Recipes for making *type aliases*

```cpp
template<class T>
using sa_vector = vector<T, my_special_allocator<T>>;

sa_vector<float> fv;

template<class Key, class Val>
using my_map = map<Key, Val, greater<Key>>;

my_map<string, int> msi;

template<class T, ptrdiff_t C, class A = std::allocator<T>, class CT = void>
using general_row_vector =
	basic_matrix<matrix_storage_engine<T, extents<1,C>, A,
	matrix_layout::row_major>, CT>;

general_row_vector<double, 20> rv;
```

### Variable Templates
#### Recipes for making variables or static data members

```cpp
template<class T>
inline constexpr T pi = T(3.1415926535897932385L);
// As long as T can be constructed from a long double literal
// then we can create this static variable

template<class T>
T circular_area(T r) { return pi<T> * r * r; }
// pi<T> is treated as a compile-time constant here

template< class T >
inline constexpr bool is_arithmetic_v = is_arithmetic<T>::value;

void init(T* p, size_t N) {
	// This is not a runtime branch, but instead is compile-time generated code
	if constexpr (is_arithmetic<T>)
		memcpy(p, 0, sizeof(T) * N);
	else
		uninitialized_fill_n(p, N, T());
	/**
	* If T is an arithmetic type, then the only code that is generated
	* is memcpy, otherwise only uninitialized_fill is generated
	 /
}
```

### Lambda Templates
#### Recipes for making *lambdas*

```cpp
auto multiply = []<class T>(T a, T b) { return a * b; };

auto d0 = multiply(1.0, 2.0);
```

### Nomenclature

![[Pasted image 20230930115933.png]]
![[Pasted image 20230930132915.png]]

- Compilation
	• The process of converting human-readable source code into binary object files
	• From a high-level perspective, there are four stages of compilation:
		• Lexical analysis
		• Syntax analysis
		• Semantic analysis
		• Code generation
	• In C++, we typically generate one object file for each source file
• Linking
	• The process of combining object files and binary libraries to make a working program
• The standard calls the compilation process translation

### Phases of compilation

- In C++, translation is performed in nine well-defined stages

• Phases 1 through 6 perform lexical analysis
	• These are what we usually refer to as pre-processing
	• The output of Phase 6 is a translation unit
 
• A translation unit is defined [5.1] as
	• A source file
	• Plus all the headers and source files included via #include directives
	• Minus any source lines skipped by conditional inclusion preprocessing directives (#ifdef)
	• And all macros expanded

Phases 7 and 8 perform syntax analysis, semantic analysis, and codegen
	• These are what we usually refer to as compilation
	• Templates are parsed in Phase 7
	• Templates are instantiated in Phase 8
	• The output is called a translated translation unit (e.g., object code)
• Phase 9 performs program image creation
	• This is what we usually think of as linking
	• The output is an executable image suitable for the intended execution environment

>A whole bunch of semantical garbage about what declarations and definitions are here

The set of definitions is a proper subset of the set of declarations:

![[Pasted image 20230930133716.png]]

![[Pasted image 20230930133750.png]]
#### Template declarations and definitions

```cpp
template<class T>
T const& max(T const& a, T const& b); //- Declaration of function template max

template<class T>
T const& max(T const& a, T const& b) //- Definition of function template max
{
	return (a > b) ? a : b;
}

template<class T1, class T2>
struct pair; //- Declaration of class template pair

template<class T1, class T2>
struct pair //- Definition of class template pair
{
	T1 first;
	T2 second;
	...
};
```

```cpp
template<class T, class Alloc = allocator<T>>
class vector
{
public:
	...
	using iterator = ...;
	using const_iterator = ...;
	...

	template<class InputIter> //- Declaration of member function template insert
	iterator insert(const_iterator pos, InputIter first, InputIter last);
	...
};

template<class T, class Alloc> //- Definition of member function template insert
template<class InputIter> auto
vector<T,Alloc>::insert(const_iterator pos, InputIter first, InputIter last) -> iterator
{ ... }
```

```cpp
template<class T, class Alloc = allocator<T>>
class vector
{
	public:
	...

	using iterator = ...;
	using const_iterator = ...;
	...

	template<class InputIter> //- Definition of member function template insert
	iterator insert(const_iterator pos, InputIter first, InputIter last)
	{ ... }
};

template<class Key, class Val>
using my_map = map<Key, Val, greater<Key>>; //- Declaration of alias template my_map

template<class T>
constexpr T pi = T(3.1415926535897932385L); //- Declaration of variable template pi
```

### The One Definition Rule

A given translation unit can contain at most one definition of any:
- variable
- function
- class type
- enumeration type
- **template**
- default argument for a parameter for a function in a given scope
- **default template argument**

- There may be multiple declarations, but there can only be one definition

- A program must contain exactly one definition of every non-inline variable or function that is used in the program
	- Multiple declarations are OK, but only one definition
 
- For an inline variable or an inline function, a definition is required in every translation unit that uses it
	- inline was originally a suggested optimization made to the compiler
	- It has now evolved to mean "multiple definitions are permitted"
 
- Exactly one definition of a class must appear in any translation unit that uses it in such a way that the class must be complete

- The same rules for inline variables and functions ***also apply to templates***

- My simple guidelines for observing ODR:
- For an inline thing (variable or function) that get used in a translation unit, make sure it is defined at least once somewhere in that translation unit

- For a non-inline, non-template thing that gets used, make sure it is defined exactly once in across all translation units

- For a template thing, define it in a header file, include the header where the thing is needed, and let the toolchain decide where it is defined
	- Except in rare circumstances where finer control is required

![[Pasted image 20230930135023.png]]

### Template Parameters
##### Come in three flavors:
1. Type parameters
2. Non-type template parameters (NTTPs)
3. Template-template parameters

### Type parameters

- Most common
- Declared using the class or typename keywords
```cpp
template<class T1, class T2> struct pair;
template<typename T1, typename T2> struct pair;

template<class T> T max(T const& a, T const& b);
template<typename T> T max(T const& a, T const& b);
```

### Non-type template parameters (NTTPs)
- Don't have to be types
```cpp
template<class T, size_t N>
class Array {
	T m_data[N]
	...
};

Array<foobar, 10> some_foobars;
```

```cpp
template<int Incr>
int IncrementBy(int val) {
	return val + Incr;
}
	int x = ...;
	int y = IncrementBy<42>(x);
```

NTTPs denote constant values that can be determined at compile or link
time, and their type must be
- An integer or enumeration type (most common)
- A pointer or pointer-to-member type
- std::nullptr_t
- And a couple of other things…

### Template-template parameters
- Template parameters can themselves be templates
	- Placeholders for class or alias templates
	- Declared like class templates, but only the class and typename keywords can be used

```cpp
#include <vector>
#include <list>

template<class T, template<class U, class A =std::allocator<U>> class C>
struct Adaptor {
	C<T> my_data;
	void push_back(T const& t) { my_data.push_back(t); }
};

Adaptor<int, std::vector> a1;
Adaptor<long, std::list> a2;

a1.push_back(0);
a2.push_back(1);
```

### Default template arguments
- Template parameters have default arguments

```cpp
template<class T, class Alloc = allocator<T>>
class vector {...};

template<class T, size_t N = 32>
class Array {...}

template<class T, template<class U, class A = allocator<U>> class C = vector>
struct Adaptor {...};

vector<double> vec; //- std::vector<double, std::allocator<double>>

Array<long> arr; //- Array<long, 32>

Adaptor<int> adp; //- Adaptor<int, std::vector<int, std::allocator<int>>>
```

- Default arguments must occur at the end of the list for class, alias, and variable templates

```cpp
template<class T0, class T1 = int, class T2 = int, class T3 = int>
class quad; //- OK

template<class T0, class T1 = int, class T2 = int, class T3 = int, class T4>
class quint; //- Error
```

**Function templates don't have this requirement
• Template type deduction can determine the template parameters**

```cpp
template<class RT = void, class T>
RT* address_of(T& value) {
	return static_cast<RT*>(&value);
};
```

### Specialization

- The concrete entity resulting from substituting template arguments for template parameters is a **specialization**
- These entities are named, and the name has the syntactic form
	`template-name<argument-list>`
	This name is formally called a template-id
 
```cpp
template<class T1, class T2>
struct pair {
	T1 first;
	T2 second;
	...
};

template<class T>
T const& max(T const& a, T const& b)
{ ... }

pair<string, double> my_pair;
double d = max<double>(0, 1);

string s1 = ...;
string s2 = ...;
string s3 = max(s1, s2);
```

- pair is a class template
- max is a function template
- These are the names of specializations
	- `pair<string, double>`
	- `max<double>`
	- `max<string>`

***How do we get from template to specialization?***
- Instantiation
- Explicit specialization

- At some point we'll want to use the recipe and make a thing
	- Most of the time the compiler knows how to cook the recipe for us
 
- At various times, the compiler will substitute concrete (actual) template arguments for the template parameters used by a template

- Sometimes this substitution is tentative
	- The compiler checks to see if a possible substitution could be valid
 
- Sometimes the result of this substitution is used to create a specialization ...

### Template instantiation
- Occurs when the **compiler** substitutes template arguments for template parameters in order to define an entity
	- I.e., generate a specialization of some template

- The specialization from instantiating a class template is sometimes called (informally) an instantiated class
	- Likewise for the other template categories (instantiated function, etc.)
	- These are also informally called instantiations
 
 - Template instantiation can occur in two possible ways
	- Implicitly
	- Explicitly

#### Implicit instantiation
- The compiler decides where, when, and how much of the specialization to create
- When the compiler sees the use of a specialization in code, it will automatically create the specialization by substituting template arguments for parameters
![[Pasted image 20230930154450.png]]
For class templates, implicit instantiation doesn't necessarily instantiate all
the members of the class
- The compiler might not generate non-virtual member functions or static data members
```cpp
void f() {
	vector<int> v{1, 2};
}
```

**Relationship between specialization and instantiation**

![[Pasted image 20230930154828.png]]
#### Explicit instantiation
- Sometimes we want to control the where and when of instantiation
- This can be accomplished with explicit instantiation

```cpp
template class vector<foo>; //- Definition
template class vector<foo, my_allocator<foo>>; //- Definition

template void swap<foo>(foo&, foo&); //- Definition
template void swap(bar&, bar&); //- Definition
```

- Explicit instantiation of a class template instantiates ***all*** members

However, individual member functions can be explicitly instantiated
```cpp
template void vector<foo, my_allocator<foo>>::push_back(foo const&); //- Definition
```

- For each template instantiated in a program, there must be exactly one definition of the corresponding specialization
	- If you explicitly instantiate a template in one translation unit, you must not explicitly instantiate in another translation unit

```cpp
//- Source file my_foo.cpp
template class vector<foo>; //- Definition
template class vector<foo, my_allocator<foo>>; //- Definition
template void swap<foo>(foo&, foo&); //- Definition
template void swap(bar&, bar&); //- Definition
```
```cpp
//- Header file my_foo.h
extern template class vector<foo>; //- Declared, not defined
extern template class vector<foo, my_allocator<foo>>; //- Declared, not defined
extern template void swap<foo>(foo&, foo&); //- Declared, not defined
extern template void swap(bar&, bar&); //- Declared, not defined
```

##### What if you want to customize the behavior of a template for a special situation?

```cpp
template<class T>
T const& min(T const& a, T const& b) {
	return (a < b) ? a : b;
}
char const* p0 = "hello";
char const* p1 = "world";
char const* pr = min(p0, p1); //- What's the answer? There is no answer
```

Sometimes situations arise where the template won't work properly. Then we can use explicit specialization – a user-provided implementation of a template with all template parameters fully substituted.

#### Explicit Specialization

```cpp

// Primary template
template <class T>
T const& min(T const& a, T const& b)  {
	return (a < b) ? a : b;
}

// Full specialization; this is only
// valid if a function template min has already been declared
template<>
char const* min(char const* pa, char const* pb)  { 
return (strcmp(pa, pb) < 0) ? pa : pb;
}
char const* p0 = "hello";
char const* p1 = "world";
char const* pr = min(p0, p1);
```

It's probably more common to use this with class templates:

```cpp
template<class T>
struct my_less //- Primary template
{
	bool operator()(T const& a, T const& b) const {
		return (a < b) ? a : b;
	}
};

template<>
struct my_less<char const*> //- Full specialization
{
	bool operator()(char const* pa, char const* pb) const {
		return strcmp(pa, pb) < 0;
	}
}

map<char const*, int, my_less<char const*>> m1;
```

An explicit specialization is valid only if a primary template has been declared.
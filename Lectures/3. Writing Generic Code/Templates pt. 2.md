[Video link](https://www.youtube.com/watch?v=FfI6Lov1O9M)

1. [[#L-values and R-values [(reference)](https //en.cppreference.com/w/cpp/language/value_category)|L-values and R-values]]
2. [[#Template argument deduction]]
3. [[#Reference collapsing]]
4. [[#Return-type deduction]]
5. [[#Template parameter scope]]
6. [[#Dependent names]]
7. [[#Full specialization]]
8. [[#Partial specialization]]
9. [[#Type traits]]
10. [[#Friends for class templates]]
11. [[#Summary]]

![[Pasted image 20230930184333.png]]

How do we ensure that the recipe is visible when the compiler needs to see it.

![[Pasted image 20230930184656.png]]

#### L-values and R-values [(reference)](https://en.cppreference.com/w/cpp/language/value_category)

#### Template argument deduction 
Example:
```cpp
template<class T, class U, class V = double>
int f(T& t, U u, V const& v) { ... }

int i = 0;
string s = "hi";

// How does the compiler determine the types of T, U, and V?
int r = f(i, 1.0, s);
```

Template argument deduction is the process by which the compiler determines the type of function template arguments based on function argument types.

```cpp
template<class T>
void f(ParameterType t);

invoke_f(SomeExpression); //- Type deduction occurs here
```
Given the form of *invoke_f* and the type of *SomeExpression*, the
compiler must determine
- *ParameterType*, the type of the function argument **t**
- *T*, the type of the template argument
- *ParameterType* and *T* are not necessarily the same (e.g., *string* const& and *string*)

It does this by pattern-matching:
- Explicitly-specified template arguments are fixed as specified
- Otherwise, template argument types must be inferred from the context

```cpp
template<class T>
void f(ParameterType t);

f(SomeExpression);
```

What are the possible forms that *ParameterType* could have?
- T
- T*
- T const*
- T&
- T const&
- T&&
- T const&&

![[Pasted image 20230930203849.png]]

#### Reference collapsing

![[Pasted image 20230930205058.png]]

![[Pasted image 20230930205345.png]]

We've seen how individual arguments are deduced. There's more to it; informally* the steps are:

- Explicitly-specified template arguments are fixed as such and don't participate in argument deduction
- The remaining function arguments contribute to the deduction process for their respective template parameter
- All deductions occur in parallel – the deduction for a given argument does not affect the deduction of any other argument
- If a template argument couldn't be deduced, but has a default argument, then the default argument is fixed as the deduced argument
- The compiler verifies that each argument that was not explicitly specified and was not fixed by a default argument has been deduced at least once, and that all deductions agree
- Any function argument that participated in deduction must match its argument type exactly

#### Return-type deduction

The compiler can sometimes determine the return type if the return type depends on the template parameters
```cpp
template<lass T>
T min(T a, T b) {
	return (b < a) ? b : a;
}

// Let the compiler find a common type between T1 and T2
template<class T1, class T2>
auto min(T1 a, T2 b) {
	return (b < a) ? b : a;
}

template<class T1, class T2>
std::common_type_t<T1,T2> min(T1 a, T2 b) {
	return (b < a) ? b : a;
}
```

#### Template parameter scope

***Class templates***

![[Pasted image 20230930210104.png]]

***Member function templates***

![[Pasted image 20230930210243.png]]

***Member functions outside of class scope***

![[Pasted image 20230930210404.png]]

#### Dependent names

![[Pasted image 20230930210751.png]]

`const_iterator` depends on the parameter through `vector` -> `vector<T>` depends on `T` -> all of the iterators (e.g `const_reverse_iterator`) also depend on `T`. So `const_iterator` transitively depends on `T` as well.

Fix that by using `typename`:

```cpp
using const_iterator = typename vector<T>::const_reverse_iterator;
```

- Tells the compiler "`vector<T>::const_reverse_iterator` is really the name of a type, will you please use it like one 🥺" and it'll do it

***Dependent names used outside of the class***

![[Pasted image 20230930211649.png]]

![[Pasted image 20230930211728.png]]

***Static data members for class templates***

![[Pasted image 20230930211932.png]]

`m_count` is defined one time for every one specific instantiation of `Stack`

#### Full specialization
What if we wanted a specialization for `int`?

```cpp
template<>
class Stack<int> {
	vector<int> m_data;
	public:
	bool is_empty() const;
	int top() const;
	void pop();
	void push(int t);
 
	// Inline definition is trivial, see below for outside
	void push_from(string const& s);
};

// Note this is NOT inline unless we prefixed it explicitly
void Stack<int>::push_from(string const& s) {
	m_data.insert(m_data.end(), s.begin(), s.end());
}
```

#### Partial specialization
What if we don't care about the type of `T`, just that it's a pointer?

```cpp
template<class T>
class Stack<T*> {
	vector<T*> m_data;

  public:
	bool is_empty() const;
	T* top() const;
	T* pop();
	void push(T const& t);
};

// Partial specialization still needs the template parameter
template<class T>
T* Stack<T*>::pop() {
	T* tmp = m_data.back();
	m_data.pop_back();
	return tmp;
}

...

Stack<string*> pstack;

pstack.push(new string("Hello"));
cout << *pstack.top() << '\n';

delete pstack.pop();
```

This partial specialization is the one that will be instantiated 

Pair class example:
```cpp
template<class T, class U>
struct Pair {
	T first;
	U second;
	Pair(T const& t, U const& u) {...};
	...
}; 

template<class T, class U>
struct Pair<T&, U> {
	T first;
	U second;
	Pair(T const& t, U const& u) {...};
	...
};

template<class T, class U>
struct Pair<T, U&> { ... };

template<class T, class U>
struct Pair<T&, U&> { ... }
...

Pair<int, float> p0;
Pair<int&, float> p1;
Pair<int, float&> p2;
Pair<int&, float&> p3;

template<class T, class U>
void f(T t, U u) {
	Pair<T, U> p(t, u);
	...
}
```

By specializing on the types we expect, we can force the same internal representation
- e.g. if we get a `string&` but want to store a `string`, we can handle that

#### Type traits

![[Pasted image 20230930213707.png]]

Note this
```cpp
template <class T>
inline constexpr bool IsPointer_V = IsPointer<T>::value;
```

Is an alias template. At compile-time, the compiler chooses to generate code only for the branch whose type trait value returns true.

***Normalize properties of types***

Removing const and volatile

![[Pasted image 20230930214236.png]]

Removing references

![[Pasted image 20230930214218.png]]

![[Pasted image 20230930214444.png]]

**We can use traits to possibly improve our Stack**

```cpp
template<class T>
class Stack {
	using DataType = RemoveCV_T<RemoveRef_T<T>>;
	vector<DataType> m_data;

  public:
	using const_iterator = typename vector<DataType>::const_reverse_iterator;
	const_iterator begin() const;
	const_iterator end() const;

	bool is_empty() const;
	DataType const& top() const;

	void pop();
	void push(T const& t);
	Stack push_all_from(Stack const& other);
};
```

If we don't trust our users and want **just `T`**, then we can strip any qualifiers in front of it to ensure that our underlying data type is what we want.

#### Friends for class templates

```cpp
template<class T>
class Stack {
	...
	// Every other stack of a different parameter type is a friend of Stack<T>
	template<class U> friend class Stack<U>;

	// The regular class FooBar is a friend too
	friend class FooBar;

	// The parameter is a friend
	friend T; //- Ignored if T is not a class type;

	// Ordinary friend functions/friend function templates
	friend void foo();
	friend void bar<T>();

  public:
	template<class U>
	Stack(Stack<U> const& src);
	...
};
```

#### Summary
- C++ templates are a vast topic
- Recommendations
	- Start simple, with the topics in this talk
	- Understand value categories and references
	- Understand function templates, especially type deduction
	- Understand overload resolution and how it applies to function templates
	- Understand class templates, especially partial specialization
	- Write some type traits, try some metaprogramming
	- Read *Vandevoorde, Josuttis, and Gregor*
	- Watch any talk about templates by Walter Brown
	- Use cppreference.com
	- Ask questions and don't give up!
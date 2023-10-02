[Video link](https://www.youtube.com/watch?v=2h2hdRqRIRk)

1. [[#C-style cast]]
2. [[#C++ Functional Cast (a.k.a Constructor Call Notation)]]
3. [[#static_cast<T>]]
4. [[#const_cast<T>]]
5. [[#dynamic_cast<T>]]
6. [[#reinterpret_cast<T>]]
7. [[#bit_cast<T>]]
8. [[#std move & std move_if_no_except]]
9. [[#std forward]]
10. [[#std as_const]]
11. [[#std to_underlying (C++ 23)]]

Why do we need casts? C++ is a statically typed language.

Casts allow us to
1. Work with raw memory
2. Navigate inheritance hierarchy

![[Pasted image 20230925203657.png]]

What gives bytes meaning? Their type.

![[Pasted image 20230925203832.png]]

![[Pasted image 20230925204003.png]]

Compiler is implicitly converting the operation up to an integer to avoid edge-cases

![[Pasted image 20230925204055.png]]

![[Pasted image 20230925204128.png]]

## Explicit type conversions (casts)
```
```
#### C-style cast
##### `(<type>)var`
1. Create a temporary of `<type>` using var
2. `<type>` can be any valid type with qualifiers
3. Overrides the type system by changing the meaning of the bits in a variable
4. Will fail to compile under some circumstances (more later)
5. Can be used in constexpr context (more later)
6. Can cause undefined behavior
7. Considered an operator and participates in operator precedence (level 3)

```cpp
struct A{};

struct B{};

int main() {
	float f = 7.406f;
	int i = (int)f;
	A* pa = (A*)&f;
	B* pb = (B*)pa;
	double d = *(double*)(pb);
	return (int)d;
}
```

You can do basically whatever you want with pointer casts and it never generates warnings

```cpp
struct tree { bool has_leaves = true;};

struct car {int model_year = 1982; };

void prune(tree* t) { t->has_leaves = false; }

void drive(const car* c ) {
	printf("Driving %d\r\n", c->model_year);
}

int main() {
	const tree oak;
	car mustang;
	drive(&mustang); //normal function call
	prune((tree*)&oak); //pruning a const tree
	drive((car*)&oak); // driving a tree
	prune((tree*)&mustang); // pruning a car
	drive(&mustang); // driving a car from 1792
}
```
![[Pasted image 20230925210839.png]]

![[Pasted image 20230925210945.png]]
```cpp
void run_function(void* fptr) {
	auto* f = (void(*)(int))fptr;
	f(7);
}

void someFunc(int i) {printf("%d\r\n", i);}

int main( ) {
	run_function((void*)someFunc);
	return 0;
}
```

![[Pasted image 20230925211049.png]]

### C++ Functional Cast (a.k.a Constructor Call Notation)
#### `<type>(var)`
1. Creates a temporary `<type>` from var
2. Provides parity with C++ constructors
for built in types
3. Can only use a single word type name
4. Participates in operator precedence
(level 2)
```cpp
struct A{virtual ~A() = default;};

struct B:public A{};

struct C{};

template <typename T, typename F>
T convert_to(F& f) { return T(f); }

int main() {
	int i = 7;
	float f = convert_to<float>(i);
	//A* pa = A*(&f); //<-- Will not compile
	using astar = A*;
	A* pa = astar(&f);
	C* pc = convert_to<C*>(pa);
	B& rb = convert_to<B&>(*pa);
	return 0;
}
```

#### Problems with C-style and Functional notation casts
1. Single notation, multiple meanings
2. Error prone
3. Not grep-able
4. Complicate the C and C++ grammar

#### Goals for C++ casting
1. Different notation or different tasks
2. Easily recognized and searchable
3. Perform all operations that C casts can
4. Eliminate unintended errors
5. Make casting less enticing

#### C++ casting operators (keywords)
1. static_cast
2. const_cast
3. dynamic_cast
4. reinterpret_cast

### `static_cast<T>`

1. Creates a temporary of type from var
2. Tries to find a path from T1 to T2 via implicit and user-defined conversion or construction. Cannot remove CV qualification.
3. Use when you want to:
	1. Clarify implicit conversion
	2. Indicate intentional truncation
	3. Cast between base and derived
	4. Cast between void* and T*

```cpp
struct B {};
struct D : public B {};

int main() {
	int i = 7001;
	float f = static_cast<float>(i); // #1
	uint8_t ui8 = static_cast<uint8_t>(1.75f * f); // #2
	D d;
	B& rb = d;
	D& rd = static_cast<D&>(rb); // #3
	return 0;
}

void* some_c_handler(void* pv) {
	B* pb = static_cast<B*>(pv); // 4
	return pb;
}
```

##### `static_cast<T>` multiple hops

1. `A` has a single constructor that takes a single `int`
2. `E` has a user-defined conversion to `int`

```cpp
struct A { explicit A(int){ puts("A");}};

struct E {
	operator int() {
		puts("B::operator int");
	return 0;
}
};
int main() {
E e;
A a = static_cast<A>(e);
return 0;
}
```

##### `static_cast<T>` and inheritance
```cpp
struct Base1 { virtual ~Base1() = default; int i;};

struct Base2 { virtual ~Base2() = default; int j;};

struct Derived : public Base1, public Base2 { int k; };

void CheckSame(void* p1, void* p2) {
	if (p1 == p2) { puts("Same!");}
	else { puts("Different!"); }
}
int main() {
	Derived d;
	Derived* derivedptr = &d;
	Base1* base1ptr = static_cast<Base1*>(&d);
	CheckSame(derivedptr, base1ptr); // Same!

	Base2* base2ptr = static_cast<Base2*>(&d);
	CheckSame(derivedptr, base2ptr); // Different! Why?
	/*
		static_cast to one of the
		base types will offset the
		pointer into the derived type to
		the base type's location
	*/

	void* derived_plus_offset = (char*)derivedptr + sizeof(Base1);
	CheckSame(derived_plus_offset, base2ptr); // Same!?

	return 0;
}
```

![[Pasted image 20230925214408.png]]
##### `static_cast<T>` is not infalliable
`static_cast` does not protect against downcasting to an unrelated type
```cpp
struct Base {
	virtual ~Base() = default;
	virtual void f() { puts("base");}
};

struct Derived : public Base {
	void f() override { puts("Derived");}
};

struct other : public Base {
	void f() override { puts("other");}
};

int main() {
Derived d;
Base& b = d;
d.f(); // Derived
b.f(); // Derived

other& a = static_cast<other&>(b);
a.f(); // Derived

static_assert(std::is_same<decltype(a), other&>::value,
"not the same");
return 0;
}
```

### `const_cast<T>`
1. Removes or adds const or volatile qualifiers from or to a variable, cannot change type
2. Does NOT change the CV qualification of the original variable

```cpp
void use_pointer( int* pi) {
	printf("Use %d\r\n", *pi);
}

void modify_pointer( int* pi) { *pi = 42; }

int main() {
	const int i = 7;
	use_pointer(const_cast<int*>(&i)); // Use 7
	modify_pointer(const_cast<int*>(&i));
	printf("Modified %d\r\n", i); // Modified 7 (didn't do what we wanted)
	int j = 4;
	const int* cj = &j;
	modify_pointer(const_cast<int*>(cj));
	printf("Modified %d\r\n", i); // Modified 47
	return 0;
}
```

#### `const_cast<T>` example: member overload

Used to prevent code duplication for member functions

```cpp
class my_array {
	public:
		char& operator[](size_t offset) {
		return const_cast<char&>(
		const_cast<const my_array&>(*this)[offset]);
	}
	const char& operator[] (size_t offset) const {
		return buffer[offset];
	}
	private:
		char buffer[10];
};

int main() {
	const my_array a;
	const auto& c = a[4];
	my_array mod_a;
	mod_a[4] = 7;
	return 0;
}
```

#### Run Time Type Information (RTTI)
1. Extra information stored for each polymorphic type in an implementation defined struct
2. Allows for querying type information at run time
3. Can be disabled to save space (gcc/clang –fno-rtti, msvc /GR-)

Used in:

### `dynamic_cast<T>`

1. See if To is in the same public inheritance tree as From
	1. Will fail with private inheritance
2. Can only be a reference or pointer
3. Cannot remove CV
4. From must be polymorphic
5. Requires RTTI
6. Returns nullptr for pointers and throws std::bad_cast for references if the types are not related

```cpp
struct A { virtual ~A() = default; };

struct B : public A { };

struct C : public A {};

int main() {
	C c;
	B b;
	std::vector<A*> a_list = { &c, &b};
	for(size_t i = 0; i < a_list.size(); ++i) {
		A* pa = a_list[i];
		if( dynamic_cast<B*>(pa)) {
			printf("a_list[%lu] was a B\r\n", i);
		}
		if( dynamic_cast<C*>(pa)) {
			printf("a_list[%lu] was a C\r\n", i);
		}
	}
	return 0;
}
```
```
a_list[0] was a C
a_list[1] was a B
```

#### `dynamic_cast<T>` example: UI Framework

```cpp
struct Widget {};

struct Label : public Widget {};

struct Button : public Widget { void DoClick(); };

struct Page {
	std::vector<Widget> mWidgetList;
	template<typename T> T* getWidget(WidgetId id) {
		return dynamic_cast<T*>(&mWidgetList[id]);
	}
	void Page::OnTouch(WidgetId id) {
		auto* touchedWidget = getWidget<Button>(id);
		if(touchedWidget) {
		touchedWidget->DoClick();
		}
	//more processing
	}
```

![[Pasted image 20230925221136.png]]

### `reinterpret_cast<T>`

1. Can change any pointer or reference type to any other pointer or reference type
2. Also called type-punning
3. Cannot be used in a constexpr context
4. Can NOT remove CV qualification
5. Does not ensure sizes of To and From are the same
6. Useful for memory mapped functionality

```cpp
struct A{};

struct B{ int i; int j;};

int main() {
	int i = 0;
	int* pi = &i;
	uintptr_t uipt = reinterpret_cast<uintptr_t>(pi);
	float& f = reinterpret_cast<float&>(i);
	A a;
	B* pb = reinterpret_cast<B*>(&a);
	char buff[10];
	B* b_buff = reinterpret_cast<B*>(buff);

	volatile int& REGISTER = *reinterpret_cast<int*>(0x1234); // #6

	return 0;
}
```

#### `reinterpret_cast<T>` accessing private base

```cpp
struct B { void m(){ puts("private to D");}};

struct D: private B {};

int main() {
	D d;
	B& b = reinterpret_cast<B&>(d);
	b.m(); // Private to D
	return 0;
}
```

You probably shouldn't ever want to do this, but you **can**.

#### Type-aliasing

The act of using the memory of one type as if it were a different type when the memory layouts of the two types are compatible.

![[Pasted image 20230925221744.png]]

A selection of additional C++ "casts"
1. bit_cast
2. move/move_if_noexcept
3. forward
4. as_const
5. to_underlying (C++23)

### `bit_cast<T>`

1. Located in the `<bit>` header
2. Converts From into a bit representation in To
3. Requires To and From to be the same size
4. Requires To and From to be trivially copyable
5. Can be used in a constexpr context*
6. Fails to compile if cast is invalid
7. Can introduce UB
8. Requires C++20

\*As long as To and From are both not or do not contain:
- union
- pointer
- pointer to member
- volatile-qualified type or have a non-static data member of reference type

If you need this type of cast in C or pre C++20, use memcpy not a cast.

### `std::move` & `std::move_if_no_except`

1. Converts named variables (lvalues) to unnamed variables (rvalues)
2. If present, calls the move constructor of To, if not, calls copy constructor
3. Do not return `std::move(var)``
4. Equivalent to `static_cast<T&&>(var)`
5. `move_if_no_except` will trigger the copy constructor if the move constructor of the destination is not marked noexcept

```cpp
struct Employee {
	Employee(std::string aName)
	: mName(std::move(aName)) {}
	private:
	std::string mName;
};

int main() {
	std::vector<std::unique_ptr<Employee>> employees;
	auto person = std::make_unique<Employee>("me");
	employees.emplace_back(std::move(person));
	return 0;
}
```

### `std::forward`

Keeps the rvalue or lvalued-ness type passed to a template when passing to another function

```cpp
void f(int const &arg) { puts("by lvalue"); }

void f(int && arg) { puts("by rvalue"); }

template< typename T >
void func(T&& arg) {
	printf(" std::forward: ");
	f( std::forward<T>(arg) );
	printf(" normal: ");
	f(arg);
}
int main() {
	puts("call with rvalue:");
	func(5);
	puts("call with lvalue:");
	int x = 5;
	func(x);
}
```
```
call with rvalue:
 std::forward: by rvalue
 normal: by lvalue
 
call with lvalue:
 std::forward: by lvalue
 normal: by lvalue
```

### `std::as_const`

1. Adds const to var
2. Less verbose way of doing static_cast<const T&>(var)

```cpp
struct S {
	void f() const { puts(“const”); }
	void f() { puts(“non-const”); }
};

int main() {
	S s;
	s.f(); // non-const
	std::as_const(s).f(); // const
	return 0;
}
```

### `std::to_underlying (C++ 23)``

1. Proposed for C++23
2. Converts a sized enum to its value as the underlying type
3. Equivalent to `static_cast<std::underlying_type_t<T>>(var)`

```cpp
enum struct Result : int16_t { Ok = 1 };

enum Unscoped : int { Fail = -1 };

void print(int i) {
	std::cout << "int: " << i << "\n";
}
void print(int16_t i) {
	std::cout << "int16_t: " << i << "\n";
}
int main() {
	auto res = Result::Ok;
	print(std::to_underlying(res)); // int16_t: 1
	auto unscoped = Fail;
	print(std::to_underlying(unscoped)); // int: -1
	return 0;
}
```

![[Pasted image 20230925223930.png]]


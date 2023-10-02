[Video link](https://www.youtube.com/watch?v=i_wDa2AS_8w)
1. `using namespace std`
	1. Not bad at the function-level, but terrible at the global. Never **ever** use in a header file.
2. `std::endl` in a loop
	1. While this *does* add a newline, it also flushes the buffer which takes extra time. Just append a `'\n'` instead if you don't need to flush.
3. Index-based `for` when only range-based is necessary
4. Neglecting to use standard library algorithms
5. Using a raw array instead of `std::array`
6. Any use of `reinterpret_cast`
	1. If it's used on any potentially useful way, it's UB
	2. Use `std::bit_cast` instead if you need to 
7. Casting away `const`
8. Forgetting that `operator[]` on a `map` actually inserts the element with a default (`T()`) value if the lookup fails
	1. Use `at()` instead to throw an exception upon lookup failure
9. Ignoring `const`-correctness
	1. Only use the modifiers that are absolutely necessary
10. Not knowing that string literals (e.g. `"Hello"`) have static lifetimes, and that it's okay to return them from functions
11. Neglecting to use structured bindings
	1. Good source [here](https://www.cppstories.com/2022/structured-bindings/)
 12. Using out-params instead of a struct (along with structured bindings)
 13. Not using `constexpr` to compute values at compile-time
 14. Forgetting to mark destructors `virtual` for base classes
 15. Thinking that class members initialize in the order of the `initializer_list`
	 1. They're actually initialized in the order they're declared in
 16. Not knowing the difference between default and value-initialization
```cpp
int x;
int* x2 = new int;
// Default initialized; contain garbage

int y{};
int* y2 = new int{};
int* y3 = new int();
// Value initialized

int z();
// Function declaration, initializes nothing
```
17. Using magic numbers instead of named constants (i.e. using `constexpr`)
18. Modifying a container–and invalidating the iterators to it–while traversing it
19. Returning a local variable with `std::move` when RVO does it for you
20. Thinking that `std::move` actually moves something when it actually just casts casts something to an r-value
21. Thinking that eval order is left-to-right, or relying on evaluation order
```cpp
int a();
int b();
int c();
int g(int, int, int);

void function_evaluation_order_not_guaranteed() {
	// Not even in C++20...
	g(a(), b(), c());
}
```
22. Unnecessary heap allocations when stack allocations work fine
23. Not using `std::unique_ptr` and `std::shared_ptr`
24. Forgetting to use `std::make_unique` and `std::make_shared`
25. Any use of `new` and `delete`
26. Any kind of manual resource management
27. Using smart pointers when raw pointers work fine
	1. E.g. if the ownership the memory used isn't relevant to the code, then use a raw pointer
	2. Note: C code doesn't share the convention of "raw pointers don't own what they point to"
		1. You *cannot* let a `std::unique_ptr` `delete` a pointer that was allocated with `malloc`
		2. You **can** still define a custom deleter that uses `free` so that you can still use the `unique_ptr`
28. Using a `std::shared_ptr` when you're uncertain whether the memory needs to be shared
	1. Use `std::unique_ptr` as the default, it's easy to convert it to a `std::shared_ptr` later
29. Thinking `std::shared_ptr` is completely thread-safe
	1. The reference-counting is, but nothing else is built-in
30. Misunderstanding how `const` is applied ([just use east-const](https://hackingcpp.com/cpp/design/east_vs_west_const.html))
31. Ignoring compiler warnings, which often leads to UB
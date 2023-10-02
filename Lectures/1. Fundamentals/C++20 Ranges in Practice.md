3 Problems
1. [[#Minimum array element]]
2. [[#Sum of squares]]
3. [[#String trimming]]

### Minimum array element

Easy solution:
```cpp
std::vector<int> vec = get_input();

auto iter = std::min_element(vec.begin(), vec.end());
```

Ranges version:
```cpp
namespace rng = std::ranges;

std::vector<int> vec = get_input();

// auto iter = rng::min_element(vec.begin(), vec.end());
// Still works, but it's better

auto iter = rng::min_element(vec);
```

What if we don't need the vector anymore after?
```cpp
namespace rng = std::ranges;

// Takes r-value reference
auto iter = rng::min_element(get_input());

// Dangling iterator?
```

Ranges protects you against above problems. When passing an r-value range into an algorithm which normally returns an iterator, ranges will return a `std::ranges::dangling` object

Attempting to use a `dangling` object will cause a compile error.
```cpp
auto iter = rng::min_element(get_input());
std::cout << *iter;
```
```
error: no match for 'operator*'
(operand type is 'std::ranges::dangling')
```

Some types have iterators which *can* safely outlive their parent, for example `std::string_view` and `std::span`
```cpp
std::string str = "Hello world";
// Weird, but okay
auto iter = std::string_view{str}.begin();
std::cout << *iter << '\n';
```

We can disable "dangling protection" for such types by specializing the `enable_borrowed_range` trait
```cpp
// Outside of any namespace
template <>
inline constexpr bool
std::ranges::enable_borrowed_range<my::StringRef> = true;
```

A borrowed range means a range that doesn't own its elements, and whose iterators can safely outlive the range.
```cpp
auto iter = rng::find(get_string_ref(), 'x');
// iter is an iterator, *not* ranges::dangling
```

A **view** is a range such that
- Its move operations are constant-time
- (if it is copyable) its copy operations are constant-time
- Destruction of a moved-from object is constant-time
- Otherwise, destruction is O(N)

Example:
`std::vector<int>` is not a view because its copy operation is O(N)
`std::vector<unique_ptr<int>>` could be a view be a view because it's not copyable

The compiler doesn't know the difference, we have to make a type opt-in to being a view using the `enable_view` trait, or by inheriting from `ranges::view_base`
- `std::ranges::view_interface` is a helper to do so!

We can turn any __movable range__ into a *view* using `std::views::all`
```cpp
std::vector<int> vec = get_input();

auto first_view = std::views::all(vec);
// Always okay, first_view is movable and copyable

auto second_view = std::views::all(get_input());
// Okay since P2415, second_view is move-only
```

View adaptors implicitly convert their arguments to views using `std::views::all`
```cpp
auto vec = get_vec();

auto v1 = vec | views::transform(func);
// Always okay: vec is an lvalue => borrowed

auto v2 = get_vec() | views::transform(func);
// Okay since P2415, v2 is move-only
```
### Sum of squares

One solution
```cpp
auto vec = get_input();

std::transform(vec.begin(), vec.end(), vec.begin(),
			   [](int i) { return i * i; });
auto sum_sq = std::acccumulate(vec.begin(), vec.end(), 0);
```
Transform in-place

Save ourselves some characters
```cpp
auto vec = get_input();

rng::transform(vec, vec.begin(),
	[](int i) { return i * i; });
auto sum_sq = std::acccumulate(vec.begin(), vec.end(), 0);
```
But this still eagerly evaluates and transforms in-place

**Using views instead**

Lazily applying transformation
```cpp
auto vec = get_input();

auto view = std::views::transform(vec,
		``[](int i) { return i * i; });

auto sum_sq = std::acccumulate(view.begin(), view.end(), 0);
```
Only applies transformation when dereferencing iterators in accumulate

Piping syntax
```cpp
auto vec = get_input();

auto view = vec
	| std::views::transform([](int i) { return i * i; })
	| std::views::common;
// views::common allows us to guarantee that the view has iterator/sentinel pairs
// that a C++17 algorithm can understand

auto sum_sq = std::acccumulate(view.begin(), view.end(), 0);
```

### String trimming

Trim front and back of the range of all whitespace

Implementation using ranges
```cpp
template <typename R>
auto trim(R && rng) {
	return trim_back(trim_front(forward<R>(rng)));
}
```

```cpp
template <typename R>
auto trim_front(R && rng) {
	// Uses C-libary isspace
	return views::drop_while(forward<R>(rng), ::isspace);
	// return forward<R>(rng) | views::drop_while(::isspace);
}
```

No special `keep_until` adapter or anything similar, but there's a way around this

```cpp
template <typename R>
auto trim_back(R && rng) {
	return forward<R>(rng)
		| views::reverse
		| views::drop_while(::isspace)
		| views::reverse;
}
```

```cpp
std::string trim_str(std::string const& str) {
	return trim(str) | rangesnext::to<std::string>;
	// C++23 feature to convert a range to the provided type
}
```

What if instead we want to just return the range adaptor without worrying about the range?

```cpp
auto trim_front() {
	return views::drop_while(::isspace);
}

auto trim_back() {
	return views::reverse
		| views::drop_while(::isspace)
		| views::reverse;
}

auto trim() {
	return trim_front() | trim_back();
}

std::string trim_str(std::string const& str) {
	return str
		| trim()
		| rangesnext::to<std::string>;
}
```

We don't necessarily need functions that always return a constant. Can just create functors that are compositions of standard library adaptors.
```cpp
inline constexpr auto trim_front = views::drop_while(::isspace);

inline constexpr auto trim_back =
		  views::reverse
		| views::drop_while(::isspace)
		| views::reverse;

inline constexpr auto trim = trim_front | trim_back;

std::string trim_str(std::string const& str) {
	return str | trim | rangesnext::to<std::string>;
}
```

[Video link](https://www.youtube.com/watch?v=IgNUBw3vcO4)

Traditionally, if we wanted a pass a function as a parameter to another function, we would use function pointers

```cpp
class Person {
  public:
	std::string firstname() const;
	std::string lastname() const;
	
	friend bool operator< (const Person&, const Person&);
	// or operator<=> since C++20
};

bool lessPerson(const Person& p1, const Person& p2) {
	return p1.lastname() < p2.lastname() ||
		(p1.lastname() == p2.lastname() && p1.firstname() < p2.firstname());
}

std::vector<Person> coll;

// sort elements with operator <
std::sort(coll.begin(), coll.end());

// sort using a custom ordering via a function pointer
std::sort(coll.begin(), coll.end(), lessPerson);
```

But function pointers kinda suck, so let's use lambdas instead.

```cpp
class Person {
  public:
	std::string firstname() const;
	std::string lastname() const;

	std::string getCustNo() const;
};

std::vector<Person> coll;

// Sort by name
std::sort(coll.begin(), coll.end(),
		 [] (const Person& p1, const Person& p2) {
			 return p1.lastname() < p2.lastname() ||
			(p1.lastname() == p2.lastname() && p1.firstname() < p2.firstname());
			});

// Sort by customer number
std::sort(coll.begin(), coll.end(),
		[] (const Person& p1, const Person& p2) {
			  return p1.getCustNo() < p2.getCustNo();
		});
```

![[Pasted image 20231002113240.png]]

**Lambdas are *function objects* defined on the fly**

```cpp
std::vector<int> coll{0, 8, 15, 42, 11, 1, 77, -1, 3};

// isOdd is an *object* that can be used like a function
auto isOdd = [] (int elem) {
	return elem % 2 != 0;
	}

// count number of elements with odd value
int num = std::count_if(coll.begin(), coll.end(), isOdd);

// find position of first element with odd value
auto pos = std::find_if(coll.begin(), coll.end() isOdd,

```

#### Combining lambdas with standard algorithms

```cpp
std::vector<std::string> coll{"Here", "are", "some", "cities", "Berlin", "LA",
"London", "Cologne"};

std::sort(coll.begin(), coll.end());

std::sort(coll.begin(), coll.end(),
	[] (const std::string& s1, const std::string& s2) {
		return std::lexicographical_compare(s1.begin(), s1.end(),
			s2.begin(), s2.end(), [] (char c1, char c2) {
				return std::toupper(c1) < std::toupper(c2);
			});
	}); 
```

Using ranges

```cpp
std::vector<std::string> coll{"Here", "are", "some", "cities", "Berlin", "LA",
"London", "Cologne"};

std::ranges::sort(coll);

std::ranges::sort(coll,
	[] (const std::string& s1, const std::string& s2) {
		auto toUpper = [](char c){return std::toupper(c);};
		return std::ranges::lexicographical_compare(s1, // 1st range
		s2, // 2nd range
		std::less{}, // comparison criterion
		toUpper, // projection for s1 element
		toUpper); // projection for s2 element
	});
```

### Named generic lambdas

```cpp
// define a generic lambda object
auto twice = [] (const auto& x) {
	return x + x;
};
// lambda is compiled for different parameter types

auto i = twice(3); // int => 6
auto d = twice(1.7); // double => 3.4
auto s = twice(std::string{"hi"}); // string => "hihi"
auto t = twice("hi"); // ERROR: char const[3] + char const[3]

// print all elements of any kind
for (const auto& elem : coll) {
	std::cout << "- " << twice(elem) << '\n';
}

// replace all elements of coll by the sum of adding them to themselves
std::transform(coll.begin(), coll.end(), // source range
			coll.begin(),
			twice);

```

**Lambdas with no captures**
- Can be used as ordinary *function pointers*
- Can be used as a sorting criterion/hash function type

```cpp
#include <cstdlib>
#include <unordered_set>

int main() {
	// lambda do be called on regular exit
	std::atexit([](){
		std::cout << "good bye\n";
	});

	// lambda to be used as a hash function
	auto hashFunc = [](const auto& obj) {
		return ...;
	};
	...
	std::unordered_set<MyType, decltype(hashFunc)> coll;
}
```
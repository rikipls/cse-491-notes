
Optimizations
1. [[#Small String Optimization]]

```cpp
struct String {
	int size;
	int capacity;
	char* data;
}
```
Intuitive, but this isn't how string is *actually* represented.

The `sizeof` a gcc string (version < 5) is actually 8 bytes, the size of a pointer.

The `size` and `capacity` of the string are actually stored on the heap, along with the pointer to the data.

![[Pasted image 20230910115253.png]]

In a naive implementation, the empty string is not actually empty. It's 1 byte because of the '\\0' terminator.
Why not have an empty string just be a `malloc(0)`? Because that's overhead for **every time you default-construct** a string.
>Instead, gcc has a **global empty string**.

Why not have `nullptr` be the empty string? Because then it creates a branch in *every string code* to check for null.
>That's why there's a 25-byte global empty string in your program.

Additionally, gcc 5 they also have a reference count to implement copy-on-write semantics, to defer copies until they're necessary.

For a while, gcc strings were used everywhere at Facebook, then Andrei Alexandrescu made `fbstring` to beat it.

### Small String Optimization

If a string is small enough, create a union between the normal string (data, size, capacity triplet) and a stack-allocated **small string**

![[Pasted image 20230910121018.png]]

The last character in the string is the amount of **spare capacity in the string**. When characters are pushed onto the string, that spare capacity decreases and once all 23 bytes are used, that count then doubles as a **null-terminator**

![[Pasted image 20230910120934.png]]

However, we also at least one bit to count as a flag, telling whether it's a small string or a normal string. Where does it go?
We have 23 bytes of user data and 1 byte, 8 bits, of zeros. Conveniently, flag bits can be zero. The spare capacity, which ranges from 0 to 23, takes 5 bits to store. We have 3 bits remaining for some flag variables.

![[Pasted image 20230910123902.png]]

fbstring has a better memory layout than gcc 5, so when used on a benchmark test that used over 100,000 strings, where the effect of page/cache misses are felt. However, when running over a small number of strings in quick succession, gcc 5's implementation is faster because all of the data fits within memory.

So worse assembly, but better memory layout. Which wins?

![[Pasted image 20230910124303.png]]

This performance improvement was across **all** of facebook's codebase.

![[Pasted image 20230910123524.png]]

`Move` is called 1-2 orders of magnitude less than `size`, so it's okay for it not to be a memcopy. Biggest problem is that there are simply fewer characters available in-situ.
# My C++ Notebook


## Keywords

### CV-Qualifiers
Const-Volatile Qualifiers are two keywords, `const` and `volitile` that can be added to a decleration of a variable.

#### `const`
Means that the qualified variable cannot change.

#### `volatile`
Instructs the compiler to not optimize-away checks done to the qualified variable because it might be changed by an external entity which the compiler is not aware of.
See [this](https://stackoverflow.com/questions/72552/why-does-volatile-exist) for example.

The position of the cv-qualifier keyword changes its meaning.
The following applies to both `const` and `volatile`:
- Const variable:
	`const int var`
- Const reference:
	`const Widget& var`
- Const pointer:
	`char *const var`  
	Read from right to left: var is a const pointer to a char.
- Pointer to const:
	`const char *var`  
	Read from right to left: var is a pointer to a char which is const.
- Const method:
	`void myMethod() const`
	Promise that `myMethod` won't change `this`.

### `constexpr`
`constexpr` means "know at compile-time".
As apposed to `const` which means "[someone] promises not to change [something]".

### `noexcept`
Declares that a function does not throw an exception.
Can be used as a function to check if an expression may throw an exception.

### `typename`
Specify that the following is a type.
See [this](https://stackoverflow.com/questions/1600936/officially-what-is-typename-for) for example.

### `typedef`
// TODO


## The Standard Library

### `std::remove_reference<T>`
If `T` is a Reference Type, evaluates to the 
original type. Otherwise, just evaluates to `T`.

### `#include <iostream>`
Input and output of data. For example:
```C++
std::cin >> my_int;  // Input
std::cout << "Hello World!" << std::endl;  // Output
```

#### Caveats
* When `std::cin` gets an input that cannot be converted to the target type it goes into "failed state" and from that point on will always return immediately without doing anything.
* [Flush the input stream when reading in a loop](https://isocpp.org/wiki/faq/input-output#istreams-remember-bad-state)
* `std::endl` outputs `'\n'` and flushes the stream. 
Use `'\n'` if you don't need to flush right-away as this slows down the code.
* To allow printing of a user-defined class [provide a `friend operator<<`](https://isocpp.org/wiki/faq/input-output#output-operator)

## Concepts and Code Constructs

### Automatic Objects, RVO
// TODO

### Pointers vs. References
A reference to an object is the same as the original object.
A pointer to an object is an object that holds the address of the original object.
Use references when you can, and pointers when you have to.

### Value Categories
#### 'lvalue':
Something that has a location in memory and that you can take its address. Typically has a name and typically survives beyond the current statement. For example, a variable name.
#### 'rvalue':
Something that does not have a location in memory. Typically does not have a name and does not survive the current statement. For example, `23`.

### References

#### Pass by lvalue referance:
Written with `&`. Implies that changes will be made to the argument passed. Therefore, it is an error to pass an rvalue by lvalue reference, because the caller will not be able to see the changes.

#### Pass by const reference:
Written with `const &`. Enforeces that no changes are made to the argument passed. Can be used with an rvalue or an lvalue.

#### Pass by rvalue reference:  # From C++11
Written with `&&`. Enforces that the caller will not refer to the passed argument after the call. The argument is "moved" to the invoked function's possession. Can only be passed an rvalue, to ensure that the caller cannot refer to the argument again. Enables us to implement easily **Move Semantics** and **Perfect Forwarding**.


### Move Semantics
Copying a resource can be expensive, and is unnecessary when it is an rvalue (= is going to be destroyed at the end of the statement). When an rvalue resource is used to construct a new object, the object may move the resource to its own possession, and "emptying" the original reference to it.
To use this functionallity explicitly, the client might use `std::move` to pass an rvalue reference and so trigerring the moving of the resource.
* [Source for Move Semantics](https://stackoverflow.com/questions/3106110/what-is-move-semantics)
* [Source for Value Categories and Move Semantics](https://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c)


### TODO: Perfect Forwarding
http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html


### Copy-and-Swap Idiom
A practice used to ensure safe copying and moving of objects.
When defining a Copy-Assignment Operator, it is important to note that the contruction of the new object might fail (as memory might be unavailable), so copying the assigned object should be done before deleting the state of the assginee. Otherwise we might end up with corrupted state. Here is the algorithm for Copy-Assginment, according to Copy-and-Swap:
* **Copy** the assgined object into a temporary.
* **Swap** the state of the assginee with the state of the temporary.
* Delete the temporary.
The best practice for implementing the algorithm is to have the assgined object copied by passing it as an argument to the `operator=` method, and create the temporary as a local so it is deleted when the method exits.
[For more details](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom)


### Casting
Changing the data type of values. There are a few types of casts (from safest to most dangerous):
- `static_cast`
	Will only compile if it can be validated at compile-time.
- `dynamic_cast`
	Used to cast a base class to a child class. Might return a `nullptr` if fails.
- `const_cast`
	Used to remove const. Might indicate that something is wrong with the data type design of the program...
- `reinterpret_cast`
	The compiler performes the cast without any validation.
	"Converts between types by reinterpreting the underlying bit pattern."
- `(int)`
	C-style cast. Can be any of the above.


### Multi-Threading
// TODO
// C++11 introduces a new memory model to handle multithreading.
// https://www.justsoftwaresolutions.co.uk/threading/multithreading-in-c++0x-part-1-starting-threads.html

### Implicit Conversions
// TODO

### `class` vs. `struct`
A `class` have default private members while `struct` has default public members.
The recomendation is to use `struct` as a plain data structure and to use `class` for all other features.

### Templates
A template is a tool for defining generic implemntations for some code. The compiler fills-in the generic parts with specific types or data accoring to the usage of the template.
A template should look like so:
```C++
template <template_parameter1_type template_parameter1_name, ...>
```
For example:
```C++
template <class T, int N>
```
**Note** that when defining a template in a separate file, the definision **must not** be separated from the implementation. Rather, they should both be in the `.h` header file. This is because templates are compiled only when they are used with specific parameters. So writing a template in a `.cpp` file won't produce anything because it will never be compiled.

* Question:
A templated class should be defined and implemented in the header file.
This is because the compiler cannot know which types to implement the template for during the compilation of the class' .cpp file.
But it turnes out that this is not true if the template class and its user are in the same directory.
Why is that?

* Tamplates can also be specialized for specific parameters.
* For more information see [this tutorial](http://www.cplusplus.com/doc/oldtutorial/templates).


# Other Weird Stuff

* This is always true: `sizeof(char) == 1`.
* A "byte" in C++ might have more than 8 bits.
* [POD type](https://isocpp.org/wiki/faq/intrinsic-types#pod-types) = Plain Old Data type. A type in C++ that can be represented in C.
* You cannot define an operator overload of a primitive type.
* 
 

# C++ / C Interview Notes

## The Three Core Features of C++

### Encapsulation

My first exposure to programming was C. When I first started writing logic-based code, it felt cumbersome — you had to define variables inside functions, write the corresponding logic sequentially, and then wrap it into functions. Everything felt scattered. Later, when I learned about structs, I realized I could group all the data needed for a piece of logic into a single struct, which made things cleaner. But a new problem emerged: while the data was now bundled together, the logic — the functions that processed that data — still lived outside. This led to the emergence of classes, which encapsulate both data and the corresponding behavior together.

**Encapsulation** combines abstracted data and behavior into an organic whole — it organically integrates data with the source code that operates on it to form a class, where both data and functions are class members. The goal is to separate the object's *users* from its *designers*, hiding implementation details (including private members), enhancing the security of code modules, and improving maintainability and modifiability.

In summary, encapsulation has two or three key characteristics:
1. **Integration**: combining attributes and methods into a unified entity
2. **Information hiding**: using interface mechanisms to conceal internal implementation details, exposing only the interface for external calls
3. **Code reuse**: reusing implementation code across different contexts

---

### Inheritance

**Class derivation** refers to creating a new class from an existing one. The original class becomes the *base class* (or parent class), and the newly created class is called the *derived class* (or child class). After a child class inherits from a base class, you can create instances of the child class to access the base class's functions, variables, and other members.

There are generally three types of inheritance:

1. **Single Inheritance**: Inheriting from a single parent class. This is the most commonly used form of inheritance.
2. **Multiple Inheritance**: A derived class inherits from multiple base classes. Class names must be separated by commas, and the inheritance access specifier must precede each class name. If two or more base classes share a variable or function, the **class name qualifier** must be used when calling them in the derived class, e.g., `obj.classA::i = 1`.
3. **Diamond Inheritance**: A form of multiple inheritance involving a 1-n-1 cross-generation inheritance pattern (e.g., B and C both inherit from A, and D inherits from both B and C). This requires the use of **virtual inheritance** to avoid ambiguity errors.

Inheritance access scope rules:

| Access in parent class | Inheritance type | Access in child class |
|:---:|:---:|:---:|
| public | public | public |
| protected | public | protected |
| private | public | no access |
| public | protected | protected |
| protected | protected | protected |
| private | protected | no access |
| public | private | private |
| protected | private | private |
| private | private | no access |

---

### Polymorphism

Polymorphism can be summarized as **"one interface, multiple implementations"** — the same interface produces different results depending on the context. There are two forms: **static polymorphism** and **dynamic polymorphism**.

**Static Polymorphism**: The design philosophy is that for related object types, each type directly implements its own definition. There is no need for a shared base class, and the types may not be related at all. Each concrete class only needs to conform to the same interface declaration. Static polymorphism is essentially the instantiation of templates.

**Dynamic Polymorphism**: For related object types, a set of common functions is identified and declared as `virtual` functions in the base class. Each subclass overrides these virtual functions to perform specific behavior. This is realized through C++'s virtual function mechanism.

Polymorphism builds on encapsulation and inheritance — it is the ability to express a single form in multiple ways. In practical terms: when calling a function through a parent class pointer, the actual member function of the type the pointer *points to* (the subclass) is invoked.

C++ polymorphism includes:
1. **Overloading**: function overloading and operator overloading — resolved at compile time.
2. **Virtual functions**: subclass runtime polymorphism — resolved at runtime. In an inheritance hierarchy, we want a method's behavior to depend on the actual object type, not the declared type of the pointer or reference.
3. **Templates**: class templates and function templates — resolved at compile time.

> **Analogy**: Suppose a relative is getting married and invites your father. In reality, you go instead, or your sister goes — either is acceptable because you are representing your father. Before you arrive, they don't know who will show up; they only know it will be someone from your family. This is polymorphism.

---

## Object-Oriented vs. Procedural Programming

Both are programming paradigms (ways of thinking about code).

**Procedural programming** analyzes the steps needed to solve a problem, implements each step as a function, and calls them in sequence.

**Object-oriented programming** decomposes the problem into objects. Objects are not defined to complete a single step — they describe the behavior of an entity throughout the entire problem-solving process.

**Example — Gomoku (Five in a Row):**

*Procedural approach*: Identify the steps — 1. Start game, 2. Black moves, 3. Draw board, 4. Check win, 5. White moves, 6. Draw board, 7. Check win, 8. Return to step 2, 9. Output result. Each step is a separate function.

*Object-oriented approach*: Decompose into objects — 1. Two players (black and white), whose behavior is identical; 2. A board system responsible for rendering; 3. A rules system responsible for judging violations and win conditions. The player object receives input and notifies the board object of the move; the board object renders the change and delegates to the rules object for judgment.

Object-oriented programming partitions problems by **functionality**, not by steps. For example, "drawing the board" only exists inside the board object, ensuring consistency. This leads to better extensibility.

**Procedural**
- Pros: Higher performance — no class instantiation overhead. Used in embedded systems, drivers, Linux/Unix kernels where performance is critical.
- Cons: Harder to maintain, reuse, and extend.

**Object-Oriented**
- Pros: Easier to maintain, reuse, and extend. Encapsulation, inheritance, and polymorphism enable low-coupling system design.
- Cons: Lower raw performance compared to procedural.

---

## struct vs. union

**struct**: Groups different types of data together. Each member has its own independent memory address. `sizeof(struct)` equals the sum of all member sizes after memory alignment. (This leads into the topic of [memory alignment](#memory-alignment-rules-for-struct).)

**union**: All members share the same block of memory. The size of a union equals the size of its largest member. The "sharing" doesn't mean multiple members are stored simultaneously — only one member can hold a value at any given time. Assigning a new value overwrites the previous one. `sizeof(union)` = size of the largest member.

> **Summary**: Both `struct` and `union` are composed of multiple members of different types. However, at any given moment, a `union` only stores one selected member, while a `struct` stores all members simultaneously. In a `struct`, each member occupies its own memory space and all exist at the same time — the total size equals the sum of all member sizes. **In a `union`, all members cannot occupy memory simultaneously.** The union's size equals the largest member's size. Assigning to one union member overwrites all others; in a struct, each member is independent.

---

## Memory Alignment Rules for struct

**Why is byte alignment needed?**

The fundamental reason is CPU memory access efficiency. Without alignment, a `double` variable might be stored at addresses 4–11 instead of the natural 0–7. The CPU would need two memory reads to retrieve it, reducing efficiency. With natural alignment, only one read is needed. Some systems (e.g., SPARC) enforce strict alignment — accessing unaligned data causes a hardware error.

**Alignment rules**

On x86, GCC aligns to 4 bytes by default. This can be overridden with `__attribute__` in GCC or `#pragma pack(n)` in MSVC.

Examples:
```cpp
// sizeof(stu) = 4 + 4 + 12 = 20
struct stu {
    char sex;       // padded to 4 bytes
    int length;     // 4 bytes
    char name[10];  // padded to 12 bytes
};

// sizeof(node1) = 1 + 1 + (2 padding) + 4 = 8
// Two chars + 2 bytes of padding to align int
struct node1 {
    char c1;
    char c2;
    int a;
} str1;

// sizeof(str4) = 5 + 2 + 1 (padding) + 4 = 12
// 5-char array + short = 7 bytes, needs 1 byte padding before int
struct str4 {
    char c1[5];
    short c;
    int b;
};

// sizeof(str6) = 1 + (7 padding) + 8 + 1 + (7 padding) = 24
struct str6 {
    char c1;
    double a;
    char c2;
};
```

**Common tricky cases**
```cpp
// sizeof = 8
struct str {
    char p;
    int a;
    int b[0];   // zero-length array: no size, but no padding omitted
};

// sizeof = 4
struct str {
    int b[0];
};

// sizeof = 1
struct str {
    // empty struct: at least 1 byte by C++ rules
};
```

Zero-length arrays (`int[]` or `int[0]`) can only be defined inside a class or struct — they are illegal outside. They occupy no space and require no initialization. The name of a zero-length array is a pointer that points to the memory address immediately after the previous member.

> You cannot define `int a[0]` or `int a[]` inside a function body.

---

## arr vs. &arr[0] vs. &arr

All three print the same value — the address of the first element of the array. The difference appears when arithmetic is applied:

```cpp
int arr[10] = {0};
printf("%p\n", arr);       // address of first element
printf("%p\n", arr + 1);   // +4 bytes (next int)
printf("%p\n", &arr[0]);   // address of first element
printf("%p\n", &arr[0]+1); // +4 bytes
printf("%p\n", &arr);      // address of the whole array (same value)
printf("%p\n", &arr + 1);  // +40 bytes (jumps the entire array!)
```

**Conclusion**: `&arr` represents the address of the *entire array*. Although its printed value is the same as the first element's address, pointer arithmetic on it operates in units of the entire array.

Additional note: `arr` itself is an lvalue denoting the array object, but in most contexts it implicitly converts to an rvalue `&(arr[0])` of pointer type. `&arr` is an rvalue pointer to the array itself. So `arr` is not an address — it *represents* the array, but decays to a pointer.

---

## char a, char a[], char *a, char *[], char **a Explained

1. **`char a`** — a single variable storing one `char` value.
2. **`char a[]`** — a character array; each element is a `char`.
3. **`char *a`** — the essence of a string (as the computer sees it) is the address of its first character. C/C++ operates on strings via their starting address in memory. Both `char a[]` and `char *a` represent the starting address of a string — there is no fundamental difference. However, `a = s` is valid (pointer reassignment), but `s = a` is not (array names are non-assignable fixed addresses).
4. **`char *a[]`** — `[]` binds tighter than `*`, so this is an array of pointers, each pointing to a `char` (i.e., an array of C strings). Example: `char *a[] = {"China", "French", "America", "German"};`
5. **`char **a`** — equivalent to `char *(*a)`, i.e., a pointer to a pointer to `char`. This has the same logical structure as `char *a[]`.

---

## 1D vs. 2D Array Names

Regardless of dimensionality, arrays occupy a contiguous linear block of memory, so at the hardware level everything is one-dimensional.

- A 1D array name is a pointer to the first element; `+1` advances by `sizeof(element)`.
- A 2D array name is also a pointer, but `+1` advances by one *row* (i.e., by `sizeof(element) * columns`).

**Why can't a 2D array name be directly assigned to a double pointer (`**`)?**

A 2D array name is a *pointer to an array* (row pointer), not a *pointer to a pointer*. They are different types.

> For `int a[i]` and `int b[i][j]`: `a` is equivalent to `int (*)` and `b` is equivalent to `int (*)[j]`. To get element `a[x]`, use `*(a+x)`. To get `b[x][y]`, use `*(*(b+x)+y)` — `b+x` points to the x-th row array, `*(b+x)` gives the first-element address of that row, and adding `y` reaches the target element.

---

## Array Pointers vs. Pointer Arrays

- **Array pointer** (pointer to an array): a pointer that points to an entire array — `int (*p)[n]`.
- **Pointer array** (array of pointers): an array whose every element is a pointer — `int *p[n]`.

---

## C++ Memory Layout / Program Segments

Also known as the process logical address space. From high to low addresses:

```
| Stack          |  ← high address
| Heap           |
| BSS segment    |
| Data segment   |
| Text segment   |  ← low address
```

- **Stack**: stores local variables, function parameters, and return values. Cleaned up automatically after each function call. Grows from high to low addresses.
- **Heap**: dynamically allocated memory (via `new`/`malloc`). Grows from low to high addresses.
- **BSS segment** (Block Started by Symbol): uninitialized global variables. Zeroed by the kernel at program load time.
- **Data segment**: initialized global variables and `static` variables.
- **Text segment**: compiled machine code. Mapped as read-only.

> BSS and data are both statically allocated (at compile time). Together they form the "data segment," which can be subdivided into:
> 1. Read-only data: constants and `const`-qualified global variables
> 2. Read-write data: initialized global and `static` variables
> 3. BSS: uninitialized global variables

---

## `static` and `const` — Usage and Interaction

### `static`

**On local variables**:
- Stored in the data segment (not the stack).
- Initialized only once across all function calls.
- Scope remains local (ends when the enclosing block exits), but the variable persists in memory until program termination.
- In short: `static` changes a local variable's *storage location* and *lifetime* but not its *scope*.

**On global variables**:
- Already stored in static memory; `static` restricts the variable's visibility to the file in which it is declared. Non-static globals can be accessed across files via `extern`; static globals cannot.

**On functions**:
- Restricts the function's visibility to its own translation unit. Prevents name collisions in large projects.

**On class member variables**:
- Makes the variable a class-level global, shared by all instances (including derived class objects).
- Must be initialized *outside* the class definition (not in the constructor), except when combined with `const`.

**On class member functions**:
- Only one copy of the function exists for all objects; it has no `this` pointer.
- Can be called without creating an instance.
- **Cannot be `const` at the same time** (see below).

---

### `const`

1. **On variables**: marks a variable as immutable after initialization.
2. **On pointers**: either a pointer-to-const or a const pointer (see [Const and Pointers](#const-and-pointers)).
3. **On functions**:
   ```cpp
   const int& fun(int& a);        // const return value
   int& fun(const int& a);        // const parameter
   int& fun(int& a) const {}      // const member function
   ```
4. **On classes**:
   - `const` member variable: constant within an object's lifetime, but different objects can have different values. Must be initialized in the initializer list (not in the body of the constructor).
   - `const` member function: guarantees it won't modify any data members. A `const` object can only call `const` member functions.

---

### Can `static` and `const` simultaneously modify a member function?

**No.** A `const` member function implicitly receives a `const this*` pointer to prevent modification of instance state. A `static` member function has no `this` pointer at all — it is unrelated to any object instance. The two semantics are contradictory: `static` says "this function has nothing to do with an instance"; `const` says "this function must not modify the instance". There is no reconciling them.

---

## `static` Initialization Timing and Thread Safety

**In C**: static local variables and global variables are all stored in the global memory region. The compiler allocates their memory and performs initialization before `main()` runs (during the static initialization phase). Therefore, in C you cannot use a runtime variable to initialize a static local variable.

**In C++**: The introduction of classes (with constructors and destructors) required a more nuanced approach. C++ distinguishes two initialization phases:

- **Static (compile-time) initialization**: zero-initialization and constant initialization of global/`const` variables. These are placed in the BSS or data segment and initialized during program load, similar to C.
- **Dynamic (runtime) initialization**: initialization that requires a function call (e.g., class constructors), primarily for local static variables and local static class objects.

**Thread safety of local static variables**

C++11 guarantees that the initialization of a local static variable is thread-safe. When one thread is initializing the variable, other threads that reach the same initialization point will block (not skip). The mechanism uses a guard variable (`guard_for_bar`) with locking via `__cxa_guard_acquire` and `__cxa_guard_release`.

```cpp
void foo() {
    static Bar bar;
}
// Conceptually compiled to:
void foo() {
    if ((guard_for_bar & 0xff) == 0) {
        if (__cxa_guard_acquire(&guard_for_bar)) {
            try {
                Bar::Bar(&bar);
            } catch (...) {
                __cxa_guard_abort(&guard_for_bar);
                throw;
            }
            __cxa_guard_release(&guard_for_bar);
            __cxa_atexit(Bar::~Bar, &bar, &__dso_handle);
        }
    }
}
```

---

## Local Static Variables in C++

**Concept**: A local static variable lives in the static storage area. Uninitialized ones are automatically set to zero. Its scope remains local — it becomes inaccessible once the enclosing function or block exits — but its value persists in memory until the program ends (you just can't access it from outside).

**Construction and destruction**: A local static variable is constructed the first time execution reaches its definition. Destruction order is the reverse of construction order. Since local statics can be scattered throughout the code and depend on execution paths, their construction order can be non-obvious — this is a common source of subtle bugs. **Use static local variables sparingly.**

**Returning a local static variable from a function**: With g++, returning the value is fine (the returned copy is valid).

---

## `const` and Pointers

Only two cases matter:

- **Pointer to const** (`const int *p` or `int const *p`): you cannot change the value pointed to via the pointer (i.e., `*p` is read-only), but you can reassign the pointer itself to a different address.
- **Const pointer** (`int * const p`): the pointer's address is fixed for its lifetime (you cannot do `p = &a`), but the value at that address can still be changed (`*p = 5` is valid).

```cpp
const int p;       // p itself is a constant integer
const int *p;      // *p is constant (pointer to const)
int const *p;      // same as above
int * const p;     // const pointer (the pointer itself is constant)
```

---

## Differences Between Pointers and References

**Underlying nature of a reference**

From a high-level perspective, a reference is an alias for a variable — it cannot exist independently of the object it refers to. At the assembly level, a reference is implemented as a **constant pointer**: it stores the address of the referenced object and cannot be retargeted.

**Comparison of reference vs. const pointer (high-level)**

1. Both occupy 4 or 8 bytes of storage and must be initialized at definition.
2. A const pointer (`p`) can be dereferenced (`&p` returns its own address; `*p` accesses the pointed-to object). A reference (`r`) does not permit its own address to be taken — `&r` returns the address of the *referenced object*, not of `r` itself.
3. A reference cannot be null; a pointer can.
4. Arrays can have pointer elements but not reference elements, to avoid ambiguity.
5. **Pointer passing** is value passing — it passes a copy of the address. Changes to the pointer itself inside the function do not affect the caller's pointer. **Reference passing** stores the caller's actual variable address; all operations on the parameter reach directly back to the caller's variable.
6. `sizeof(reference)` yields the size of the *referenced object*; `sizeof(pointer)` yields the size of the pointer itself.

---

## `const` vs. `#define` for Constants

**Type safety and checking**
- `#define` is textual substitution with no type — no type checking by the compiler, and it can produce unexpected side effects.
- `const` declares a typed constant, checked by the compiler at compile time.

**Processing phase**
- `#define` is handled by the preprocessor (before compilation); it cannot be debugged; its lifetime ends at compilation.
- `const` is a runtime concept; the value lives in the data segment and can be debugged.

**Storage**
- `#define` involves direct substitution with no memory allocation; it lives in the code segment.
- `const` requires memory allocation; it lives in the data segment.

---

## `typedef` vs. `#define`

**`typedef`**: Gives an existing type a new alias (not a new type). Main uses: (1) simplifying complex type declarations, (2) defining platform-independent type aliases that can be updated in one place for cross-platform portability.

**`#define`**: Macro definition — pure textual substitution, processed by the preprocessor.

**Key differences**:
1. `typedef` operates at compile time and includes type checking; `#define` is preprocessor-level with no checking.
2. `#define` has no scope; it applies everywhere after its definition. `typedef` respects scope.
3. Behavior with pointers differs:

```cpp
typedef int * pint;
#define PINT int *

const pint p1 = &i1;   // p1 is: int * const p (pointer itself is const)
const PINT p2 = &i2;   // p2 is: const int * p (pointed-to value is const)
```

---

## `#include <>` vs. `#include ""`

- `#include <>`: searches the compiler's configured include paths (for standard library headers).
- `#include ""`: searches the directory of the current source file first, then falls back to the standard include paths.

> If you write your own header file, always use `#include ""`. Your file won't be in the compiler's standard include path, so `#include <>` would fail to find it.
>
> If `#include ""` successfully finds a file, it shadows any same-named file that `#include <>` would have found.

---

## Lvalues and Rvalues

### History
In C, expressions were classified as *lvalues* (locator values — expressions that identify an object with a memory address) and everything else. C++ inherited this, but called non-lvalue expressions *rvalues*, and added the rule that references can bind to lvalues but only `const` references can bind to rvalues.

Since C++11, value categories were refined into five types: **lvalue**, **xvalue** (expiring value), **prvalue** (pure rvalue), **glvalue** (generalized lvalue), and **rvalue**. For most purposes, focus on lvalue, prvalue, and xvalue.

### Lvalues

An lvalue (locator value) is a data item stored in memory with a well-defined address. It is an expression that persists beyond the current statement.

Characteristics:
1. Its address can be obtained via `&`
2. Modifiable lvalues can appear on the left side of an assignment
3. Can be used to initialize an lvalue reference

What qualifies as an lvalue:
1. Variable names, function names, data member names
2. Function calls that return an lvalue reference
3. Assignment and compound-assignment expressions (e.g., `a=b`, `a-=b`)
4. Dereference expressions: `*ptr`
5. **Pre-increment and pre-decrement**: `++a`, `--b`
6. Member access via dot operator
7. Member access via arrow operator (`->`)
8. Subscript operator results (`[]`)
9. String literals (`"abc"`)

### Rvalues

An rvalue (read value) provides a data value but is not necessarily addressable (e.g., data stored in a register). In general, rvalues are temporary — they don't outlive the expression that creates them.

What qualifies as an rvalue:
1. Numeric/character/boolean literals (except string literals): `1`, `'a'`, `true`
2. Non-reference-returning function calls: `str.substr(1,2)`, `str1+str2`, `it++`
3. **Post-increment and post-decrement**: `a++`, `a--`
4. Arithmetic, logical, and comparison expressions
5. Address-of expression
6. Lambda expressions: `auto f = []{return 5;};`

> A rule of thumb: values whose lifetime is controlled by the compiler (you can only rely on them within a single line) are rvalues. Values whose lifetime you can reason about from scope rules are lvalues.

Rvalues are further subdivided: *prvalues* include non-reference temporary results, literals, and lambdas; *xvalues* (expiring values) are related to rvalue references (e.g., `std::move` return values, `T&&` casts).

---

## Lvalue References and Rvalue References

### Lvalue References

```cpp
// lvalue reference
int a = 10;
int &b = a;
b = 20;

// const lvalue reference (binds to const objects)
const int temp = 10;
const int &var = temp;
```

### Rvalue References

Introduced in C++11 to solve two problems: (1) unnecessary expensive copies of temporary objects, and (2) perfect forwarding of arguments in template functions.

**Line 1 — what is an rvalue?**
```cpp
int i = getVar();
```
`getVar()` produces a temporary value (rvalue) that is destroyed after the statement ends.

**Line 2 — extending the lifetime of a temporary via rvalue reference**
```cpp
T&& k = getVar();
```
`k` is an rvalue reference. The temporary returned by `getVar()` is now "kept alive" and shares `k`'s lifetime.

**Line 3 — move constructor**
```cpp
T(T&& a) {
    m_val = a.m_val;
    a.m_val = nullptr;
}
```
For classes managing heap memory, a default (shallow) copy constructor causes "dangling pointer" problems because two objects end up owning the same memory. A deep copy is safe but expensive. The move constructor solves this by performing a shallow copy *and* nulling out the source's pointer — the resource is *transferred*, not duplicated.

**Move semantics (`std::move`)**: allows explicitly moving an lvalue into an rvalue reference, enabling the move constructor. `std::move` itself just performs a cast — it moves nothing. For built-in types without a move constructor, it still copies.

**Line 4 — perfect forwarding (`std::forward`)**
```cpp
template <typename T>
void f(T&& val) {
    foo(std::forward<T>(val));
}
```
Without `std::forward`, rvalue arguments passed into a template function become lvalues inside the function. `std::forward` preserves the value category of the original argument, forwarding rvalues as rvalues and lvalues as lvalues.

---

## Deep Copy vs. Shallow Copy

When no copy constructor is explicitly defined, the compiler generates a default one — this performs a **shallow copy**, copying each member value one-to-one.

When data members contain **no pointers**, shallow copy is fine.

**When data members contain pointers, shallow copy causes problems.** Two objects end up pointing to the same heap memory. When either object is destroyed, its destructor `delete`s that memory. When the second object is destroyed, it tries to `delete` already-freed memory — a **dangling pointer** / double-free error.

**Deep copy** allocates a new block of heap memory in the copy constructor and copies the contents there. Simple rule: **whenever a class has pointer members, implement deep copy**.

---

## Compiler Optimization Levels: O0, O1, O2, O3

- **-O0**: No optimization (default). Simplest to debug.
- **-O1**: Basic optimizations — branch, constant, and expression optimization. Reduces code size and execution time. Includes loop optimization (moving constant expressions out of loops, simplifying loop conditions).
- **-O2**: Adds register-level and instruction-level optimizations on top of O1. Uses more compile-time memory and time. Generally safe and recommended for production.
- **-O3**: Further optimizations beyond O2, including pseudo-register networks, function inlining, and more aggressive loop transformations. Not always recommended — can increase compile failures or introduce unexpected behavior.

---

## The Four Variable Storage Classes in C++

1. **auto** (automatic storage): the default for local variables declared inside a function. Lives on the stack; created on function entry, destroyed on exit. Default value is undefined.
2. **static** (static storage): see [static section](#static-and-const--usage-and-interaction) above.
3. **extern** (external storage): declares that a variable is defined in another translation unit (another `.cpp` file). Used for sharing global variables across files.
   ```cpp
   extern int a;  // a is defined elsewhere
   ```
4. **register** (register storage): hints to the compiler to store the variable in a CPU register for speed. The address operator `&` cannot be applied to a register variable since it might not reside in RAM.

---

## C++ `this` Pointer

### Where does `this` come from?

Historically, early C++ compilers translated C++ to C before compiling. Classes became structs, but member functions became global functions. To allow member functions to access their object's data, the object's address was passed as an implicit parameter — this became `this`.

`this` is a hidden pointer present in every non-static member function, pointing to the object on which the function is called.

1. `this` points to the object that invoked the non-static member function.
2. When a member function is called, the compiler assigns the object's address to `this`; all member data accesses implicitly use `this`.
3. `this` is an rvalue — its own address cannot be taken (`&this` is illegal).

### When is `this` created?

`this` is constructed when the member function begins executing and destroyed when it returns.

### Where is `this` stored?

It depends on the compiler — typically in a register (e.g., `ecx` on x86), but it can also be on the stack or elsewhere.

### Can you call `delete this` inside a member function?

You can, but after `delete this`, any operation that touches `this` (accessing data members, calling virtual functions) produces undefined behavior. The object's memory has been freed.

### What happens if `delete this` is called inside a destructor?

Stack overflow. `delete` calls the destructor, which calls `delete this`, which calls the destructor again — infinite recursion, leading to a crash.

---

## `inline` Functions vs. Macros

C macros are efficient because they avoid function call overhead (no argument push, no return). `inline` functions achieve the same performance benefit but add:

1. The inline function body is expanded at the call site — no function call overhead.
2. Unlike macros, inline functions have proper type checking and behave like real functions.
3. Functions defined inside a class body are implicitly `inline` (except virtual functions).
4. The compiler allocates stack space for local variables in inline functions.

**Advantages over macros**:
- Same performance (code expansion at call site)
- Proper type safety and automatic type conversion
- Debuggable at runtime (macros are not)

**Disadvantages**:
- Code bloat — the function body is duplicated at every call site.
- If the inline function definition changes, all translation units that use it must be recompiled.
- Recursive inline functions expand indefinitely — very bad.

**Can virtual functions be inline?**

Inline is a compiler hint; the decision is the compiler's. Virtual functions exhibit polymorphism at runtime, not compile time. **When used polymorphically (called via a pointer/reference), virtual functions cannot be inlined.** They can be inlined only when the compiler knows the exact object type at compile time (i.e., called on a concrete object, not through a pointer or reference).

> `inline` is a suggestion, not a command. The compiler may ignore it (e.g., for a 100-line function). Inline functions must be defined in header files so the compiler can see the definition at every call site.

---

## `struct` and `typedef struct`

```c
// C style
typedef struct test3 {
    int a, b, c;
} test4;
// test3 is a tag; test4 is a type alias. Use: test4 t;  (equiv. to struct test3 t)

// C++ style
struct test3 {
    int a, b, c;
} test4;
// In C++, test3 is a type name directly. test4 here is a variable, not a type alias.
```

In C++, `struct` names are directly usable as type names without the `struct` keyword. The `typedef struct` pattern is mainly a C idiom.

---

## `explicit` Keyword

`explicit` is used to modify single-parameter constructors (or constructors where all parameters after the first have defaults). Such constructors also act as implicit type-conversion operators, which can cause accidental conversions.

**`explicit` prevents implicit conversions.**

```cpp
class Test {
public:
    explicit Test(int i);
};

Test t1 = 1;    // ERROR: implicit conversion blocked by explicit
Test t2(1);     // OK: direct initialization
```

**Implicit type conversion in C++**: the compiler automatically converts one type to another. Examples include `int` to `double` in arithmetic, and derived-to-base conversions in inheritance. Single-argument constructors create another conversion path — `explicit` blocks this.

---

## Friend Classes and Friend Functions

After encapsulation, private members are hidden. Sometimes external functions need to access them without going through the full member-function interface (e.g., for performance). Friend functions are declared inside a class and granted direct access to its private members.

- **Friend function**: a non-member function granted access to a class's private/protected members. Declared with `friend` inside the class.
- **Friend class**: all member functions of the friend class can access the target class's private/protected members.

**Trade-off**: Friends improve efficiency (avoiding repeated member function calls) but break encapsulation. Use them sparingly.

---

## C++ Virtual Functions

### Background

Dynamic dispatch (runtime polymorphism) requires calling a function based on the *actual type* of an object, not the declared type of a pointer. C++ allows a base class pointer to point to a derived class object:

```cpp
BrassPlus dilly;
Brass *pb = &dilly;   // upcasting — safe, implicit in C++
```

Assigning a derived-class object's address to a base-class pointer is called **upcasting** — C++ allows this implicitly. The reverse (**downcasting**) requires an explicit cast.

With a base class pointer pointing to a derived object, calling a `virtual` function dispatches to the *derived class's* version. Without `virtual`, the *base class version* is always called (determined by the pointer's static type). This mechanism allows a single `Brass*` array to call the correct `attack()` on `Wolf`, `Spider`, and `Python` objects.

### How Virtual Functions Work

The compiler implements virtual dispatch via:
- **vptr** (virtual table pointer): a hidden pointer added to the front of every object of a class with virtual functions.
- **vtbl** (virtual table): a per-class array of function pointers to virtual function implementations.

When a virtual function is called:
1. The object's `vptr` is followed to find the class's `vtbl`.
2. The appropriate function pointer is fetched from the `vtbl`.
3. The function is called through that pointer.

This costs: one extra pointer per object (for `vptr`), one extra table per class (for `vtbl`), and one extra indirection per virtual call.

### Virtual Function Tables Under Inheritance

- **Base class**: vtbl contains base virtual function addresses.
- **Single inheritance, no override**: child's vtbl copies base's vtbl, appends its own new virtual functions.
- **Single inheritance, with override**: the overridden slots are replaced with the child's function addresses.
- **Multiple inheritance**: the object has multiple `vptr`s (one per base class with virtual functions), each pointing to a separate vtbl.

### Performance Analysis

- Single inheritance: virtual call overhead is minimal (a pointer lookup and an indirect call — a few extra instructions).
- Multiple inheritance: offset calculations to find the correct `vptr` add some complexity.
- Space: each class generates a vtbl (a few function pointers). Each object gets one (or more) extra `vptr`(s). If many subclasses only override a few of many virtual functions, the vtbl entries for the unchanged functions are duplicated across every subclass — potentially wasting significant address space.

### Virtual Function FAQs

**Can constructors be virtual?**
No. Virtual dispatch requires a vtbl, and the vtbl pointer is set *during* construction. Before construction completes, the vtbl doesn't exist yet. Additionally, a virtual constructor makes no conceptual sense.

**Virtual destructors**
If a class is a base class, its destructor **must** be `virtual`. Otherwise, `delete`-ing a derived object through a base pointer only calls the base destructor, leaking derived-class resources.

**What cannot be virtual?**
- Friend functions (not class members)
- Static member functions (no `this` pointer → no vtbl access)
- Inline functions (must be expanded at compile time; virtual requires runtime dispatch)
- Constructors
- Member function templates (the vtbl size must be known at compile time; template instantiations are determined at call sites — incompatible)

**Is the vtbl shared or per-object?**
The vtbl is per-class, shared by all objects of that class. In GCC it is stored in the read-only data section (`.rodata`).

**Where are vtbl and vptr stored?**
The vtbl is compiled into the executable and loaded into read-only memory at startup — not on heap or stack. The vptr is embedded in the object, so it lives wherever the object lives (heap, stack, or global memory).

**How does the compiler build a vtbl for a derived class?**
1. Copy the base class's vtbl (or each base's vtbl in multiple inheritance).
2. Replace entries for overridden virtual functions with the derived class's addresses.
3. Append entries for any new virtual functions introduced by the derived class.

**Calling virtual functions in constructors/destructors**
Do not do this. In a constructor, the derived part of the object hasn't been built yet; the compiler uses the base class's vtbl, so you get the *base class* version of the virtual function — defeating the purpose. In a destructor, the derived part has already been destroyed, so again the base version is called.

---

## Implementing Polymorphism in C

C++ runtime polymorphism is fundamentally just callbacks via function pointers. C can replicate this:

**Compile-time polymorphism** (via variadic macros):
```c
#define Check(...) printf(__VA_ARGS__);
```

**Runtime polymorphism** (via function pointer structs):
```c
struct base_vtbl {
    void (*dance)(void *);
    void (*jump)(void *);
};

struct base {
    struct base_vtbl *vptr;
};
```
Each "class" has a vtbl-like struct of function pointers. "Inheritance" is achieved by embedding the base struct as the first member of the derived struct. "Polymorphism" is achieved by swapping out the function pointers in the vtbl to point to the derived implementations. Casting a `derived*` to `base*` is safe because `base` is at offset 0.

---

## Abstract Base Classes and Pure Virtual Functions

A **pure virtual function** declares an interface without providing an implementation:
```cpp
virtual void draw() = 0;
```
`= 0` does not mean the return value is 0. It means:
1. This function has no implementation in this class (it is abstract).
2. Any non-abstract derived class **must** override this function.

A class containing at least one pure virtual function is an **abstract class**. Abstract classes **cannot be instantiated** — they exist only to define interfaces.

---

## Creating Objects Only on the Heap or Only on the Stack

**Only on the heap** (no stack objects):
Make the destructor `private` (or `protected`). The compiler checks destructor accessibility before allocating a stack object. With a private destructor, stack allocation fails at compile time. Provide a public member function `destroy()` that calls `delete this`, since the destructor is inaccessible outside the class.

```cpp
class A {
protected:
    A() {}
    ~A() {}
public:
    static A* create() { return new A(); }
    void destroy() { delete this; }
};
```

**Only on the stack** (no heap objects):
Overload `operator new` as private to prevent heap allocation:
```cpp
class A {
private:
    void* operator new(size_t t) {}
    void operator delete(void* ptr) {}
public:
    A() {}
    ~A() {}
};
```
Note: the constructor cannot be made private (it's needed by both stack and heap allocation). Only `operator new` should be blocked.

---

## Preventing a Class from Being Instantiated

**Why?** Two common cases: abstract base classes and utility classes (e.g., a `Math` class with only static methods).

**Method 1: Pure virtual function**
```cpp
class Sharp {
public:
    virtual void get_s() = 0;
};
```

**Method 2: Private constructor**
```cpp
class Calculate {
private:
    Calculate();
public:
    static int add(int x, int y);
    // ...
};
```

---

## All Types of Constructors in C++

1. **Default constructor**: generated by the compiler if no constructors are defined; takes no parameters.
2. **Parameterized constructor**: accepts various arguments; multiple overloads are allowed if parameter types/counts differ.
3. **Copy constructor**: `ClassName(const ClassName& other)` — creates a new object as a copy of an existing one. Compiler-generated one does a shallow copy. If the class has pointer members, a deep copy copy constructor is needed.
4. **Move constructor**: `ClassName(ClassName&& other)` — transfers resource ownership from a temporary into a new object using shallow copy, then nulls the source pointer. Avoids expensive deep copies when the source is a temporary.
5. **Assignment operator**: `operator=` overload — assigns from an existing object. Not technically a constructor, but closely related. Both sides must already exist.
6. **Type-conversion constructor**: a single-argument constructor (or one where all extra arguments have defaults) that implicitly converts a value of the parameter type to the class type. Use `explicit` to prevent this.

---

## When is the Copy Constructor Called?

1. An object is passed **by value** into a function.
2. An object is returned **by value** from a function.
3. A new object is initialized from another existing object.

---

## Why Must the Copy Constructor Take a Reference Parameter?

**To prevent infinite recursion.** If the copy constructor took its argument by value, calling it would require making a copy of the argument — which would call the copy constructor again — infinitely.

If the parameter were a pointer instead of a reference, it would no longer be a copy constructor — it would just be an ordinary constructor taking a pointer.

---

## Can Constructors and Destructors Throw Exceptions?

**Constructors can throw exceptions.** An object is only fully constructed after its constructor completes. If an exception is thrown mid-constructor, the object is considered never to have been constructed — its destructor will not be called. Any resources already acquired in the constructor will be leaked unless managed by RAII (smart pointers). Note: if a derived class constructor throws, the base class's constructor and destructor execute normally.

**Destructors should NOT throw exceptions** (C++ standard effectively requires this):

1. If a destructor throws, code after the throw point in the destructor is skipped — potentially leaving resources un-freed.
2. If a destructor throws during stack unwinding (while an exception is already propagating), the program calls `std::terminate()`.

Solution: wrap the risky code in a try-catch inside the destructor and call `abort()` if it fails:
```cpp
DBConn::~DBConn() {
    try { db.close(); }
    catch (...) { abort(); }
}
```

---

## Constructor and Destructor Order in Inheritance

**Object creation order**: `static code → non-static code → constructor`. Static members (for the base class, then derived) are initialized first. For each class in the hierarchy, members are initialized in declaration order (not initializer-list order). The initializer list runs before the constructor body.

**Constructor call order**:
1. Base class constructor(s) — in order of appearance in the base class list (not initializer list order)
2. Member object constructors — in order of their declaration in the class
3. Derived class constructor

**Destructor call order** (reverse of construction):
1. Derived class destructor
2. Member object destructors
3. Base class destructor

---

## Member Variable Initialization Order

```cpp
class A {
private:
    int n1;
    int n2;
public:
    A() : n2(0), n1(n2 + 2) {}
    void Print() {
        cout << "n1:" << n1 << ", n2: " << n2 << endl;
    }
};
// Output: n1 is a garbage value (or 2 on some compilers); n2 is 0
```

**Key rules**:
1. When using an initializer list, initialization order matches the **declaration order of member variables**, not the order in the initializer list.
2. When initializing inside the constructor body, order matches the position within the constructor body.
3. Class members cannot be initialized at their declaration point (except `static const` integrals and C++11 in-class initializers).
4. `const` member variables must be initialized in the initializer list.
5. `static` member variables must be initialized outside the class definition.

**Variable initialization order (full hierarchy)**:
1. Base class static members (in declaration order)
2. Derived class static members (in declaration order)
3. Base class non-static members and constructor body
4. Derived class non-static members and constructor body

---

## C++ Access Inheritance (`public`, `protected`, `private`)

C++ classes default to **private** inheritance; structs default to **public** inheritance.

### What Cannot Be Inherited?

1. **Constructors**: A derived class must call the base constructor explicitly (via the initializer list). Constructors are not inherited because their name must match the class name, and base constructors can't be renamed to the derived class name.
2. **Destructors**: Cannot be inherited; the destruction sequence (derived first, then base) is enforced by the language.
3. **Assignment operator (`=`)**: It is overridden (shadowed) by the compiler-generated one in the derived class, not truly inherited. The base's signature (`Base& operator=(const Base&)`) doesn't match the derived's auto-generated one.
4. **`final` keyword** (C++11): prevents a class from being further derived.
5. **Private constructor/destructor**: prevents instantiation and inheritance.
6. **Friend + virtual inheritance trick**: by making CFinalClass's constructor private and making Cparent a friend, any class trying to derive from Cparent must call CFinalClass's constructor (due to virtual inheritance), but it can't (it's not a friend), causing a compile error.

---

## Member Initializer List

**When the initializer list is mandatory**:
1. `const` member variables (cannot be assigned, only initialized)
2. Reference member variables (cannot be reseated after initialization)
3. Member objects or base classes with no default constructor — the constructor must be explicitly called in the initializer list

**Why is the initializer list faster?**

For class-type members, using the initializer list calls the copy constructor directly. Using assignment in the constructor body first default-constructs the member (one constructor call), then assigns to it (one more call). The initializer list eliminates the redundant default construction.

---

## `new`/`delete` vs. `malloc`/`free`

**Similarities**: Both allocate and release dynamic memory.

**Differences**:

| | `malloc`/`free` | `new`/`delete` |
|---|---|---|
| What they are | Library functions | Operators |
| Return type | `void*` (requires cast) | Typed pointer (type-safe) |
| On failure | Returns `nullptr` | Throws `std::bad_alloc` |
| Size specification | Must provide size in bytes | Determined automatically by type |
| Constructor/destructor | Not called | Called automatically |
| Memory resizing | `realloc` available | No equivalent |

**`new` internals**:
1. Call `operator new` to allocate raw memory
2. Call the constructor to initialize the object
3. Return a typed pointer

**Why keep `malloc`/`free` if `new`/`delete` exist?**
C++ programs often call C functions, which only use `malloc`/`free`. They must coexist. Never mix: `new` with `free` or `malloc` with `delete`.

---

## `new`/`delete` vs. `new[]`/`delete[]` — Must They Match?

Yes — always use matching pairs.

- For custom types, `delete[]` calls the destructor exactly `N` times (where `N` is the array length). It knows `N` because `new[]` secretly stores the array size in the 4 bytes just before the returned pointer.
- Using `delete` on a `new[]` allocation: only one destructor call instead of `N`, and the pointer passed to `free` is 4 bytes off (it misses the hidden size field) — both are wrong.
- Using `delete[]` on a `new` allocation: it reads the 4 bytes before the allocation as if they were an array size — undefined behavior.

---

## Understanding `free`

`free` knows how much memory to release because `malloc`'s implementation (e.g., ptmalloc) stores the allocation size just before the returned pointer. `free` reads that metadata.

After `free(p)`, `p` still holds the old address. Accessing memory through `p` is undefined behavior (the memory may have been reallocated). Always set `p = nullptr` after freeing.

**Is freed memory immediately returned to the OS?**
Not necessarily. ptmalloc caches freed blocks in a double-linked list for potential reuse, reducing system call overhead. Small free blocks may be merged to avoid fragmentation. The memory is only returned to the OS under certain conditions (e.g., via `brk` or `munmap`).

---

## Stack Memory Allocation: `alloca`

`alloca` allocates memory on the stack. It does not need to be freed manually — the memory is reclaimed automatically when the enclosing scope exits.

`alloca` is not recommended for general use: allocating a large struct may silently overflow the stack without any error, corrupting adjacent memory.

Suitable for small, short-lived buffers (e.g., log messages) where the size is known to be safe.

---

## What Happens if a Non-void Function Omits `return`?

It is **undefined behavior** — but not a compilation error (typically just a warning). In practice, some garbage value from a register is returned. `main` is special: the compiler automatically appends `return 0` if omitted.

---

## The Four C++ Casts

### Upcasting and Downcasting

A class object's memory contains its own data followed (or preceded by vptr) by inherited base data.

- **Upcasting**: converting a derived pointer/reference to a base type — always safe and implicit. The base pointer can access only the base portion, which is always present.
- **Downcasting**: converting a base pointer/reference to a derived type — unsafe without a runtime check, because the base pointer might not actually point to a derived object.

### `static_cast`
- Basic type conversions
- `void*` ↔ other pointer types
- Upcasting (derived → base)
- Use to make implicit conversions explicit. Best practice: replace all C-style casts with appropriate C++ casts.

### `dynamic_cast`
- Only for pointers and references to objects in an inheritance hierarchy
- Used for **safe downcasting** — the only cast processed at runtime
- Returns `nullptr` (for pointers) or throws `std::bad_cast` (for references) if the cast fails
- Requires RTTI to be enabled

### `const_cast`
- Removes `const` or `volatile` qualifiers from a pointer or reference
- The target must be a pointer or reference (not an object directly)
- The underlying object must not actually be `const` — undefined behavior to write through the cast if it is

### `reinterpret_cast`
- Reinterprets the binary representation of any pointer type as any other pointer type
- No actual conversion — just a type-system lie
- Low-level: pointer ↔ integer, function pointer reinterpretation, etc.
- Very unsafe; use only when absolutely necessary

### Why Four Casts?

C's single cast operator allowed any conversion with no safety guarantees. The four C++ casts:
1. Make intent explicit (easy to search for in code reviews)
2. Provide appropriate safety levels per use case
3. Prevent accidental conversions (e.g., accidentally casting away `const`)

### When to Avoid `dynamic_cast`

Not that it can never be used — but problems solvable at compile time (via virtual functions or templates) shouldn't be punted to runtime with `dynamic_cast`. Over-reliance on `dynamic_cast` often signals a design smell.

---

## RAII (Resource Acquisition Is Initialization)

**Concept**: RAII is a C++ idiom by which resource acquisition happens in a constructor and release happens in a destructor. Since C++ guarantees constructor/destructor calls even during stack unwinding (exception propagation), RAII ensures resources are always properly released.

**Problem it solves**: Managing resources (memory, file handles, sockets, locks) requires: acquire → use → release. In complex control flows (many early returns, exception paths), the release step is easily forgotten or bypassed. RAII eliminates this by binding the resource's lifetime to an object's lifetime.

**Core insight**: wrap the resource in a class. The constructor acquires; the destructor releases. The C++ language guarantees the destructor runs when the object goes out of scope — even in the presence of exceptions (stack unwinding).

> **RAII is one of C++'s most distinctive features** compared to other languages. The practical implication: avoid raw `delete`; use smart pointers instead. Every resource acquisition should happen in a single statement, and management should immediately be handed to an RAII object.

---

## C++11 `enum class` vs. `enum`

**Problems with old-style `enum`**:
1. Enumerators pollute the enclosing namespace (accessible without scope resolution).
2. Implicit conversion to integer types — assigning between unrelated enums compiles without error.
3. Underlying type is implementation-defined — unknown size.

**`enum class` / `enum struct` (C++11)**:
1. No implicit conversion to integers (requires `static_cast`)
2. Underlying type is explicit (defaults to `int`; can be specified: `enum class Color : uint8_t { ... }`)
3. Enumerators require the scope operator: `Color::Red`

---

## Memory Leaks

**Definition**: heap memory allocated by the programmer (via `new`, `malloc`) that is never freed. Over time this reduces available memory and can crash the application.

**Causes**:
1. `new`/`malloc` without matching `delete`/`free`
2. `new[]` without `delete[]`
3. Base class destructor not declared `virtual` — when a base pointer points to a derived object, `delete` only calls the base destructor

**Detection/prevention**:
1. Good design: encapsulate all resource management in constructors/destructors (RAII)
2. Smart pointers
3. **Valgrind**: reports which line of code caused the leak
4. On Linux: `swap` command to monitor shrinking free memory over time
5. Tools like `netstat`, `vmstat` to spot gradual memory growth

---

## C++11 New Features

### `auto` and `decltype`

Both enable automatic type deduction, but differ in usage.

**`auto`**: deduces the type of a variable from its initializer. Resolves at compile time (no runtime overhead). Cannot be used without an initializer, cannot deduce function parameter types or array types.

```cpp
auto x = container.begin();  // deduces iterator type
auto& element = *pos++;       // deduces reference type
```

`auto` ignores top-level `const` and deduces the base type from references (not the reference itself).

**`decltype`**: deduces the type of an expression *without evaluating it*. Useful when you need the type but not the value.

```cpp
int a = 8, b = 3;
auto c = a + b;          // executes a+b at runtime to initialize c
decltype(a + b) d;       // compile-time type deduction; d is uninitialized int
```

`decltype` preserves top-level `const` and deduces reference types (unlike `auto`).

**Why keep `decltype` when `auto` exists?**

`auto` requires an initializer expression, which may have side effects. `decltype` deduces from an expression without executing it — essential for type computations in templates and trailing return types.

---

### Smart Pointers

Smart pointers are RAII wrappers around raw pointers. They automatically `delete` the managed object at the appropriate time.

| Pointer | Standard | Description |
|---|---|---|
| `unique_ptr` | C++11 | Exclusive ownership |
| `shared_ptr` | C++11 | Shared ownership via reference counting |
| `weak_ptr` | C++11 | Non-owning weak reference to a `shared_ptr`-managed object |
| `auto_ptr` | Removed in C++17 | Deprecated — use `unique_ptr` instead |

**Why smart pointers?**
1. Prevent memory leaks — no manual `delete` needed
2. Safe object lifetime management in multi-threaded contexts

#### `auto_ptr` (deprecated)

Problems: cannot be used with arrays, cannot be used with STL containers, and ownership transfer on copy leads to dangling pointers. Avoid entirely.

#### `shared_ptr`

Uses a reference count. Each copy increments the count; each destruction decrements it. When the count reaches zero, the managed object is `delete`d.

**Thread safety**:
- The reference count itself is atomic — thread-safe.
- The pointer stored inside a `shared_ptr` is **not** thread-safe for concurrent writes (multiple threads modifying the same `shared_ptr` object). Solution: add a mutex.
- The *managed object* has its own thread-safety properties independent of `shared_ptr`.

**Key methods**:
- `reset()`: releases ownership (decrements count); optionally takes a new pointer to manage
- `make_shared<T>(args)`: preferred factory — performs one heap allocation instead of two (avoids separate allocation for the control block and the object). Also exception-safe.
- `swap()`: exchanges ownership with another `shared_ptr`
- `shared_from_this()`: use when a class needs to produce a `shared_ptr` to itself from within a member function. Requires inheriting from `std::enable_shared_from_this<T>`.

#### `weak_ptr`

`weak_ptr` does not affect the reference count. Its primary purpose is breaking **circular references** that would prevent `shared_ptr` from ever reaching zero.

```cpp
// Circular reference — memory leak:
struct A { shared_ptr<B> b; };
struct B { shared_ptr<A> a; };

// Fixed with weak_ptr:
struct A { shared_ptr<B> b; };
struct B { weak_ptr<A> a; };
```

Key methods:
- `expired()`: returns `true` if the managed object has been destroyed
- `use_count()`: returns the current reference count
- `lock()`: returns a `shared_ptr` — non-null if the object still exists, null otherwise
- `reset()`: clears the weak_ptr

> Common usage: parent holds a `shared_ptr` to child; child holds a `weak_ptr` back to parent. Or: use `shared_ptr` when defining an object, `weak_ptr` everywhere else that references it.

#### `unique_ptr`

Exclusive ownership — no two `unique_ptr`s may own the same object.

```cpp
unique_ptr<A> a1(new A());
unique_ptr<A> a2 = a1;              // ERROR: copying not allowed
unique_ptr<A> a3 = std::move(a1);  // OK: ownership transferred; a1 is now null
```

Key methods:
- `get()`: returns the raw pointer (use sparingly)
- `bool()`: checks if it holds a pointer
- `release()`: relinquishes ownership; returns the raw pointer without deleting it
- `reset()`: releases and destroys the old pointer; optionally takes a new one

---

### Lambda Expressions

```cpp
[capture](params) opt -> ret { body; };
```

**Capture list `[capture]`**:
- `[]` — capture nothing
- `[&]` — capture all by reference
- `[=]` — capture all by value
- `[=, &foo]` — capture all by value, except `foo` by reference
- `[this]` — capture the current object's `this` pointer

**`mutable`**: allows modification of value-captured variables inside the lambda (the copy is modified, not the original).

**Return type**: often omitted — deduced automatically.

---

### `nullptr` vs. `NULL`

`NULL` is defined as `0` in C++ — it is an integer, not a pointer. This causes ambiguity:

```cpp
void f(void*) {}
void f(int) {}
f(NULL);  // calls f(int) — not what you want!
```

`nullptr` has type `std::nullptr_t`, which is implicitly convertible to any pointer type but not to integer types. It always resolves to the pointer overload:

```cpp
f(nullptr);  // calls f(void*) — correct!
```

---

### Member Initializer List

**When mandatory**:
1. `const` member variables
2. Reference member variables
3. Member objects or base classes without a default constructor

**Properties**:
1. Initialization order matches **declaration order** in the class, not initializer-list order
2. Initializer list executes before the constructor body
3. Faster for class-type members: directly calls the copy constructor instead of default-constructing + assigning

---

### Rvalue References

(See [Lvalue References and Rvalue References](#lvalue-references-and-rvalue-references) above.)

---

### Range-based `for` Loop

```cpp
for (auto& element : container) { /* ... */ }
```

---

### Variadic Templates

```cpp
template <class... T>
void f(T... args);
```

The `...` declares a *parameter pack*, which can contain 0 or more template arguments. The pack can be expanded later.

---

### `std::atomic` — Atomic Operations

Atomic operations guarantee that a read-modify-write on a shared variable is completed without interruption from another thread. C++11 provides `std::atomic<T>` (e.g., `atomic<int>`, `atomic<bool>`). Atomic operations are closer to hardware than mutexes, so they are more efficient for simple counters and flags.

---

### `std::function` and `std::bind`

**Callable objects in C++**: functions, function pointers, lambda expressions, function objects (classes with `operator()` overloaded), and `bind` objects.

**`std::function`**: a type-erased wrapper for any callable with a given signature.

```cpp
typedef std::function<int(int, int)> comfun;
int add(int a, int b) { return a + b; }
auto mod = [](int a, int b) { return a % b; };
struct divide { int operator()(int a, int b) { return a / b; } };

comfun a = add;
comfun b = mod;
comfun c = divide();
```

**`std::bind`**: creates a new callable by binding arguments to a function.

```cpp
auto f1 = std::bind(fun, 1, 2, 3);              // all args fixed
auto f2 = std::bind(fun, _1, _2, 3);            // first two args from caller
auto f3 = std::bind(fun, _2, _1, 3);            // args swapped
f2(1, 2);   // calls fun(1, 2, 3)
f3(1, 2);   // calls fun(2, 1, 3)
```

`_1`, `_2`, etc. are placeholders from `std::placeholders`.

---

### C++11 Keywords

**`noexcept`**: declares that a function will not throw. If it does at runtime, `std::terminate()` is called immediately.

```cpp
void swap(Type& x, Type& y) noexcept { x.swap(y); }
```

**`override`**: explicitly marks a virtual function as overriding a base class virtual function. Compile error if no matching virtual function exists in the base class.

**`final`**: applied to a class to prevent further inheritance, or to a virtual function to prevent further overriding.

**`= default`**: instructs the compiler to generate the default implementation for a special member function (constructor, destructor, copy/move operators). Useful when you want a default that the compiler wouldn't automatically generate (because another constructor was declared).

**`= delete`**: explicitly suppresses the generation of a function.

**`using`** (C++11 extensions):
1. Type alias (replaces `typedef`): `using MyInt = int;`
2. Bringing an overloaded base class function into scope in a derived class

---

## Implementing a Reference Counter

```cpp
template <class T>
class Ref_count {
private:
    T* ptr;
    int* count;
public:
    // Construct from raw pointer
    // WARNING: do NOT construct two Ref_count objects from the same raw pointer
    Ref_count(T* t) : ptr(t), count(new int(1)) {}

    ~Ref_count() { decrease(); }

    // Copy constructor
    Ref_count(const Ref_count<T>& tmp) {
        count = tmp.count;
        ptr = tmp.ptr;
        increase();
    }

    // Assignment operator: left side loses one reference, right side gains one
    Ref_count<T>& operator=(const Ref_count& tmp) {
        if (&tmp != this) {
            decrease();
            ptr = tmp.ptr;
            count = tmp.count;
            increase();
        }
        return *this;
    }

    T* operator->() const { return ptr; }
    T& operator*() const { return *ptr; }

    void increase() { if (count) (*count)++; }

    void decrease() {
        if (count) {
            (*count)--;
            if (*count == 0) {
                delete ptr; ptr = nullptr;
                delete count; count = nullptr;
            }
        }
    }

    T* get() const { return ptr; }
    int get_count() const { return count ? *count : 0; }
};
```

> **Never** pass the same raw pointer to multiple `Ref_count` objects — each will maintain its own independent count, causing a double-free.

---

## Why is Hash Table Lookup O(1)?

A hash table's physical storage is an **array**. Arrays provide O(1) random access given an index.

The hash function maps a key to an array index. Given the index, finding the storage location is a simple array lookup — O(1).

When collisions occur (two keys hash to the same index), a linked list is stored at that index. The array element holds a pointer to the head of the list. Lookup traverses the list to find the matching key. With a good hash function and a properly-sized table, collisions are rare and chains are short — average lookup stays O(1).

---

## `sizeof` Questions

`sizeof` is a compile-time operator returning the number of bytes of its operand. It works on types, expressions, arrays, and objects (but not dynamically allocated memory).

**Empty class**: `sizeof` is 1. An instance must occupy at least 1 byte so that two different objects have different addresses. An empty class/struct returns 1.

**Class with only a constructor and destructor (no virtual)**: still 1. Function addresses are independent of object size.

**Class with a virtual function**: `sizeof` = size of a pointer (4 bytes on 32-bit, 8 bytes on 64-bit) — for the `vptr`.

**Computing `sizeof(int)` without `sizeof`**:
```cpp
int i = 1, count = 0;
while (i) { i <<= 1; count++; }
cout << count / 8 << endl;  // bits → bytes
```

---

## `volatile`

`volatile` tells the compiler: "do not optimize reads/writes to this variable — always access it from memory." Without `volatile`, the compiler may cache the value in a register and skip memory reads.

Common use cases: hardware registers, signal handlers, variables shared between the main program and an ISR.

**`volatile` does NOT make operations atomic or thread-safe.** It only prevents compiler optimizations. For multi-threaded synchronization, use `std::atomic<T>` or mutexes. In C/C++ multi-threaded code, do not rely on `volatile` for synchronization.

**Can `volatile` and `const` be combined?** Yes — a `const volatile` variable cannot be modified by the program but may be changed by external hardware.

---

## `extern` Keyword

**Use 1**: `extern "C"` — tells the compiler to use C linkage (no name mangling) for a block of code. Required when C++ code calls C functions, because C++ mangles function names (to support overloading) but C does not.

**Use 2**: `extern int a;` — declares that a variable is defined in another translation unit. Used to share global variables across `.cpp` files without re-defining them.

---

## How `auto` Type Deduction Works Internally

`auto` uses **template argument deduction**. The compiler treats `auto` as a virtual template parameter `T`, creates a conceptual function template call, and deduces `T` from the initializer — exactly as it would deduce a template parameter from a function argument.

```cpp
auto pos = container.begin();
// Equivalent to deducing T in:
template<typename T> void deducePos(T pos);
deducePos(container.begin());
```

---

## C++ Program Optimization Techniques

- Cache frequently-read resources in memory when space allows
- Avoid repeated construction/destruction of large objects — cache unused ones for reuse
- Use C++11 move semantics to eliminate temporary copies
- Prefer `inline` for simple short functions; prefer composition over deep inheritance hierarchies
- Optimize loop conditions to reduce iteration count
- Prefer atomic operations over mutexes when possible; prefer user-space synchronization over kernel objects
- Use memory pools for frequent small allocations/deallocations
- Use thread pools for frequently-created/destroyed threads
- Prefer event-driven notification over polling or `sleep`
- Put long-running operations on background threads to keep UI responsive
- Regularly refactor code; optimize at the architectural and algorithmic level

---

## `void*` (Generic Pointer)

A pointer carries two pieces of information: the address of the target object, and the type of that object (enabling dereferencing and pointer arithmetic). `void*` carries only the address — it has no type. Therefore:
- Cannot be dereferenced
- Cannot be used in pointer arithmetic
- Can be cast to any other pointer type (and vice versa)

Useful as a generic container for addresses when the type is unknown at compile time (e.g., `malloc` returns `void*`).

---

## RVO / NRVO (Return Value Optimization)

**RVO** (Return Value Optimization) and **NRVO** (Named RVO) are compiler optimizations that eliminate the construction of temporary objects when a function returns by value.

Without RVO, `return obj;` would: construct `obj`, copy-construct a temporary at the call site, destroy `obj`, and the caller would receive the temporary. With RVO, the compiler constructs the return value directly in the caller's storage — zero copies.

```cpp
// Conceptually transformed from:
MyClass get() { return MyClass(); }
// To:
void get(MyClass& result) { new(&result) MyClass(); }
```

RVO is standard in modern compilers and is guaranteed in many cases by C++17 (mandatory copy elision).

---

## From Source Code to Executable

```
Source (.cpp) → Preprocessor → Compiler → Assembler → Linker → Executable
```

1. **Preprocessing** (→ `.i`): expands macros, processes `#include`/`#ifdef`, removes comments.
2. **Compilation** (→ `.s`): lexical analysis, syntax analysis (parse tree), semantic analysis (type checking), code generation. Produces assembly code.
3. **Assembly** (→ `.o`): converts assembly to machine code (binary object file). Each assembly instruction becomes one machine instruction.
4. **Linking** (→ executable): combines object files and resolves references to library functions. Produces the final executable.

---

## Static Linking vs. Dynamic Linking

**Static linking**: library code is copied into the final executable at link time.
- Pro: self-contained, portable, no runtime dependency
- Con: large binary; library updates require recompilation; each process has its own copy of the library in memory — wasteful

**Dynamic linking**: the executable references external shared library files (`.so` / `.dll`) that are loaded at runtime.
- Pro: smaller binary; library updates don't require recompilation; multiple processes share one copy of the library in RAM
- Con: requires the shared library to be present at runtime; slightly slower startup

**Analogy**: static linking is like copying a reference book's pages into your notes (convenient but bulky). Dynamic linking is like noting "see page X of Book Y" (compact, but you need the book available when you study).

---

## Static vs. Dynamic Compilation

- **Static compilation**: links all required library code into the executable at compile time. Standalone; no external dependencies at runtime.
- **Dynamic compilation**: links against shared libraries at runtime. Smaller binary, faster compilation, easier library updates — but requires the shared library to be installed on the target system.

---

## Static vs. Dynamic Binding (Linkage)

- **Static binding** (early binding): the function to call is resolved at compile time. Based on the pointer/reference type. Used for non-virtual functions. Higher performance, less flexible.
- **Dynamic binding** (late binding): the function to call is resolved at runtime. Based on the actual object type. Used for virtual functions via the vtbl mechanism. More flexible, slight runtime overhead.

C++ defaults to static binding and switches to dynamic binding when virtual functions are involved.

---

## Uninitialized Pointers (Wild/Dangling Pointers)

**Wild pointer**: a pointer that has never been initialized. Compiles without error but accessing it is undefined behavior — the pointer holds whatever value happened to be in memory at that address.

**Dangling pointer**: a pointer to memory that has already been freed or gone out of scope.

Behavior: if the address happens to be in OS-reserved memory, a crash (segfault). If it is in user memory, the data there may be silently corrupted — harder to debug.

Always initialize pointers: `int* p = nullptr;`. `nullptr` is guaranteed not to point to any object, unlike an uninitialized pointer.

---

## What Happens if a Basic Variable is Uninitialized?

Nothing happens if you never use it. But if you do use it, behavior is undefined. For example, a `char buf[2000]` used as a string buffer without being zeroed might not have a null terminator, causing string functions to read past the buffer.

Best practice: always initialize variables in the constructor or at the declaration point.

---

## Function Parameter Push Order and Evaluation Order

**C/C++ pushes function arguments onto the stack right-to-left.** This ensures the first argument (leftmost) ends up at the top of the stack — closest to the callee's stack frame — which is essential for variadic functions like `printf` that must locate the format string (first argument) reliably.

**Parameter evaluation order** is unspecified in C++ (implementation-defined). Do not write code that relies on a specific left-to-right or right-to-left evaluation order of arguments.

**`printf` mechanics**: the format string (first argument, at stack top) is found first. The format string is parsed to determine how many additional arguments to expect and their types, allowing the function to access them via stack offsets.

---

## Template Metaprogramming

Generic programming is a style of programming that uses *algorithms and data structures written in terms of types to be specified later*. Templates allow one implementation to serve many types.

C++ templates come in two flavors: **function templates** and **class templates**.

### How Templates Work Internally

The compiler does **not** compile a function template into a single function that handles all types. Instead, it performs **template instantiation**: for each concrete set of template arguments used in the program, it generates a separate function/class. The template code itself is compiled twice: once for the template declaration, and once per instantiation.

This is why the template definition must be visible at the point of use — both declaration and definition typically go in the header file.

### Function Templates

```cpp
template <typename T>
T mySwap(T& a, T& b) { /* ... */ }

mySwap(a, b);        // implicit type deduction
mySwap<int>(a, b);   // explicit type specification
```

- Implicit deduction requires all `T` usages to resolve to the same concrete type
- Implicit deduction does **not** perform implicit type conversions; explicit specification does

**Ordinary function vs. function template call priority**:
1. Ordinary functions are preferred if they match exactly
2. Use empty template argument list `<>` to force a template call
3. If the template produces a better match, it is preferred

### Class Templates

```cpp
template <class T1, class T2>
class Person { T1 name; T2 age; };
```

Class templates have **no implicit type deduction** — you must always specify template arguments explicitly: `Person<string, int> p;`.

Member functions of class templates are instantiated only when called.

**Class template inheritance**:
```cpp
class Son : public Father<int> {}; // must specify T for the base
// For flexibility:
template <class T1, class T2>
class Son : public Father<T2> { T1 obj; };
```

### Why Can Member Function Templates Not Be Virtual?

The vtbl size must be determined at compile time. If a member function template were virtual, the compiler would need to scan all code to find all instantiations before knowing the vtbl size — not feasible with current compiler architecture.

### Can Template Definition and Declaration Be in Separate Files?

**No.** The template definition must be visible at the point of instantiation. Splitting declaration (`.h`) from definition (`.cpp`) means the compiler, when compiling a user's `.cpp`, only sees the declaration and cannot instantiate — leading to linker errors. The solution is to put everything in the header.

### Templates vs. Inheritance

| | Templates | Inheritance |
|---|---|---|
| Polymorphism | Compile-time (static) | Runtime (dynamic) |
| Relationship type | Code duplication for different types | Object relationship (is-a) |
| Flexibility | Very high — any type conforming to the interface | Constrained by the class hierarchy |
| Use case | Generic algorithms/data structures | Modeling domain relationships |

Templates = code reuse across types. Inheritance = modeling relationships between types.

---

## `mutable` Keyword

In a class, `mutable` marks a data member as modifiable even in a `const` member function or on a `const` object:

```cpp
struct Test {
    int a;
    mutable int b;
};
const Test test = {1, 2};
test.a = 10;  // ERROR
test.b = 20;  // OK — mutable
```

Also used in lambda expressions: by default, value-captured variables in a lambda are `const`. Adding `mutable` allows modification of the local copy (without affecting the original).

---

## RTTI (Run-Time Type Identification)

**Concept**: RTTI allows a program to determine the actual type of an object pointed to by a base class pointer at runtime. C++ provides RTTI through:

1. **`typeid` operator**: returns a `type_info` object describing the type of its expression. Useful for comparing actual types.
2. **`dynamic_cast`**: safely downcast a base pointer/reference to a derived type at runtime; returns `nullptr` or throws on failure.

RTTI is only applicable to classes with at least one virtual function (i.e., polymorphic classes). This is because the actual type information (`type_info`) is stored alongside the vtbl.

C++ is a statically-typed language — types are determined at compile time. RTTI is the mechanism for the rare cases when runtime type discovery is needed (e.g., when the static type is a base class pointer that could point to many derived types).

---

## Virtual Inheritance

**Why?** Diamond inheritance causes a derived class to contain two copies of the virtual base class's members, leading to ambiguity and wasted memory.

```
     A
    / \
   B   C
    \ /
     D
```

Without virtual inheritance: `D` has two copies of `A`'s members — accessing `A::x` through `D` is ambiguous.

With virtual inheritance (`B` and `C` virtually inherit `A`): `D` contains only one copy of `A`. The "most derived class" (`D`) is responsible for constructing `A` directly.

```cpp
class A {};
class B : public virtual A {};
class C : public virtual A {};
class D : public B, public C {};
```

**Cost**:
- Time: an extra level of indirection (via a virtual base pointer) to locate `A`'s members
- Space: an extra pointer per virtual base class in each derived object; but saves space by having only one copy of the base

**Key rule**: the most-derived class's constructor initializes the virtual base class directly — intermediate classes' calls to the virtual base constructor are ignored. This is necessary because B and C might try to initialize `A` differently, and only one instance of `A` exists.

**Usage**: avoid unless you truly need diamond inheritance. The C++ standard library's `iostream` hierarchy is the canonical example: `iostream` inherits from both `istream` and `ostream`, which both virtually inherit from `ios_base`.

---

## C++ Memory Issues

1. **Buffer overflow**: writing past the end of an allocated buffer. Use `vector`/`string` to avoid.
2. **Dangling pointer / wild pointer**: use `shared_ptr` and `weak_ptr`.
3. **Double free**: use `scoped_ptr`; ensure `delete` only happens in destructors.
4. **Memory leak**: use `scoped_ptr` or RAII wrappers; avoid raw `delete`.
5. **`new[]` / `delete` mismatch**: replace `new[]` with `vector`.
6. **Memory fragmentation**: advanced — requires careful allocator design or memory pooling.

---

## Compile-time Polymorphism vs. Runtime Polymorphism

**Compile-time (static) polymorphism**: function overloading and templates. The function called is determined at compile time. No runtime overhead.

**Runtime (dynamic) polymorphism**: virtual functions. The function called is determined at runtime via the vtbl. Requires a base class hierarchy and at least one virtual function.

**Trade-offs**:

| | Runtime polymorphism | Compile-time polymorphism |
|---|---|---|
| Flexibility | High — handles heterogeneous collections | Lower — types determined at compile time |
| Performance | Virtual call overhead, can't be inlined | No overhead; can be fully inlined |
| Code complexity | Clear inheritance hierarchy | Template code can be hard to read |
| Error detection | Runtime | Compile time |
| Compilation | Fast | Slow (template instantiation) |

---

## Pointer Passing vs. Reference Passing

**Pointer passing** is *value passing* of an address. The callee receives a copy of the pointer. Changing the pointer itself (making it point elsewhere) does not affect the caller's pointer. To change what the caller's pointer points to, you'd need a pointer-to-pointer.

**Reference passing**: the callee operates directly on the caller's variable. All operations go through an implicit pointer to the original object. The reference itself cannot be reseated.

From the compiler's perspective: both add an entry to the symbol table. The pointer's symbol table entry holds the pointer variable's own address; the reference's entry holds the address of the referenced object. This is why references can't be reseated (the symbol table is fixed after compilation), while pointers can.

---

## Function Pointers

A **function pointer** stores the entry-point address of a function. With a function pointer, you can:
- Call the function indirectly
- Pass functions as arguments (callbacks)
- Store functions in data structures

```cpp
int (*fp)(int, int);   // pointer to a function taking two ints, returning int
fp = &add;
int result = fp(3, 4);
```

---

## How to Execute Code Before `main`

```cpp
// Method 1: GCC attribute
__attribute__((constructor)) void before() { printf("before main\n"); }

// Method 2: static variable initialization
int test1() { cout << "before main" << endl; return 1; }
static int i = test1();

// Method 3: lambda
int a = []() { cout << "before main" << endl; return 0; }();
```

Similarly, `__attribute__((destructor))` runs code after `main` exits.

---

## Pre-increment (`++i`) vs. Post-increment (`i++`)

```cpp
// ++i (pre-increment): no temporary object
int& operator++() { *this += 1; return *this; }

// i++ (post-increment): creates a temporary copy
const int operator++(int) { int old = *this; ++(*this); return old; }
```

`++i` is more efficient for non-trivial types (e.g., iterators) because it avoids creating and destroying a temporary. For built-in integers the compiler usually optimizes them to the same thing.

---

## Variable Declaration vs. Definition

- **Declaration**: informs the compiler of a name and its type; no memory allocated.
- **Definition**: allocates memory; only one definition per variable (ODR — one definition rule).

A variable can be declared multiple times (via `extern`) but defined only once.

---

## C++ Exception Handling

**Method 1: `try` / `throw` / `catch`**
```cpp
try { /* code */ throw SomeException(); }
catch (SomeException& e) { /* handle */ }
catch (...) { /* catch-all */ }
```

**Method 2: Exception specification (pre-C++11)**
```cpp
int fun() throw(int, double, A);  // may throw int, double, or A
int fun() throw();                 // throws nothing
```

**Method 3: `std::exception` hierarchy**

Standard exceptions include `std::bad_alloc` (from `new`), `std::bad_cast` (from `dynamic_cast`), `std::runtime_error`, `std::logic_error`, `std::out_of_range`, and others. All derive from `std::exception`.

**Method 4: User-defined exception classes**

Derive from `std::exception` and override `what()`.

---

## Function Call Stack

When `main` calls `func()` which calls `f()`:
1. `main`'s arguments, return address, and local variables are pushed onto the stack.
2. When `func()` is called, the current state of `main`'s frame is saved; `func()`'s return address, arguments (right to left), and local variables are pushed.
3. When `f()` is called, same process for `func()`'s frame.
4. On return, frames are popped in reverse order.

---

## Core Dump

A core dump is a snapshot of a process's memory (registers, stack, heap, code) at the moment it crashed. Used with debuggers (e.g., `gdb program core`) to locate the crash: examine the call stack, variable values, and the faulting instruction.

---

## Comparing Floating-Point Numbers for Equality

**Never use `==` directly on floating-point numbers.** Due to floating-point representation errors, two values that should be mathematically equal may differ by a tiny epsilon.

Instead, compare with a tolerance:
```cpp
bool equal = (fabs(a - b) < 1e-9);
```
Choose the epsilon appropriate for the scale and precision of your computation.

---

---

# STL (Standard Template Library)

## Overview

STL is a C++ standard library providing generic, reusable components. It consists of six parts:

1. **Containers**: data structures (vector, list, map, set, ...)
2. **Iterators**: uniform interface for traversing containers
3. **Algorithms**: sorting, searching, transforming, ...
4. **Functors (function objects)**: objects callable like functions (overload `operator()`)
5. **Adapters**: wrappers that modify container/iterator behavior (stack, queue, priority_queue)
6. **Allocators**: memory management for containers

---

## Container Summary

**Sequence containers**: elements stored in a linear order you control.

| Container | Underlying structure | Notes |
|---|---|---|
| `array` | Fixed-size array | No add/delete; O(1) random access |
| `vector` | Dynamic array | O(1) amortized push_back; O(n) insert/delete in middle |
| `deque` | Segmented array | O(1) push/pop at both ends; O(1) random access |
| `list` | Doubly linked list | O(1) insert/delete anywhere; O(n) random access |
| `forward_list` | Singly linked list | Single-direction traversal |

**Associative containers** (ordered, implemented as red-black trees):

| Container | Ordered | Duplicates | Notes |
|---|---|---|---|
| `set` | Yes | No | O(log n) all ops |
| `multiset` | Yes | Yes | |
| `map` | Yes | No | key-value pairs |
| `multimap` | Yes | Yes | |

**Unordered containers** (hash table):

| Container | Notes |
|---|---|
| `unordered_map` | O(1) avg; no key order |
| `unordered_set` | O(1) avg; no element order |
| `unordered_multimap` | Allows duplicate keys |
| `unordered_multiset` | Allows duplicate values |

**Adapters** (not standalone — built on other containers):
- `stack` — LIFO, typically wraps `deque`
- `queue` — FIFO, typically wraps `deque`
- `priority_queue` — max-heap, typically wraps `vector`

**Supports random access**: `string`, `array`, `vector`, `deque`  
**Supports efficient insert/delete anywhere**: `list`, `forward_list`  
**Supports fast push-back**: `vector`, `string`, `deque`

---

## `vector`

### Description

A dynamic array. Maintains a contiguous block of memory. Internally tracks three pointers:
- `start`: beginning of allocated space
- `finish`: one past the last used element
- `end_of_storage`: one past the end of allocated space

Provides random-access iterators.

### Memory Growth Mechanism

When capacity is exhausted (e.g., `push_back`): allocates a new block (typically **2× the current capacity** in GCC, **1.5× in MSVC**), copies all elements, frees the old block.

Any reallocation **invalidates all iterators, pointers, and references** into the vector.

`clear()` removes elements but does not release the capacity.

### `reserve()` vs. `resize()`

- **`reserve(n)`**: ensures at least `n` capacity — allocates if needed but does **not** change `size` or construct elements. Use before a series of `push_back` operations to avoid repeated reallocations.
- **`resize(n)`**: changes `size` to `n`. If `n > size`, constructs new default elements. If `n < size`, destroys excess elements. Also adjusts `capacity` if needed.

`reserve` is for push_back optimization; `resize` is for direct indexed access.

### `size()` vs. `capacity()`

- `size()`: number of elements currently stored
- `capacity()`: maximum elements that can be stored without reallocation

### Can `vector` Element Type Be a Reference?

No. Container elements must be copyable and assignable. References are neither (they can't be default-constructed, and assigning to a reference modifies the referenced object, not the reference itself). Use pointers instead if indirection is needed.

### Iterator Invalidation

- **Reallocation** (due to `push_back`, `insert`, etc.): all iterators become invalid.
- **`erase`**: the erased iterator and all iterators after it become invalid. `erase` returns the next valid iterator.
- **`insert` at a position**: that position and all subsequent iterators become invalid.

Always reassign the iterator from the return value of `erase`:
```cpp
it = vec.erase(it);
```

### Is `vector` on the Heap or the Stack?

The *elements* of a vector are always on the heap — the vector's internal array is dynamically allocated regardless of where the vector object itself lives. A `vector<int> v` on the stack has its three control pointers on the stack, but the actual element data is on the heap.

### `emplace_back` vs. `push_back`

Both accept lvalues (copy) and rvalues (move). The difference:

- `push_back(MyClass(x, y))`: constructs a temporary `MyClass(x, y)`, then move-constructs it into the vector.
- `emplace_back(x, y)`: constructs `MyClass` directly in-place inside the vector from arguments `x` and `y` — **one fewer construction**.

Use `emplace_back` when constructing from arguments; use `push_back` when you already have an object.

### Releasing `vector` Memory

`clear()` sets `size` to 0 but does not free the internal array. To truly release memory:
```cpp
vector<int>().swap(vec);   // swap with a default-constructed empty vector
```

### `vector` Insert Complexity

- Push-back (amortized): O(1)
- Insert/delete at an arbitrary position: O(n) (elements must be shifted)
- Random access: O(1)

### Why Grow by a Multiplier Rather Than a Fixed Size?

With fixed-size growth, the amortized time complexity of `push_back` is O(n). With multiplicative growth (factor k), the amortized cost per push_back is O(1) — each element is copied O(log_k n) times across all reallocations.

### Why 1.5× or 2× (Not 3× or 4×)?

With 2× growth, each new allocation is larger than all previous allocations combined — freed memory can **never** be reused. With 1.5×, after several reallocations, the new allocation size falls within the range of previously freed memory — it can be reused, improving cache locality and reducing heap fragmentation. MSVC uses 1.5×; GCC uses 2×.

---

## `deque`

### Description

A double-ended queue supporting O(1) push/pop at both front and back, and O(1) random access. Unlike `vector`, it doesn't use a single contiguous block.

### Internal Structure

`deque` uses a **map array** (array of pointers) where each pointer points to a fixed-size **buffer** (segment). Elements are distributed across these segments. The map array tracks which segment comes first and last.

When space runs out at either end, a new segment is allocated and its pointer is added to the map. If the map array itself fills up, a new larger map array is allocated and the segment pointers are copied.

### Iterator Design

A `deque` iterator tracks: `cur` (current element), `first` (start of current segment), `last` (end of current segment), and `node` (pointer to the map entry for this segment). Incrementing the iterator checks if it crosses a segment boundary and adjusts accordingly.

### `deque` vs. `vector`

- `deque` supports efficient push/pop at both ends; `vector` only at the tail
- `deque` has no `capacity`/`reserve` concept
- `vector` is better for random access performance (single contiguous block, better cache locality)
- Choose `deque` when you need efficient insertion at both ends; choose `vector` otherwise

---

## `list`

### Description

A doubly linked list. Each `insert`/`erase` allocates/frees exactly one node. All existing iterators remain valid after insert/erase (except the erased iterator itself). Provides bidirectional iterators.

### `list` vs. `vector`

| | `vector` | `list` |
|---|---|---|
| Memory | Contiguous | Scattered nodes |
| Random access | O(1) | O(n) |
| Insert/delete at middle | O(n) (shift) | O(1) |
| Memory per element | Compact | + 2 pointers overhead |
| Iterator invalidation | On reallocation | Only erased element |

---

## `stack` and `queue`

`stack` (LIFO): wraps `deque` by sealing one end. Or can wrap `list`.  
`queue` (FIFO): wraps `deque` by sealing one end for input and the other for output.

These are *adapters*, not independent containers.

---

## Container Selection Guide

- Default choice for sequences: **`vector`**
- Default choice for key-value lookup: **`unordered_map`**
- Need fast insert/delete at both ends: **`deque`**
- Need fast insert/delete anywhere with stable iterators: **`list`**
- Need sorted unique keys: **`set`** / **`map`**

---

## Associative Containers (map, set, multimap, multiset)

### Why Are Insert/Delete Efficient?

These containers store nodes (not contiguous arrays). Insert/delete only reassigns a few pointers in the red-black tree — no memory copying or shifting.

### `set` vs. `multiset` Insertion

`set::insert` returns a `pair<iterator, bool>` — `false` if the key already exists, `true` if inserted. `multiset::insert` always succeeds and returns only an iterator.

### Why Are Iterators Stable After Insert?

Each node's memory doesn't move. Iterators point to node addresses, which remain unchanged.

### `map[key]` vs. `map.find(key)`

- **`map[key]`**: if `key` doesn't exist, **inserts a default-valued element** and returns a reference to it. Use only when you want this behavior.
- **`map.find(key)`**: returns `end()` if not found; never inserts. Preferred for safe lookup.

### Why Red-Black Trees?

- Plain BST: can degenerate to O(n) in worst case
- AVL tree: too strictly balanced — rotations on every insert/delete are expensive
- Red-black tree: guarantees O(log n) worst case with amortized cheap rebalancing — the optimal balance of guarantees and performance

---

## Deleting Elements While Iterating

**Sequence containers** (`vector`, `deque`): `erase` returns the next valid iterator.
```cpp
it = container.erase(it);
```

**Associative containers** (`map`, `set`, etc.): only the erased iterator is invalidated. Use:
```cpp
container.erase(it++);
// or
it = container.erase(it);  // C++11
```

---

## Iterators

**Why iterators?** Different containers have different internal structures — arrays, linked lists, trees. Iterators provide a uniform traversal interface so algorithms don't need to know container internals.

**Underlying mechanism**: iterators use *traits* (type deduction) and *partial template specialization* to dispatch different iterator categories to different algorithm implementations at compile time.

### Iterator Categories

1. **Input iterator**: read-only, single-pass forward
2. **Output iterator**: write-only, single-pass forward
3. **Forward iterator**: read-write, single-pass forward, position-preserving
4. **Bidirectional iterator**: forward + backward (`--`)
5. **Random-access iterator**: bidirectional + arithmetic (`+n`, `-n`, `[]`, `<`)

`vector`/`deque`/`array` → random-access; `list` → bidirectional; `forward_list` → forward; `set`/`map` → bidirectional.

### Iterator Invalidation Summary

- **`erase`** in sequence containers: invalidates erased and all subsequent iterators
- **`insert`** in sequence containers: if reallocation occurs, all iterators; otherwise, inserted position and later
- **Reallocation** (`push_back` beyond capacity): all iterators
- **Associative container `erase`**: only the erased iterator; all others remain valid

### Iterator vs. Pointer

Iterators are class templates that overload pointer-like operators (`*`, `->`, `++`, `--`). They are not raw pointers (though random-access iterators on `vector` may be raw pointers). Iterators provide a safe, container-aware abstraction over traversal.

---

## Are STL Containers Thread-Safe?

**No.** STL containers are not thread-safe by default.

Concurrent reads are generally safe. Concurrent writes, or simultaneous read and write, require external synchronization.

For `vector` specifically: even a single-writer, multi-reader scenario is unsafe if the writer triggers reallocation — the internal array moves, invalidating all readers' iterators.

**Workarounds**:
1. Add a mutex around all container accesses
2. For `vector` in concurrent read scenarios: pre-size with `resize()` before any concurrent access, then use only index-based reads with no `push_back` — avoids reallocation entirely:
   ```cpp
   vector<int> v;
   v.resize(1000);  // resize, not reserve — must actually construct elements
   ```
   Note: `resize`, not `reserve` (reserve doesn't construct elements, so concurrent reads may access uninitialized memory).

---

## Memory Pool / Connection Pool

In high-concurrency scenarios, frequently creating and destroying database connections (or any resource) is expensive due to:
1. TCP three-way handshake
2. Authentication overhead
3. Resource allocation/deallocation

A **connection pool** pre-allocates a set of connections. When a request needs a connection, it borrows one from the pool; when done, it returns it. This amortizes the connection overhead across many requests.

The same concept applies to **memory pools**: pre-allocate a block of memory and serve small allocations from it, avoiding the overhead of repeated `malloc`/`free`.

### Connecting to MySQL (Steps)

1. Initialize connection environment (`mysql_init`)
2. Connect to server — provide: IP, port, username, password, database name
3. Execute queries/updates
4. For writes: commit or rollback transactions based on success/failure
5. For reads: iterate through the result set
6. Free the result set and close the connection

---

# C++ Utility Patterns

## Double Buffer (Lock-Free for Read-Heavy Workloads)

In "one-writer, many-readers" scenarios, locks introduce contention. The double buffer pattern avoids this:

- Maintain two copies of the data: the **active** copy (readers use this) and the **backup** copy (writer updates this).
- The writer updates the backup while readers use the active copy.
- When the update is complete, atomically swap the two pointers.
- Readers always use the currently active copy; they never see partially-written data.

Implementation uses `shared_ptr` to safely manage lifetime: `use_count()` tells the writer whether any readers are currently using a copy before the swap.

**Limitations**: not suitable for "many-writers" or "many-writers, one-reader" patterns, because:
- Multiple writers still need a mutex among themselves
- The swap step itself still requires coordination

**Best use case**: configuration data, reference tables, or any data that is updated infrequently but read very frequently.

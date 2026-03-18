# C++ Interview Notes — Part 2

---

# Keywords

## `static` and `const`

### `static`

`static` can be used to declare static variables and static member functions. There are several categories:

**Local static variable**: stored in the static storage area. Initialized only once during the entire program run. Scope is still local — valid only within the function or block where it is defined. Resources are reclaimed by the OS when the program ends.

**Global static variable**: stored in the static storage area (which persists for the lifetime of the program). Uninitialized global static variables default to zero. Scope is limited to the file in which it is declared.

**Class static member variable**: shared by all objects of the class, including derived class objects. Must be initialized outside the class definition — cannot be initialized inside a constructor.

**Class static member function**: shared by all objects; has no `this` pointer; cannot access non-static members of the class.

### `const`

Used for constant declarations and declaring const member functions. `const` and `static` cannot simultaneously modify a class member function — `const` on a member function means "this function may not modify the object's state", while `static` means "this function belongs to the class, not to any object". The two are contradictory. When applied to a variable, `const` means the variable cannot be modified. When applied to a member function, it means no data members may be modified.

---

## Common Data Structures

1. **`vector`**: dynamic array; contiguous storage; supports random access.
2. **`deque`**: double-ended queue; contiguous segmented storage; supports random access.
3. **`list`**: doubly linked list; non-contiguous memory; does not support random access.
4. **`stack`**: stack (LIFO); no random access; elements can only be added or removed at one end.
5. **`queue`**: single-ended queue; elements added at the back, removed from the front.
6. **`set`**: sorted set; implemented as a red-black tree; supports random access. Lookup, insert, and delete are O(log n).
7. **`map`**: key-value map; implemented as a red-black tree; supports random access. Lookup, insert, and delete are O(log n).
8. **`hash_set`** / **`unordered_set`**: hash table; supports random access. Lookup, insert, and delete average O(1).

---

## `const` vs. `#define`

1. Both can define constants, but `const` is more versatile.
2. A `const` constant has a data type; a macro constant does not. The compiler performs type-safety checking on `const` but only does textual substitution for macros — no type checking, and substitution can produce unexpected errors.
3. Some integrated debugging tools can debug `const` constants but cannot debug macro constants.

---

## `struct` vs. `class`

In C++, both `class` and `struct` can define a class. The differences are:

1. **Default inheritance access**: inheritance from a `class` is `private` by default; inheritance from a `struct` is `public` by default.
2. **Default member access**: `class` members default to `private`; `struct` members default to `public`.

These two points are the fundamental, essential differences.

3. In C++, `struct` has been extended well beyond its C origins — it fully supports member functions, constructors, destructors, inheritance, polymorphism, and access specifiers (`private`, `protected`, `public`).
4. When a `class` inherits from a `struct` (or vice versa), the default access for the inheritance is determined by the **derived class** keyword, not the base class keyword:

```cpp
class A {};
class B : A {};   // private inheritance (B is a class)
struct B : A {};  // public inheritance (B is a struct)
```

**Which to use?** Both keywords work. By convention: use `struct` for plain data structures; use `class` for objects with behavior and encapsulation.

---

## When Does `delete` Need Square Brackets `[]`?

When the variable being deleted is an array. For example:
```cpp
int* arr = new int[10];
delete[] arr;
```
Using `delete` without `[]` on an array is undefined behavior.

---

## `sizeof` vs. `strlen` / `malloc` vs. `new` / Memory Layout

**`sizeof` vs. `strlen`**

- `sizeof` is a compile-time operator that returns the number of bytes of a type or variable — not affected by the variable's value. Examples: `sizeof(int)` = 4, `sizeof(char)` = 1.
- `strlen` is a runtime function that returns the number of characters in a C string, not including the null terminator `'\0'`. Example: `strlen("hello")` = 5.

**`malloc` vs. `new`**

Both allocate memory on the heap. Key differences:

- `malloc` returns `void*` (must be cast); `new` returns a typed pointer directly.
- `malloc` only allocates memory — does not call constructors. `new` allocates memory and then automatically calls the constructor.
- Memory from `malloc` is freed with `free`; memory from `new` must be freed with `delete`.

**C/C++ program memory layout**

- **Stack**: local variables, function parameters and return values. Managed automatically by the compiler.
- **Heap**: dynamically allocated memory (via `malloc`/`new`). Must be manually released with `free`/`delete`.
- **Global/static storage**: global variables and `static` variables. Allocated before `main` runs, released when the program ends.

---

## `strcpy` / `sprintf` / `memcpy` — Differences

All three are C standard library functions for string and memory manipulation:

1. **`strcpy(char *dest, const char *src)`**: copies a null-terminated string from `src` to `dest`, including the `'\0'` terminator.
2. **`sprintf(char *buf, const char *format, ...)`**: formats data according to a format string and stores the result as a string in `buf`.
3. **`memcpy(void *dest, const void *src, size_t n)`**: copies exactly `n` bytes of raw memory from `src` to `dest` — does not interpret the data as strings.

In summary: `strcpy` and `memcpy` are for copying; `sprintf` is for formatted conversion. Choose based on your specific need.

---

## Constants

**How to define a constant**: use the `const` keyword.

**Memory location of constants**:
- Local constants are stored on the stack.
- Global constants generally don't have a dedicated memory allocation — they are placed in the symbol table for access efficiency.
- String literals and other literal constants are stored in the constants section (read-only data segment).

---

## C++ Type Casts

**1. C-style cast** — simple but unsafe; not recommended.
```cpp
double d = 3.14;
int i = (int)d;
```

**2. `static_cast`** — for conversions between related types; type-checked at compile time.
```cpp
int i = static_cast<int>(3.14);
```

**3. `dynamic_cast`** — for safe downcasting in polymorphic hierarchies; type-checked at runtime. Returns `nullptr` (for pointers) or throws `std::bad_cast` (for references) if the cast fails.
```cpp
Base* basePtr = new Derived();
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
if (derivedPtr) {
    derivedPtr->show();
}
```

**4. `const_cast`** — adds or removes `const`/`volatile` qualifiers. Only valid on pointers or references; the underlying object must not actually be `const`.
```cpp
const int i = 42;
int* p = const_cast<int*>(&i);
```

**5. `reinterpret_cast`** — reinterprets the bit pattern of any pointer type as another pointer type, or between pointers and integers. Almost no type checking — use with extreme caution.
```cpp
int i = 42;
void* p = reinterpret_cast<void*>(&i);
int* ip = reinterpret_cast<int*>(p);
```

---

## `new`/`delete` vs. `malloc`/`free`

`new` and `delete` are **operators**; `malloc` and `free` are **library functions**.

**Why do we need `new`/`delete` when `malloc`/`free` exist?**

For non-built-in types (class objects), `malloc`/`free` alone are insufficient. Objects must have their constructor called when created and their destructor called before destruction. Since `malloc`/`free` are library functions (not compiler operators), the compiler cannot force them to invoke constructors and destructors. `new`/`delete` are operators under compiler control and handle this automatically.

---

# Pointers

## What Are Smart Pointers? What Types Exist?

A smart pointer is a wrapper class around a raw pointer that manages the pointer's lifetime automatically. When the smart pointer goes out of scope, its destructor releases the managed memory — eliminating the need for manual `delete` and preventing memory leaks.

### 1. `std::unique_ptr` — Exclusive Ownership

Only one `unique_ptr` can own a resource at a time. Copying is disallowed; ownership can be transferred with `std::move`.

```cpp
std::unique_ptr<int> ptr1(new int(10));
// std::unique_ptr<int> ptr2 = ptr1;        // ERROR: copying not allowed
std::unique_ptr<int> ptr2 = std::move(ptr1); // OK: transfer ownership
// ptr1 is now null
if (!ptr1) {
    std::cout << "ptr1 is null" << std::endl;
}
```

### 2. `std::shared_ptr` — Shared Ownership

Multiple `shared_ptr` instances can share ownership of the same resource. A reference count tracks how many owners exist; when the count drops to zero, the resource is automatically released.

```cpp
std::shared_ptr<int> ptr1 = std::make_shared<int>(20);
std::shared_ptr<int> ptr2 = ptr1;  // shared ownership
std::cout << ptr1.use_count() << std::endl;  // prints 2
```

### 3. `std::weak_ptr` — Non-Owning Weak Reference

`weak_ptr` does not affect the reference count. Used alongside `shared_ptr` to break circular references. Must use `lock()` to obtain a `shared_ptr` for actual access.

```cpp
std::shared_ptr<int> ptr1 = std::make_shared<int>(30);
std::weak_ptr<int> weakPtr = ptr1;
if (auto sharedPtr = weakPtr.lock()) {
    std::cout << *sharedPtr << std::endl;
} else {
    std::cout << "weakPtr is expired" << std::endl;
}
```

### Summary

| Type | Ownership | Copy | Notes |
|---|---|---|---|
| `unique_ptr` | Exclusive | ✗ | Transfer via `std::move` |
| `shared_ptr` | Shared | ✓ | Reference counting |
| `weak_ptr` | None | ✓ | Breaks circular refs; use `lock()` to access |

---

## Pointers vs. References — Differences and When to Use Each

**Differences**:

1. A pointer is a variable that stores an address — it is an entity in its own right. A reference is merely an alias for an existing variable — it *is* the original variable, just with another name.
2. There are `const` pointers, but no `const` references (a reference is already inherently "const" in the sense that it cannot be reseated).
3. Pointers can be multi-level (`int** p` is valid); references can only be one level (`int&& a` as a variable declaration is not a double reference — it's an rvalue reference, a different concept).
4. A pointer can be null; a reference must be initialized to a valid object and cannot be null.
5. A pointer can be reassigned to point to a different object after initialization; a reference permanently refers to the same object from the moment of initialization.
6. `sizeof(reference)` yields the size of the referenced object; `sizeof(pointer)` yields the pointer size itself (4 or 8 bytes).
7. The `++` operator on a pointer advances the memory address; on a reference it increments the referenced value.

**Similarities**:
1. Both can be used to modify the variable they refer/point to.
2. Both are address-based concepts at a low level.

**When to use which**:
1. Use a **pointer** when there is a possibility of pointing to nothing (null).
2. Use a **pointer** when you need to point to different objects at different times.
3. Use a **reference** when it will always refer to the same object and will never be null.
4. Use a **reference** when overloading operators (the syntax is cleaner).

---

## Wild Pointers vs. Dangling Pointers

**Wild pointer**: a pointer that has never been initialized. It points to an unknown or random memory address. Using it causes crashes or undefined behavior.

**Dangling pointer**: a pointer that once pointed to valid memory, but that memory has since been freed or gone out of scope. Accessing it accesses invalid memory, potentially causing crashes or data corruption.

**Key distinction**:
- Wild pointer → never initialized → random address
- Dangling pointer → was valid, now stale → freed or out-of-scope memory

---

## How to Avoid Wild Pointers

1. Initialize all pointers to `nullptr` at declaration.
2. Set pointers to `nullptr` immediately after calling `delete` or `free`.
3. Use smart pointers — they manage lifetime automatically.
4. Avoid manual array pointer arithmetic; use STL containers (`vector`, etc.) instead.
5. For pointers to stack variables, be careful about the variable's lifetime — never use such a pointer after the variable has gone out of scope.

---

## Function Pointers

**Definition**: a function pointer is a pointer variable that stores the entry-point address of a function. Every function has a unique entry address assigned at compile time. Once you have a function pointer, you can call the function through it, just as you would use any other pointer to access a value.

**Uses**: calling a function indirectly, and passing functions as arguments to other functions — most commonly used for **callback functions**.

---

# Object-Oriented Programming

## What Is Your Understanding of OOP?

C++ object-oriented programming treats everything as objects. Each object is described by its **attributes** (data) and **methods** (behavior). For example, defining a "Cat" object: the cat's eyes, fur, and mouth are attributes; the cat's meowing and walking are methods.

Programming with objects improves code reusability and maintainability compared to purely procedural code. The three pillars of C++ OOP are **encapsulation**, **inheritance**, and **polymorphism**.

---

## The Three Features of OOP / Empty Class Member Functions

**The three features of OOP**:

- **Encapsulation**: data and the functions that operate on it are bundled together in a class. Implementation details are hidden; only a necessary interface is exposed. This ensures data security and reliability.
- **Inheritance**: a new class can extend an existing class, reusing and augmenting its functionality. Reduces code duplication; improves reusability and maintainability.
- **Polymorphism**: through function overloading, virtual functions, and templates, different objects can respond differently to the same message. Increases program flexibility and extensibility.

**Empty class default-generated member functions**:

A C++ class with no explicitly defined members will have the following automatically generated by the compiler when needed:
- Default constructor
- Destructor
- Copy constructor
- Copy assignment operator (`operator=`)
- (C++11) Move constructor and move assignment operator

---

## Encapsulation

### Parameter Passing: by Value, by Pointer, by Reference

**1. Pass by value**: the parameter is a copy of the argument — a separate storage location. Changes to the parameter inside the function have no effect on the original argument. This is the default behavior.

**2. Pass by pointer**: the argument passed is an address. The parameter is a pointer. Any modification to the data *at* that address inside the function affects the original variable. However, reassigning the pointer itself (making it point elsewhere) does not affect the caller's pointer.

**3. Pass by reference**: the reference parameter is an alias for the caller's variable. All operations on the reference are operations on the original. Any change is immediately reflected in the caller.

**Efficiency**:
- For built-in types, all three are roughly equivalent.
- For user-defined types (classes), pass by reference is most efficient because it avoids making a copy of the object.

---

### Override vs. Redefine (Hiding) in C++

**Override (重写)**: redefining a virtual function from a base class in a derived class. The function signature (return type, name, parameter list) must be identical. When called through a base class pointer or reference, the derived class version is invoked (dynamic dispatch).

Characteristics:
1. Must be a `virtual` function in the base class.
2. Signature must exactly match the base class virtual function.
3. Use the `override` keyword (C++11+) to make the intent explicit.

```cpp
class Base {
public:
    virtual void display() const {
        std::cout << "Base display" << std::endl;
    }
};

class Derived : public Base {
public:
    void display() const override {  // overrides Base::display
        std::cout << "Derived display" << std::endl;
    }
};

// Base* ptr = new Derived(); ptr->display(); → prints "Derived display"
```

**Redefine / Hide (重定义)**: defining a function in a derived class that has the same name as a non-virtual function in the base class. This hides the base class function — it does not participate in dynamic dispatch.

Characteristics:
1. Applies to non-virtual functions.
2. Signature may or may not match the base class function.
3. Calling through a base class pointer always calls the base version, regardless of the actual object type.

```cpp
class Base {
public:
    void display() const { std::cout << "Base display" << std::endl; }
};

class Derived : public Base {
public:
    void display() const { std::cout << "Derived display" << std::endl; }
};

Base* ptr = new Derived();
ptr->display();  // prints "Base display" — non-virtual, static dispatch
```

**Summary**:

| | Override | Redefine/Hide |
|---|---|---|
| Applies to | Virtual functions | Non-virtual functions |
| Signature | Must match exactly | May differ |
| Via base pointer | Calls derived version | Calls base version |
| Keyword | `override` (recommended) | — |

---

### C++ Constructors Overview

1. **Default constructor**: no parameters; used for default initialization.
2. **Parameterized constructor**: takes arguments; initializes member variables.
3. **Copy constructor**: initializes a new object as a copy of an existing one.
4. **Move constructor**: transfers resources from a temporary object to a new one, avoiding unnecessary copying.
5. **Delegating constructor** (C++11): one constructor calls another constructor of the same class to reduce code duplication.

---

### Why Constructors Are Generally Not Virtual

1. Calling a virtual function only requires knowing the function's interface — not the object's complete type. But *creating* an object requires knowing its complete, exact type. A virtual constructor is therefore conceptually meaningless — you can't dispatch to the right constructor if the object doesn't yet exist to tell you what type it is.

2. From an implementation perspective: virtual function calls are resolved by following the object's `vptr` to the vtbl. At the moment a constructor is called, the `vptr` hasn't been set up yet (that's part of what the constructor does). So there's no vtbl to consult, making a virtual dispatch impossible.

---

### Destructors

A destructor has no parameters, no return value, cannot be overloaded, and a class can have only one. Its purpose is to release resources (memory, file handles, etc.) owned by the object.

---

### Why Destructors Are Generally Declared Virtual

Without a virtual destructor, if a base class pointer points to a derived class object and `delete` is called on that pointer, only the base class destructor runs. The derived class destructor is never called, leading to resource leaks.

With a virtual destructor, the compiler uses dynamic dispatch to call the correct destructor — the derived class destructor runs first, then the base class destructor — ensuring proper cleanup.

**Rule**: if a class is designed to be a base class (i.e., someone might `delete` a derived object through a base pointer), its destructor must be `virtual`.

---

### Calling Virtual Functions in Constructors or Destructors

This is strongly discouraged, because it does not produce the intended polymorphic behavior.

**In a constructor**: when a derived class object is being constructed, the base class portion is built first. While the base class constructor is running, the derived class portion doesn't exist yet. The compiler treats the object as a base class instance at that point, so any virtual function call resolves to the **base class version** — not the derived class version.

**In a destructor**: the reverse applies. After the derived class destructor has run, the derived class portion is considered destroyed. Any virtual function call in the base class destructor resolves to the **base class version**.

In short: virtual function calls in constructors/destructors do not produce dynamic dispatch — they always resolve to the version belonging to the class whose constructor/destructor is currently executing.

---

## Inheritance

### When to Use Inheritance vs. Composition

**Inheritance**: a mechanism for code reuse by extending an existing class.

**Composition**: building a new class by embedding existing class objects as members.

**Guidelines**:
1. If the relationship is "is-a" and the derived class should expose the full interface of the base class, use **inheritance**.
2. If the relationship is "has-a", prefer **composition**.
3. When in doubt — and there is no compelling reason to use inheritance — default to **composition**. Composition operates at the implementation level; inheritance operates at the extension/interface level. If you don't need everything the base class offers (including all its interface), don't inherit — you'd be forced to inherit things you don't need, which is inheritance abuse.

---

### Inheritance vs. Derivation — What's the Difference?

**Different perspectives**: inheritance is described from the **child class's** point of view; derivation is described from the **parent class's** point of view.

**Different definitions**:
- *Derivation*: a new class branches off from an existing class, extending or specializing it.
- *Inheritance*: the child class acquires the parent class's attributes, methods, and interface, and may redefine or augment them.

They describe the same relationship from opposite ends.

---

### Single Inheritance vs. Multiple Inheritance

**Single inheritance**: a derived class has exactly one direct base class.
```cpp
class Derived : public Base {};
```

**Multiple inheritance**: a derived class has two or more direct base classes.
```cpp
class Derived : public Base1, protected Base2, private Base3 {};
```

Each inheritance specifier applies only to the base class immediately following it, not to subsequent ones.

---

## Polymorphism

### Understanding Polymorphism

1. **Definition**: the same operation produces different results when applied to different objects. In C++, polymorphism means that when a virtual member function is called, the actual function invoked depends on the real type of the object, not the declared type of the pointer/reference.

2. **Implementation**: via virtual functions (`virtual` keyword). A base class pointer or reference can point to different derived objects; calling a virtual function through it invokes the appropriate derived class implementation.

3. **Overload (重载)**: functions with the same name but different parameter types or order. Resolved at compile time.

4. **Override (重写)**: a derived class redefines a `virtual` function from a base class. Resolved at runtime.

5. **Hide (隐藏 / Overwrite)**: a derived class function shadows a base class function of the same name. If signatures differ — always hides. If signatures match but the base function is not `virtual` — also hides (not overrides). The base class function is no longer accessible via the derived class name without explicit qualification.

---

### How Polymorphism Works / Linked Lists vs. Arrays / Queues vs. Stacks

**Polymorphism implementation**:

The key mechanism is virtual functions and pointers. Declaring a base class function as `virtual` enables derived classes to override it. When called through a base class pointer or reference, the call is dispatched at runtime based on the actual type of the pointed-to object (dynamic binding). This allows the same function call syntax to produce type-specific behavior.

**Linked lists vs. arrays**:

- **Array**: contiguous memory; fixed size; O(1) random access by index; O(n) insert/delete in the middle (elements must be shifted).
- **Linked list**: non-contiguous nodes; dynamic size; O(n) traversal to access an element; O(1) insert/delete at a known position (just pointer adjustments).

**Queues vs. stacks**:

- **Queue**: FIFO (first in, first out) — insert at the back, remove from the front. Typical use: OS process scheduling.
- **Stack**: LIFO (last in, first out) — insert and remove at the top only. Typical uses: function call management, expression evaluation.

---

### Pure Virtual Functions (Interface Inheritance vs. Implementation Inheritance)

The purpose of pure virtual functions is to give base classes flexibility in what they offer to derived classes:

1. Sometimes we want derived classes to inherit **only the interface** (the function signature), not any implementation.
2. Sometimes we want derived classes to inherit **both the interface and a default implementation**, which they may override.
3. Sometimes we want derived classes to inherit the interface and implementation but **must provide their own override** — the base class provides no acceptable default.

A pure virtual function (`virtual void f() = 0;`) requires that any non-abstract derived class provide its own implementation. The class containing the pure virtual function becomes an **abstract class** and cannot be instantiated.

Interestingly, a pure virtual function *can* have a body defined in the base class, but it can only be called explicitly (e.g., `Base::f()`), not through normal virtual dispatch.

---

### Virtual Function Table (vtbl)

Polymorphism is implemented via virtual functions, and virtual functions are implemented via the virtual function table.

If a class contains virtual functions, the compiler creates a **virtual function table (vtbl)** for that class — an array of function pointers, one per virtual function. Every object of that class contains a hidden **virtual pointer (vptr)** (stored at the very beginning of the object's memory, for performance) that points to the class's vtbl.

Important distinctions:
- Each **object** has a `vptr`.
- Each **class** has a `vtbl`.
- A derived class generates its own vtbl, which is compatible with the base class's vtbl (base function slots are present, overridden ones replaced with derived versions).

---

### When to Use Virtual Functions / Difference from Pure Virtual Functions / Virtual Destructor

**When to use virtual functions**: when you want runtime polymorphism — when calling a member function through a base class pointer or reference should invoke the most-derived override. Virtual destructors are a special case: they ensure correct destruction order when deleting through a base pointer.

**Differences between virtual and pure virtual functions**:

| | Virtual function | Pure virtual function |
|---|---|---|
| Implementation in base class | Yes (has a body) | No (only declaration: `= 0`) |
| Class type | Ordinary (instantiable) | Abstract (cannot be instantiated) |
| Derived class obligation | May override | **Must** override |
| Can be called directly | Yes | Only via explicit scope (`Base::f()`) |

Additional notes:
1. A class with at least one pure virtual function is abstract and cannot be instantiated.
2. Virtual functions cannot be `static` — `static` functions have no `this` pointer, which is required for vtbl lookup.
3. The correct form: `virtual void f() { }` for virtual; `virtual void f() = 0;` for pure virtual.
4. Both virtual and pure virtual functions support polymorphic calling via base class pointers/references.
5. C++ supports two kinds of polymorphism: **compile-time** (function/operator overloading) and **runtime** (virtual functions).

---

### Pure Virtual vs. Ordinary Virtual Functions

1. **Pure virtual**: declared with `= 0`; no implementation in the base class; the derived class **must** provide one. The base class becomes abstract.
2. **Ordinary virtual**: has both declaration and implementation in the base class; the derived class *may* override it but is not required to.

---

### Purpose of Virtual Inheritance

Virtual inheritance eliminates the **diamond problem** in multiple inheritance — it ensures that when a class inherits from two classes that both derive from a common base, only **one instance** of that common base exists in the most-derived object.

Without virtual inheritance: class D would contain two separate copies of class A's members, causing ambiguity when accessing them.

With virtual inheritance (`class B : public virtual A`): the most-derived class (D) contains exactly one shared instance of A, and D's constructor is responsible for initializing A directly.

---

# Processes and Threads

## Processes vs. Threads

A **process** is the unit of resource allocation by the OS; a **thread** is the unit of scheduling. Threads within a process share the process's resources.

**Process**: a program in execution — an instance of a running program, including its program counter, registers, and current variable values. A process is the execution flow of a program, along with all the resources the program needs. Multiple processes can run concurrently; at any given instant a single CPU core runs one process, but by rapidly switching between processes (time-slicing) the OS creates the illusion of parallelism.

**Thread**: an execution flow within a process. A process can have multiple threads. From a resource perspective: a process bundles related resources (address space, open files, etc.) into a platform. From a runtime perspective: a process is code executing on that resource platform — but a thread is what actually executes. The relationship is: **process = thread(s) + shared resources**.

---

## Inter-Process Communication (IPC) Methods

- Pipes
- Message queues
- Shared memory
- Semaphores
- Sockets
- Files

---

## Deadlock — What It Is and How It Occurs

A deadlock occurs when a set of threads are each waiting for a resource held by another thread in the set — everyone is blocked, nobody can proceed.

**Four necessary conditions for deadlock** (all must hold simultaneously):

1. **Mutual exclusion**: a resource can only be held by one process/thread at a time.
2. **No preemption**: a resource can only be released voluntarily by the process/thread holding it.
3. **Hold and wait**: a thread holds at least one resource while waiting to acquire additional resources held by other threads.
4. **Circular wait**: a cycle exists in the resource-allocation graph — T1 waits for T2's resource, T2 waits for T3's, ..., Tn waits for T1's.

---

## How to Resolve Deadlock

1. **Avoidance**: break one of the four necessary conditions — e.g., enforce a global resource acquisition order to prevent circular wait.
2. **Prevention**: use techniques like resource reservation (request all needed resources upfront), safe sequences, or deadlock detection algorithms.
3. **Recovery**: once deadlock is detected, apply algorithms like the Banker's algorithm or resource preemption to break the deadlock.

---

## Optimizing Multi-threaded Performance When Locking Degrades It

1. **Reduce lock granularity**: group multiple operations into a single critical section to minimize the number of lock/unlock operations.
2. **Use atomic operations**: C++11 provides atomic types (`std::atomic<T>`) and operations (e.g., `compare_exchange_strong`) that replace locks for simple shared-variable updates.
3. **Use finer-grained locking mechanisms**: reader-writer locks (`shared_mutex`) allow concurrent reads while protecting writes. Condition variables replace polling.
4. **Use coarser-grained locking strategically**: sometimes a single coarse lock reduces overhead compared to many fine-grained locks with high contention.
5. **Use non-blocking (lock-free) techniques**: CAS (Compare-And-Swap) based algorithms eliminate blocking entirely and are often the highest-performing option for specific patterns.

---

# Compilation

## Static Libraries vs. Dynamic Libraries

**Static library** (`.lib` / `.a`): library code is fully compiled and linked into the executable at link time. The resulting binary is self-contained.
- Linking happens at compile time.
- The program has no runtime dependency on the library file.
- Portable and easy to distribute.
- Downside: larger binary size; each process has its own copy of the library code in memory; updating the library requires recompiling the program.

**Dynamic library** (`.dll` / `.so`): library code is loaded at runtime (not embedded in the executable).
- Linking is deferred to runtime.
- Multiple processes can share a single copy of the library in memory (hence also called "shared library").
- Updates to the library don't require recompiling programs that use it.
- Explicit (programmatic) loading is possible, giving full control to the developer.
- Downside: the shared library file must be present at runtime.

**Difference between a dynamic library's `.lib` and a static library's `.lib` (on Windows)**:
- A dynamic library's `.lib` is an **import library** — it only contains address symbol tables and stubs that tell the linker where to find the actual code at runtime. The actual code lives in the `.dll`.
- A static library's `.lib` **contains the actual executable code** and symbol tables directly.

---

# Memory

## Common Causes of Memory Leaks / How to Prevent Them

**Memory leak**: the program allocates heap memory (`new` / `malloc`) but never releases it (`delete` / `free`). Over time the unreleased memory accumulates, reducing available memory and potentially crashing the application.

**Common causes**:
1. Pointer reassignment (losing the only pointer to an allocation before freeing it)
2. Incorrect memory release (e.g., calling `delete` on a `new[]` allocation)
3. Not handling return values properly (ignoring a pointer returned from an allocating function)
4. `new` and `delete` not paired

**How to prevent**:
1. Never access a null pointer — always check before use.
2. Prefer smart pointers (`unique_ptr`, `shared_ptr`) over raw pointers.
3. Always pair `new` with `delete` and `new[]` with `delete[]`.

---

## Three Ways C++ Allocates Memory

1. **Static storage area**: allocated at compile time; exists for the entire program lifetime. Includes global variables and `static` variables.
2. **Stack**: allocated and released automatically as functions are called and return. Stores local variables. Stack allocation is implemented in hardware (push/pop instructions) — very fast, but capacity is limited.
3. **Heap**: dynamic memory allocated at runtime via `new`/`malloc`; released manually via `delete`/`free`. Lifetime is programmer-controlled; flexible but prone to leaks and fragmentation.

---

## C++ Memory Segments

C++ memory is divided into five regions:

- **Code segment**: stores compiled executable instructions; typically read-only.
- **Data segment**: stores global variables and `static` variables (initialized ones in the data segment, uninitialized ones in BSS).
- **Heap**: dynamic memory; programmer-managed via `new`/`malloc` and `delete`/`free`.
- **Stack**: local variables and function call frames; managed automatically by the compiler.
- **Constants section**: stores constant data (string literals, `const` globals); typically read-only.

---

## Struct Memory Alignment — Rules and Reasons

**Why align?** CPUs access memory most efficiently when data is stored at addresses that are multiples of the data's size. Misaligned accesses may require multiple memory reads (slower) or, on some architectures (e.g., SPARC), cause a hardware fault. Memory alignment trades a small amount of space for significantly better access performance.

**Alignment rules** (GCC x86 default — 4-byte alignment; adjustable with `__attribute__((packed))` or `#pragma pack(n)`):

- Each member is placed at an offset that is a multiple of its own size (or the packing value, whichever is smaller).
- The total struct size is padded to a multiple of the largest member's alignment requirement.

---

# Comparisons with Other Languages

## C++ vs. C

1. C++ introduces new keywords and syntax, plus user-defined namespaces.
2. C++ supports function overloading and virtual functions (for polymorphism).
3. C++ introduces the `class` concept with access control (`private`, `public`, `protected`). C has only `struct`, whose members are `public` by default. C++ `class` defaults to `private`.
4. C++ adds templates and the STL standard library.
5. C favors procedural programming; C++ is an object-oriented language (while still supporting procedural style).

---

## C++ vs. Java

1. **Memory management**: in C++, the programmer manually allocates and frees memory. Java has an automatic garbage collector.
2. **Language features**:
   - C++ supports multiple inheritance; Java supports inheriting from only one class but implementing multiple interfaces.
   - C++ has implicit type conversion; Java requires explicit casts.
3. **Type system**: Java eliminates `struct` and `union` — everything must be part of a class. C++ retains both.

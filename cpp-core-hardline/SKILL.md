---
name: cpp-core-hardline
description: >
  Enforce CppCoreGuidelines mandatory red-line rules (hardline subset) when writing,
  reviewing, or refactoring C++ code. This skill is triggered when C++ code is involved
  in any capacity — writing new code, reviewing existing code, refactoring, or code audit.
  It covers 30 non-negotiable rules distilled from the C++ Core Guidelines, covering
  resource management, type safety, concurrency, interface design, modern C++ idioms,
  and common pitfalls. When triggered, every C++ code change MUST satisfy all 30 rules
  without exception.
---

# CPP Core Hardline

## Overview

This skill encodes a hardline subset of the C++ Core Guidelines — 30 non-negotiable,
mandatory rules. These rules represent the minimum safety baseline for modern C++
development. When this skill is active, every C++ code review, edit, or generation
MUST comply with all 30 rules. Violations are treated as blocking defects.

## When to Use This Skill

Trigger this skill whenever:
- Writing new C++ code (headers, source files, templates)
- Reviewing or auditing existing C++ code
- Refactoring or modernizing legacy C++
- Generating C++ code snippets, classes, or functions
- User mentions "C++" alongside "guideline", "standard", "review", "audit", or "best practice"

## Rule Priority

Rules are ordered by priority: **Safety** > **Correctness** > **Readability** > **Performance**.

- Rules 1–10: Safety — prevent UB, memory errors, leaks, data races, and undefined behavior
- Rules 11–20: Correctness — type safety, semantic correctness, and interface design
- Rules 21–27: Readability — make intent explicit and code self-documenting
- Rules 28–30: Performance — avoid unnecessary copies and allocations

---

## Part I: Safety (Rules 1–10)

### Rule 1: No Bare new/delete/malloc/free — RAII Everywhere

Explicit `new`, `delete`, `malloc`, `free`, `calloc`, or `realloc` are forbidden.
All resource acquisition MUST go through RAII wrappers. Use factory functions
(`std::make_unique`, `std::make_shared`) for heap allocation.

**Violation:**
```cpp
int* p = new int[100];  // BAD: bare new
delete[] p;             // BAD: bare delete
```

**Compliant:**
```cpp
auto p = std::make_unique<int[]>(100);  // GOOD: RAII
// or
std::vector<int> v(100);  // GOOD: container
```

---

### Rule 2: No Raw Owning Pointers — Use Smart Pointers

Never use a raw pointer (`T*`) to express ownership. All owning resources MUST be
managed by `std::unique_ptr<T>` (exclusive ownership) or `std::shared_ptr<T>`
(shared ownership). Raw pointers are only acceptable for non-owning observation
(e.g., function parameters, cached references).

**Violation:**
```cpp
Widget* w = new Widget();  // BAD: raw owning pointer
delete w;
```

**Compliant:**
```cpp
auto w = std::make_unique<Widget>();  // GOOD
std::shared_ptr<Widget> w2 = std::make_shared<Widget>();  // GOOD
```

---

### Rule 3: No Manual Resource Management in Destructors — Use RAII Members

Destructors MUST NOT contain manual cleanup code (`delete`, `free`, `CloseHandle`,
etc.). Instead, every resource should be wrapped in its own RAII class
(`std::unique_ptr` with custom deleter, `std::fstream`, etc.), and the destructor
should rely on member destruction order.

**Violation:**
```cpp
class Connection {
    ~Connection() { close(fd_); }  // BAD: manual resource cleanup
    int fd_;
};
```

**Compliant:**
```cpp
class Connection {
    // No destructor needed — FileHandle::~FileHandle() closes automatically
    FileHandle fd_;
};
```

---

### Rule 4: No Dangling References or Pointers

Never return, store, or pass a reference/pointer to a local stack object,
temporary, or any object whose lifetime has ended. This includes:
- Returning `const T&` or `T*` to a local variable.
- Storing `std::string_view` from a temporary `std::string`.
- Capturing by reference in a lambda that outlives the captured variable.

**Violation:**
```cpp
std::string_view sv = std::string("temp");  // BAD: sv is dangling
```

**Compliant:**
```cpp
std::string s = "temp";
std::string_view sv = s;  // GOOD: s outlives sv
```

---

### Rule 5: Move Semantics — Return Locals, Never Local References

When returning a local object from a function, rely on move semantics or RVO/NRVO.
NEVER return a reference or pointer to a local (stack) variable. Move-eligible
objects (`std::move` on return) are generally unnecessary and can inhibit RVO —
let the compiler do its job.

**Violation:**
```cpp
const std::string& get() {
    std::string local = "hello";
    return local;  // BAD: dangling reference
}
```

**Compliant:**
```cpp
std::string get() {
    std::string local = "hello";
    return local;  // GOOD: RVO or move
}
```

---

### Rule 6: No Raw Arrays — Use std::array / std::vector

C-style arrays (`T[]`, `T[N]`) are forbidden. Use `std::array<T, N>` for
compile-time fixed sizes and `std::vector<T>` for dynamic sizes. For string
buffers, prefer `std::string` or `std::string_view`.

**Violation:**
```cpp
int data[100];              // BAD: raw array
char buffer[256];           // BAD: raw char array
void process(int arr[]);    // BAD: decays to pointer
```

**Compliant:**
```cpp
std::array<int, 100> data;           // GOOD
std::vector<char> buffer(256);       // GOOD
void process(std::span<int> arr);    // GOOD: C++20
```

---

### Rule 7: No C-Style String Functions — Use std::string

C-style string manipulation functions (`strcpy`, `strcat`, `strlen`, `sprintf`,
`strcmp`, etc.) are forbidden. Use `std::string`, `std::string_view`, and
`std::format` (C++20) / `fmt::format` instead. The only exception is when
interfacing with C APIs — and even then, wrap in a safe abstraction.

**Violation:**
```cpp
char buf[256];
sprintf(buf, "value: %d", x);  // BAD: buffer overflow risk
```

**Compliant:**
```cpp
std::string buf = std::format("value: {}", x);  // GOOD: C++20
```

---

### Rule 8: No void* or C Varargs (va_list)

`void*` erases type safety and MUST NOT be used except for low-level interop
with C APIs (and even then, wrap it in a type-safe interface). C-style variadic
functions (`va_list`, `va_start`, `va_arg`, `va_end`) are forbidden. Use variadic
templates, `std::initializer_list`, or `std::variant` instead.

**Violation:**
```cpp
void process(void* data);              // BAD: type-erased
void log(const char* fmt, ...);        // BAD: C varargs
```

**Compliant:**
```cpp
template<typename T>
void process(T* data);                 // GOOD: type-safe

template<typename... Args>
void log(std::format_string<Args...> fmt, Args&&... args);  // GOOD: C++20
```

---

### Rule 9: No Uninitialized Variables — Always Initialize

Every variable MUST be initialized at the point of declaration. Use brace
initialization (`{}`) or value initialization to guarantee defined state.
Never rely on default-initialization leaving POD types in an indeterminate
state.

**Violation:**
```cpp
int x;               // BAD: uninitialized
std::string s;       // OK (default-constructed) but int is not
Widget* w;           // BAD: uninitialized pointer
```

**Compliant:**
```cpp
int x = 0;           // GOOD
int x{};             // GOOD: zero-initialized
Widget* w = nullptr; // GOOD
```

---

### Rule 10: Concurrency — No Lock-Free Shared Mutable State

Shared mutable state between threads MUST be protected by synchronization:
- `std::mutex` + `std::lock_guard` / `std::scoped_lock` for complex state.
- `std::atomic<T>` for simple shared scalars.
- Never rely on "happens to work" lock-free patterns without formal proof.
- Never share mutable data between threads without synchronization.

**Violation:**
```cpp
int shared_counter = 0;  // BAD: accessed from multiple threads without sync
```

**Compliant:**
```cpp
std::atomic<int> shared_counter{0};  // GOOD
// or
std::mutex mtx;
int shared_counter = 0;  // protected by mtx
```

---

## Part II: Correctness (Rules 11–20)

### Rule 11: Type Casts — No C-Style, Use Named Casts

C-style casts `(type)expr` and function-style casts `type(expr)` are forbidden.
Use the appropriate named cast:
- `static_cast<T>` for well-defined implicit conversions.
- `dynamic_cast<T>` for safe downcasting in polymorphic hierarchies.
- `const_cast<T>` only when interfacing with legacy const-incorrect APIs (and document it).
- `reinterpret_cast<T>` only for low-level interop (and document it).

**Violation:**
```cpp
int x = (int)3.14;         // BAD: C-style cast
auto p = (Base*)derived;   // BAD: C-style cast
```

**Compliant:**
```cpp
int x = static_cast<int>(3.14);               // GOOD
auto p = dynamic_cast<Base*>(derived);         // GOOD
```

---

### Rule 12: Use nullptr, Never 0 or NULL for Pointers

Always use `nullptr` for null pointer values. `0` and `NULL` are integer types
and can cause ambiguous overload resolution and type deduction issues.

**Violation:**
```cpp
Widget* w = NULL;      // BAD
if (w == 0) { }        // BAD
void foo(int);
void foo(Widget*);
foo(NULL);             // BAD: calls foo(int), not foo(Widget*)!
```

**Compliant:**
```cpp
Widget* w = nullptr;   // GOOD
if (w == nullptr) { }  // GOOD
foo(nullptr);          // GOOD: unambiguously calls foo(Widget*)
```

---

### Rule 13: Use enum class, Not Plain enum

Always use `enum class` (scoped enumerations) instead of plain `enum`.
`enum class` provides type safety, avoids namespace pollution, and prevents
implicit conversion to `int`.

**Violation:**
```cpp
enum Color { Red, Green, Blue };  // BAD: unscoped enum
Color c = Red;  // Red pollutes namespace
int x = Red;    // BAD: implicit conversion to int
```

**Compliant:**
```cpp
enum class Color { Red, Green, Blue };  // GOOD
Color c = Color::Red;  // scoped
// int x = Color::Red;  // ERROR: no implicit conversion
```

---

### Rule 14: Virtual Destructor for Polymorphic Bases

Every class with any virtual function MUST have a virtual destructor (either
user-declared or `= default`). Classes without virtual functions MUST NOT have
a virtual destructor — do not pay for polymorphism you don't use.

**Violation:**
```cpp
class Base {
public:
    virtual void foo();  // BAD: virtual function but no virtual destructor
};
```

**Compliant:**
```cpp
class Base {
public:
    virtual ~Base() = default;  // GOOD
    virtual void foo();
};
```

---

### Rule 15: Rule of Five — Declare All or None

If a class declares or deletes any of: destructor, copy constructor, copy
assignment, move constructor, or move assignment, it MUST declare or explicitly
`= default` / `= delete` all five. Resource-managing classes must be clear about
their copy and move semantics.

**Violation:**
```cpp
class Buffer {
    ~Buffer() { delete[] data; }  // BAD: custom destructor, but no copy/move declared
    char* data;
};
```

**Compliant:**
```cpp
class Buffer {
public:
    ~Buffer() { delete[] data; }
    Buffer(const Buffer&) = delete;            // explicit: non-copyable
    Buffer& operator=(const Buffer&) = delete;
    Buffer(Buffer&&) noexcept = default;       // movable
    Buffer& operator=(Buffer&&) noexcept = default;
private:
    char* data;
};
```

---

### Rule 16: No Slicing — Pass/Return Polymorphic Objects by Reference/Pointer

Never pass or return a polymorphic (inheritance hierarchy) object by value.
Doing so slices the object to the base type. Always pass by `const T&` or
by `std::unique_ptr<T>` / `std::shared_ptr<T>`.

**Violation:**
```cpp
void draw(Shape s) { }  // BAD: slices derived objects
std::vector<Shape> shapes;  // BAD: cannot store Circle, Square, etc.
```

**Compliant:**
```cpp
void draw(const Shape& s) { }  // GOOD
std::vector<std::unique_ptr<Shape>> shapes;  // GOOD
```

---

### Rule 17: Exceptions — Handle or Declare noexcept, Never Swallow

- Functions that never throw MUST be declared `noexcept`.
- Catch blocks MUST handle or translate the exception; never silently swallow
  (`catch(...) { }` with no action) without a documented, justified reason.
- Destructors, move constructors, move assignments, and swap functions MUST
  be `noexcept`.

**Violation:**
```cpp
void safe_op() { }  // BAD: not noexcept despite never throwing

try { risky(); }
catch (...) { }  // BAD: silent swallow
```

**Compliant:**
```cpp
void safe_op() noexcept { }  // GOOD

try { risky(); }
catch (const std::exception& e) { log(e.what()); }  // GOOD: handled
```

---

### Rule 18: Avoid Default Capture in Lambdas — Be Explicit

Never use `[=]` or `[&]` default capture in lambdas unless the lambda is
immediately consumed (e.g., passed directly to a function and not stored).
Explicit capture (`[this, &x, y]`) makes lifetime dependencies visible and
prevents accidental dangling references.

**Violation:**
```cpp
auto func = [&]() { return x + y; };  // BAD: implicit capture — what does it depend on?
```

**Compliant:**
```cpp
auto func = [&x, y]() { return x + y; };  // GOOD: explicit capture
```

---

### Rule 19: explicit Constructors for Single-Argument Constructors

Every constructor callable with a single argument MUST be declared `explicit`
unless implicit conversion is intentionally desired and documented. This
includes constructors with default parameters where only one argument is required.

**Violation:**
```cpp
class String {
public:
    String(const char* s);  // BAD: implicit conversion from const char*
};
```

**Compliant:**
```cpp
class String {
public:
    explicit String(const char* s);  // GOOD
};
```

---

### Rule 20: Function Parameters — const&, Value, or Return

- **Read-only (in) parameters**: Pass by `const T&` for non-trivial types.
- **Small trivially-copyable types** (≤ 2 words): Pass by value.
- **Output**: Use return values, NOT output parameters (`T& out` or `T* out`).
  If multiple outputs are needed, return `std::tuple` or a named struct.

**Violation:**
```cpp
void compute(const std::string& input, std::string& output);  // BAD: output parameter
```

**Compliant:**
```cpp
std::string compute(const std::string& input);  // GOOD: return value
```

---

## Part III: Readability (Rules 21–27)

### Rule 21: constexpr > const — Compile-Time Over Runtime

Prefer `constexpr` over `const` whenever the value can be computed at compile
time. Use `constexpr` functions to move computation from runtime to compile time.
`const` is acceptable when the value is only known at runtime.

**Violation:**
```cpp
const int kMaxSize = 1024;            // BAD: could be constexpr
const double kPi = 3.1415926535;     // BAD: could be constexpr
```

**Compliant:**
```cpp
constexpr int kMaxSize = 1024;        // GOOD
constexpr double kPi = 3.1415926535;  // GOOD
constexpr int square(int x) { return x * x; }  // GOOD
```

---

### Rule 22: No Magic Numbers — Named Constants or Enums

Every literal value other than 0, 1, `nullptr`, or well-known sentinels MUST
be given a named constant or enum. This includes array sizes, buffer limits,
timeouts, error codes, and configuration values.

**Violation:**
```cpp
if (retry_count > 5) { }           // BAD: what is 5?
std::array<int, 256> buffer;       // BAD: what is 256?
```

**Compliant:**
```cpp
constexpr int kMaxRetries = 5;
if (retry_count > kMaxRetries) { }  // GOOD

constexpr size_t kBufferSize = 256;
std::array<int, kBufferSize> buffer;  // GOOD
```

---

### Rule 23: No Mutable Global Variables

Mutable global (namespace-scope) variables are forbidden. Global constants
MUST be declared `constexpr` (or `const` if truly runtime-determined).
Shared state must be passed explicitly or managed through dependency injection.

**Violation:**
```cpp
int g_counter = 0;                     // BAD: mutable global
extern int g_config;                   // BAD: mutable global
```

**Compliant:**
```cpp
constexpr int kDefaultPort = 8080;     // GOOD: constexpr global
inline constexpr std::string_view kAppName = "MyApp";  // GOOD
```

---

### Rule 24: auto — Use When Type is Clear, Avoid When Ambiguous

Use `auto` when:
- The type is obvious from the initializer (e.g., `auto it = vec.begin()`).
- The exact type is complex or verbose (e.g., iterator types, lambda types).
- Using `auto&` or `const auto&` in range-for loops.

Avoid `auto` when:
- The type is not obvious and aids readability (e.g., `auto result = compute()` — what is `result`?).
- The type matters for API contract clarity.

**Violation:**
```cpp
auto result = compute_value();  // BAD: ambiguous — what type is result?
```

**Compliant:**
```cpp
auto it = container.begin();          // GOOD: obvious iterator
const auto& value = map.at(key);      // GOOD: reference is clear
int result = compute_value();         // GOOD: explicit type
```

---

### Rule 25: Prefer using Over typedef

Always use `using` alias declarations instead of `typedef`. `using` is more
readable, supports template aliases, and is the modern C++ idiom.

**Violation:**
```cpp
typedef std::map<std::string, int> NameMap;          // BAD
typedef void (*Callback)(int);                       // BAD
```

**Compliant:**
```cpp
using NameMap = std::map<std::string, int>;          // GOOD
using Callback = void(*)(int);                       // GOOD
template<typename T>
using Vec = std::vector<T>;                          // GOOD: template alias
```

---

### Rule 26: Minimal Interface — Private Members, Get/Set on Demand

All data members MUST be `private` (or `protected` only in true inheritance
hierarchies). Expose getters/setters only when external access is genuinely
needed — prefer behavioral interfaces over data accessors. Avoid trivial
get/set pairs that just expose internals.

**Violation:**
```cpp
class Person {
public:
    std::string name;  // BAD: public data member
    int age;           // BAD: public data member
};
```

**Compliant:**
```cpp
class Person {
public:
    const std::string& name() const { return name_; }  // GOOD: getter only
    void set_name(std::string name) { name_ = std::move(name); }
private:
    std::string name_;
    int age_ = 0;  // no setter needed
};
```

---

### Rule 27: Prefer std::optional Over Sentinel Values

Use `std::optional<T>` to represent "value or nothing" instead of sentinel
values like `-1`, `nullptr`, empty string, or special enums. This makes the
contract explicit and prevents bugs from forgotten sentinel checks.

**Violation:**
```cpp
int find_index(const std::vector<int>& v, int target) {
    // ... returns -1 if not found  // BAD: sentinel value
}
```

**Compliant:**
```cpp
std::optional<int> find_index(const std::vector<int>& v, int target) {
    // ... returns std::nullopt if not found  // GOOD: explicit
}
```

---

## Part IV: Performance (Rules 28–30)

### Rule 28: Range-for Without Copying — Use auto& / const auto&

In range-based for loops, always use `const auto&` (read-only) or `auto&`
(mutable) to avoid unintended copies of elements. Use `auto&&` for generic
code (templates) where the value category is unknown. Plain `auto` copies
each element and is rarely correct.

**Violation:**
```cpp
for (auto item : container) {  // BAD: copies every element
    process(item);
}
```

**Compliant:**
```cpp
for (const auto& item : container) {  // GOOD: no copy
    process(item);
}
for (auto& item : container) {  // GOOD: mutable reference
    modify(item);
}
```

---

### Rule 29: Use std::span or std::string_view for Read-Only Range Parameters

When a function needs read-only access to a contiguous sequence, prefer
`std::span<T>` (C++20) or `std::string_view` over `const std::vector<T>&`
or `const std::string&`. This avoids forcing the caller to use a specific
container type.

**Violation:**
```cpp
void process(const std::vector<int>& data);  // BAD: forces vector
void print(const std::string& s);            // BAD: forces std::string
```

**Compliant:**
```cpp
void process(std::span<const int> data);     // GOOD: accepts vector, array, etc.
void print(std::string_view s);              // GOOD: accepts string, const char*, etc.
```

---

### Rule 30: Containers Store Only Smart Pointers or Values

Standard library containers (`std::vector`, `std::map`, etc.) MUST NOT store raw
pointers that imply ownership. Store value types directly, or store
`std::unique_ptr<T>` / `std::shared_ptr<T>` when polymorphism or shared ownership
is needed.

**Violation:**
```cpp
std::vector<Widget*> widgets;  // BAD: who owns these?
```

**Compliant:**
```cpp
std::vector<Widget> widgets;                          // GOOD: value
std::vector<std::unique_ptr<Widget>> widgets;         // GOOD: unique ownership
std::vector<std::shared_ptr<Widget>> widgets;         // GOOD: shared ownership
```

---

## Compliance Checklist

When reviewing or generating C++ code under this skill, verify every rule:

### Safety (Rules 1–10)

| # | Rule | Check |
|---|------|-------|
| 1 | No bare new/delete/malloc/free | RAII wrappers for all resources |
| 2 | No raw owning pointers | All `new`-ed objects go into smart pointers |
| 3 | RAII members in destructors | No manual cleanup in ~T() |
| 4 | No dangling refs/pointers | Lifetime analysis passes |
| 5 | Return locals by value, not reference | No dangling references to locals |
| 6 | No raw arrays | std::array / std::vector only |
| 7 | No C string functions | std::string / std::format |
| 8 | No void* or C varargs | Type-safe alternatives |
| 9 | Always initialize variables | Brace or value initialization |
| 10 | Synchronized shared state | mutex or atomic for shared data |

### Correctness (Rules 11–20)

| # | Rule | Check |
|---|------|-------|
| 11 | Named casts only | No C-style casts |
| 12 | nullptr over 0/NULL | Type-safe null |
| 13 | enum class over plain enum | Scoped, type-safe |
| 14 | Virtual destructor for polymorphic bases | virtual ~Base() present |
| 15 | Rule of Five consistency | All 5 declared or all defaulted |
| 16 | No object slicing | Polymorphic by ref/pointer |
| 17 | Exceptions handled or noexcept | No silent swallowing |
| 18 | Explicit lambda capture | No [=] or [&] |
| 19 | explicit single-arg constructors | No accidental implicit conversion |
| 20 | Function parameter conventions | In by const&, out by return |

### Readability (Rules 21–27)

| # | Rule | Check |
|---|------|-------|
| 21 | constexpr over const | Compile-time where possible |
| 22 | No magic numbers | All literals named |
| 23 | No mutable globals | Global state is constexpr |
| 24 | Judicious auto usage | Type clear from context |
| 25 | using over typedef | Modern alias syntax |
| 26 | Minimal interface, private data | All members private |
| 27 | std::optional over sentinels | Explicit "maybe" semantics |

### Performance (Rules 28–30)

| # | Rule | Check |
|---|------|-------|
| 28 | Range-for with refs | const auto& or auto& |
| 29 | std::span / string_view for ranges | Container-agnostic interfaces |
| 30 | Containers: smart pointers or values | No raw owning pointers in containers |

## Enforcement Stance

When this skill is active, violations of any of the 30 rules are **blocking**.
Do not produce code that violates any rule. If an existing codebase contains
violations, flag them explicitly with the rule number and suggest the compliant
alternative. Do not silently "fix" violations — always explain what rule is
violated and how to fix it.
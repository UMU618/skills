# Google C++ Style Guide Reference

## Background

C++ is one of the main development languages used by many of Google's open-source projects. This guide provides the coding conventions for writing C++ code that follows Google's standards.

## Goals of the Style Guide

- **Style rules should pull their weight**: Rules must have enough benefit to justify engineers remembering them.
- **Optimize for the reader, not the writer**: Code is read more often than it is written.
- **Be consistent with the broader C++ community**: Follow standard practices when reasonable.
- **Avoid surprising or dangerous constructs**: Safety and clarity first.
- **Scale to Google's codebase**: Rules must work at massive scale.

## Header Files

### Self-contained Headers
- Header files should be self-contained (compile on their own).
- End header file guards with `_H_` suffix: `#ifndef FOO_BAR_BAZ_H_`
- All headers should have `#define` guard to prevent multiple inclusion.
- Prefer `#ifndef`/`#define`/`#endif` over `#pragma once`.

### The #define Guard
```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
// ... contents ...
#endif  // FOO_BAR_BAZ_H_
```

### Include What You Use
- If a source or header file refers to a symbol defined elsewhere, include the header that provides the declaration.
- Do not rely on transitive includes.
- Include headers in the following order:
  1. Related header (the .h for the current .cc)
  2. C system headers
  3. C++ standard library headers
  4. Other libraries' headers
  5. Your project's headers

### Forward Declarations
- Avoid using forward declarations where possible. Prefer `#include`.

### Inline Functions
- Define inline functions only when they are small (10 lines or fewer).
- Do not define functions in header files unless they are:
  - Inline functions
  - Template functions
  - constexpr functions

### Names and Order of Includes
```cpp
#include "foo/server/fooserver.h"  // Related header

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "base/basictypes.h"
#include "foo/server/bar.h"
```

## Scoping

### Namespaces
- **Use namespaces with care.** Almost all code should be in a namespace.
- Use unnamed namespaces in `.cc` files (preferred over `static`).
- Never use `using namespace` in header files at global scope.
- Use `using` declarations sparingly in `.cc` files.
- Namespace names are all lowercase with underscores, based on project and path.

```cpp
namespace my_project {
namespace internal {
// ...
}  // namespace internal
}  // namespace my_project
```

### Unnamed Namespaces and Static Variables
- Use unnamed namespaces for internal linkage in `.cc` files.
- Do not use `static` for file-scope functions or variables in `.cc` files.

```cpp
namespace {
// This is only visible in this translation unit.
const int kMaxBufferSize = 4096;
}  // namespace
```

### Nonmember, Static Member, and Global Functions
- Prefer nonmember functions in a namespace over static member functions.
- Avoid bare global functions; place them in a namespace.

### Local Variables
- Place variables in the narrowest scope possible.
- Initialize variables at declaration when possible.

### Static and Global Variables
- Variables of class type with static storage duration are forbidden unless trivially destructible.
- Use `constexpr` or `const` for global constants.

## Classes

### Doing Work in Constructors
- Avoid virtual method calls in constructors.
- Avoid complex initialization that could fail.
- Use `Init()` methods for complex initialization when necessary.

### Implicit Conversions
- Use `explicit` on single-argument constructors and conversion operators.
- Copy and move constructors should not be explicit.

```cpp
class MyClass {
 public:
  explicit MyClass(int value);  // GOOD
  explicit MyClass(std::string name);  // GOOD
};
```

### Copyable and Movable Types
- A class's public API must make clear whether the class is copyable, move-only, or neither.
- Declare copy/move operations explicitly or delete them.

```cpp
class Copyable {
 public:
  Copyable(const Copyable&) = default;
  Copyable& operator=(const Copyable&) = default;
};

class MoveOnly {
 public:
  MoveOnly(MoveOnly&&) = default;
  MoveOnly& operator=(MoveOnly&&) = default;
  MoveOnly(const MoveOnly&) = delete;
  MoveOnly& operator=(const MoveOnly&) = delete;
};
```

### Structs vs. Classes
- Use `struct` only for passive objects that carry data (POD types).
- Use `class` for everything else.
- In structs, all fields are public by convention.

### Inheritance
- Composition is often more appropriate than inheritance.
- When using inheritance, make it `public`.
- If a class has virtual methods, the destructor should be virtual.
- Limit use of `protected` to member functions; data members should be `private`.
- Use `override` keyword explicitly for all overridden virtual functions.
- Use `final` only when strictly necessary.

### Operator Overloading
- Overload operators judiciously.
- Do not overload `&&`, `||`, `,` (comma), or unary `&`.
- Define overloaded operators only if their meaning is obvious.

### Access Control
- Make data members `private`, unless they are `static const` (or `constexpr`).
- For technical reasons, allow `protected` in test fixtures.

### Declaration Order
Group declarations in this order:
1. Types and type aliases (`typedef`, `using`, `enum`, nested structs/classes)
2. Static constants
3. Factory functions
4. Constructors and assignment operators
5. Destructor
6. All other functions (static and instance member functions, and friend functions)
7. Data members (static and non-static)

Access sections in order: `public:`, `protected:`, `private:`.

## Functions

### Inputs and Outputs
- Output parameters should be pointers (not references) when the parameter can be null.
- Prefer return values over output parameters.
- Use `const` references for input parameters that are not modified.

### Write Short Functions
- Prefer small, focused functions.
- If a function exceeds ~40 lines, consider breaking it up.

### Function Overloading
- Use overloaded functions only if the reader can tell which overload is called without looking at the definition.

### Default Arguments
- Default arguments are allowed on non-virtual functions.
- Must always be `const` values or `constexpr`.
- Avoid default arguments in virtual functions.

### Trailing Return Type Syntax
- Use trailing return types only when the leading return type is impractical (e.g., lambdas, templates with complex return types).

```cpp
template <typename T, typename U>
auto Add(T t, U u) -> decltype(t + u);
```

## Other C++ Features

### Rvalue References and Move Semantics
- Use rvalue references to define move constructors and move assignment operators.
- Use `std::move` only when necessary for move semantics.

### Friends
- Use friend classes and functions only within reasonable bounds.
- Friend is typically used for testing or tightly coupled classes.

### Exceptions
- **Google C++ Style: Do not use C++ exceptions.**
- Code should be exception-free.
- Use error codes, `absl::Status`, or `absl::StatusOr` for error handling.

### noexcept
- Use `noexcept` for move constructors and move assignment operators when possible.
- Do not use `noexcept` specifiers on other functions unless required.

### Run-Time Type Information (RTTI)
- **Avoid RTTI.** Do not use `typeid` or `dynamic_cast`.
- Use virtual methods or tagged unions instead.

### Casting
- Use C++-style casts: `static_cast`, `const_cast`, `reinterpret_cast`.
- Avoid C-style casts `(type)value`.
- `dynamic_cast` is not allowed (no RTTI).

```cpp
int value = static_cast<int>(double_value);
```

### Streams
- Use streams only for logging. Prefer `absl::StrCat()`, `absl::StrFormat()` for string building.
- Do not use streams for serialization or file I/O in production code.

### Preincrement and Predecrement
- Use prefix form (`++i`) for iterators and other template types.
- Postfix (`i++`) is acceptable for simple value types when the result is needed.

### Use of const
- Use `const` wherever it makes sense.
- Use `const` on methods that do not modify the object.
- Prefer `constexpr` for true constants where possible.
- Use `const` for variables that should not be modified.

```cpp
const int kDaysInWeek = 7;
constexpr int kBufferSize = 1024;

void MyMethod() const;
```

### constexpr
- Use `constexpr` for true constants and functions that can be evaluated at compile time.

### Integer Types
- Use `int` for most integers unless a specific size is needed.
- Use `<cstdint>` types (`int64_t`, `uint32_t`, etc.) when specific sizes matter.
- Use `size_t` for sizes and indices.
- Avoid unsigned types except when representing bit patterns or modular arithmetic.

### 64-bit Portability
- Code should be 64-bit and 32-bit friendly.

### Preprocessor Macros
- Avoid macros. Prefer inline functions, enums, and `const` variables.
- If you must use a macro, name it with all caps and underscores: `MY_MACRO()`.

### 0, nullptr, and NULL
- Use `nullptr` for pointers.
- Use `0` for integers, `0.0` for doubles.
- Never use `NULL`.

### sizeof
- Prefer `sizeof(varname)` over `sizeof(type)`.

### auto
- Use `auto` to avoid type names that are noisy or obvious.
- Do not use `auto` when the type is not clear to the reader.
- Use `auto` with iterators and complex template types.

```cpp
auto it = my_map.find(key);  // GOOD: type is obvious from context
auto result = CalculateSomething();  // BAD: type is unclear
```

### Braced Initializer List
- Use braced initialization `{}` for initialization (not for assignment).
- Never use braced initialization with `auto`.

```cpp
std::vector<int> v{1, 2, 3};  // GOOD
int x{5};  // GOOD
auto x{5};  // BAD: x is std::initializer_list<int>
```

### Lambda Expressions
- Use lambdas where appropriate, with default capture `[=]` or `[&]`.
- Prefer explicit captures when the lambda might outlive the current scope.
- Keep lambdas short and self-contained.
- Use trailing return types only if needed for clarity.

```cpp
std::sort(v.begin(), v.end(), [](int a, int b) {
  return a > b;
});
```

### Template Metaprogramming
- Use template metaprogramming only in small, focused doses.
- Avoid complex template metaprogramming that is hard to read.

### C++14 and C++17 Features
- Use modern C++ features where appropriate.
- Prefer `std::make_unique` over `new`.
- Use structured bindings where they improve readability.
- Use `if`/`switch` initializers (C++17) where useful.

### std::hash
- Do not specialize `std::hash` for user-defined types unless the type is designed to be used as a hash key.

## Naming

### General Naming Rules
- Names should be descriptive and avoid abbreviation.
- Give as much context as makes sense.

### File Names
- Filenames should be all lowercase and can include underscores `_` or dashes `-`.
- C++ source files: `.cc`
- C++ header files: `.h`
- Inline template implementation: `-inl.h` (or `_impl.h`)

### Type Names (Classes, Structs, Type Aliases, Enums, Type Template Parameters)
- PascalCase with no underscores.
- All types start with a capital letter and have a capital letter for each new word.

```cpp
class UrlTable { ... };
class UrlTableTester { ... };
struct UrlTableProperties { ... };
using PropertiesMap = hash_map<UrlTableProperties, std::string>;
enum class UrlTableError { ... };
```

### Variable Names
- **snake_case** for local variables and data members.
- Data members of classes: snake_case with trailing underscore `_`.

```cpp
std::string table_name;  // local variable
class TableInfo {
 private:
  std::string table_name_;  // member variable
  static int pool_count_;   // static member
};
```

### Struct Data Members
- Data members of structs: snake_case without trailing underscore.

```cpp
struct TableProperties {
  std::string name;
  int num_entries;
};
```

### Constant Names
- `k` prefix followed by PascalCase: `kDaysInWeek`, `kMaxBufferSize`.
- Use `constexpr` for compile-time constants.

```cpp
const int kDaysInWeek = 7;
constexpr int kMaxBufferSize = 4096;
```

### Function Names
- PascalCase with a capital first letter.
- Usually verb-like: `AddTableEntry()`, `DeleteUrl()`, `OpenFileOrDie()`.

```cpp
void AddTableEntry();
void DeleteUrl();
int CountErrors();
```

### Namespace Names
- All lowercase with underscores.
- Top-level namespace based on project name.
- Avoid nested namespaces that match well-known top-level namespaces.

### Enumerator Names
- For unscoped enums: `kEnumName` style like constants.
- For scoped enums (`enum class`): PascalCase (since they are types).

```cpp
enum class UrlTableError {
  kOk = 0,
  kOutOfMemory,
  kMalformedInput,
};
```

### Macro Names
- All caps with underscores: `MY_MACRO()`.
- Avoid macros whenever possible.

```cpp
#define ROUND(x) ...
#define PI_ROUNDED 3.0
```

### Exceptions to Naming Rules
- If you are naming something analogous to an existing C or C++ entity, follow its naming convention.
  - `bigopen()` - function name, follows form of `open()`
  - `uint` - typedef
  - `sparse_hash_map` - STL-like entity

## Comments

### Comment Style
- Use `//` for most comments, even multi-line.
- Use `/* */` only for large blocks that need special formatting.

### File Comments
- Every file should start with a license boilerplate.
- Include a file-level comment describing the contents.

### Class Comments
- Every non-obvious class declaration should have a comment describing what it is for and how it should be used.

```cpp
// Iterates over the contents of a GargantuanTable.
// Example:
//    std::unique_ptr<GargantuanTableIterator> iter = table->NewIterator();
//    for (iter->Seek("foo"); !iter->done(); iter->Next()) {
//      process(iter->key(), iter->value());
//    }
class GargantuanTableIterator {
  ...
};
```

### Function Comments
- Declaration comments describe use of the function.
- Definition comments describe operation.
- Document: inputs, outputs, thread-safety, ownership, performance implications.
- Use `@param`, `@return`, etc. for Doxygen-style comments when appropriate.

```cpp
// Returns an iterator for this table, positioned at the first entry
// lexically greater than or equal to `start_word`. If there is no
// such entry, returns nullptr. The client must not use the iterator
// after the underlying GargantuanTable has been destroyed.
//
// @param start_word The starting word for the search.
// @return An iterator or nullptr.
std::unique_ptr<Iterator> NewIterator(std::string_view start_word);
```

### Variable Comments
- Comment the purpose of non-obvious data members.

### Implementation Comments
- Explain tricky, non-obvious, interesting, or important parts of the code.
- Use `TODO(username)`: for temporary, short-term solutions.

```cpp
// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke): Change this to use relations.
```

### Punctuation, Spelling, and Grammar
- Pay attention to punctuation, spelling, and grammar.
- Write complete sentences with proper capitalization.

## Formatting

### Line Length
- Maximum 80 characters per line.
- Exceptions: comments with example commands or URLs, raw string literals.

### Non-ASCII Characters
- Use UTF-8 for source files.
- Non-ASCII characters should be rare; use Unicode escapes when needed.

### Spaces vs. Tabs
- Use only spaces, indent 2 spaces at a time.
- **Never use tabs.**

### Function Declarations and Definitions
- Return type on the same line as function name.
- Parameters on the same line if they fit; otherwise, wrap with each parameter on its own line with 4-space indent.
- Opening brace always on the same line as the last parameter.

```cpp
ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
  ...
}
```

### Lambda Expressions
- Format parameters and bodies like any other function.
- Use braces for the body.

```cpp
std::sort(v.begin(), v.end(), [](int a, int b) -> bool {
  return a > b;
});
```

### Function Calls
- Write the call all on one line if it fits.
- If not, wrap arguments at the parenthesis with each on its own line.

```cpp
bool retval = DoSomething(argument1, argument2, argument3);

bool retval = DoSomething(
    argument1, argument2,
    argument3, argument4);
```

### Braced Initializer List Format
- Format like function calls.

### Conditionals
- Always use curly braces for `if`, `else`, `for`, `while` blocks.
- No space inside parentheses.
- `if` and `else` keywords on separate lines.

```cpp
if (condition) {
  DoSomething();
} else {
  DoSomethingElse();
}
```

### Loops and Switch Statements
- `switch` statements always have a `default` case.
- Fall-through must be annotated with `[[fallthrough]];` (C++17) or a comment.

```cpp
switch (var) {
  case 0: {
    DoSomething();
    break;
  }
  case 1: {
    [[fallthrough]];
  }
  default: {
    DoDefault();
    break;
  }
}
```

### Pointer and Reference Expressions
- No space around `.` or `->`.
- No space after `*` or `&` in declarations.
- When declaring a pointer variable or argument, place the asterisk adjacent to the type.

```cpp
// GOOD:
std::string* ptr;
const std::string& ref;

// BAD:
std::string *ptr;
const std::string &ref;
```

### Boolean Expressions
- Break long boolean expressions at logical operators.
- Extra parentheses for clarity are acceptable.

### Return Values
- Do not use parentheses around return values unless necessary.

```cpp
return result;            // GOOD
return (some_condition);  // Unnecessary parentheses
```

### Variable and Array Initialization
- Use `=`, `()`, or `{}` for initialization as appropriate.

```cpp
int x = 3;
std::string name("Some Name");
std::string name = "Some Name";
```

### Preprocessor Directives
- Preprocessor directives should not be indented.
- Start at the beginning of the line, even within indented code.

```cpp
void MyFunction() {
  if (condition) {
#if DEBUG
    DoDebugThing();
#endif
    DoOtherThing();
  }
}
```

### Class Format
- Access specifiers (`public:`, `protected:`, `private:`) indented 1 space.
- Declarations within each section indented 2 spaces.

```cpp
class MyClass : public OtherClass {
 public:
  MyClass();
  ~MyClass() override;

  void DoSomething();

 private:
  int value_;
};
```

### Constructor Initializer Lists
- All on one line if they fit, or each on its own line indented 4 spaces.

```cpp
MyClass::MyClass(int var) : some_var_(var), other_var_(var + 1) {}

MyClass::MyClass(int var)
    : some_var_(var),
      some_other_var_(var + 1),
      yet_another_var_(var + 2) {}
```

### Namespace Formatting
- Contents of namespaces are not indented.

```cpp
namespace my_namespace {

void MyFunction() {  // NOT indented
  ...
}

}  // namespace my_namespace
```

### Horizontal Whitespace
- Never put trailing whitespace.
- Use one space after keywords (`if`, `for`, `while`, `switch`).
- No space before semicolons or commas.
- One space before `{` on same line.

### Vertical Whitespace
- Minimize blank lines.
- Use at most one blank line to separate logical groups.
- The coding standard says: "Use your best judgment."

## Exceptions to the Rules

- Existing non-conforming code: If you are modifying existing code, follow its existing style.
- Windows code: Some adaptations are acceptable for Windows-specific code.

## Parting Words

- **Be consistent.** If you're editing code, match the style of the surrounding code.
- The goal is to have code that is clear, consistent, and maintainable.

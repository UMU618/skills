---
name: google-cpp-style
description: Enforce Google C++ Style Guide when writing, reviewing, or refactoring C++ code. Covers naming conventions, formatting rules, header file guidelines, scoping, class design, function best practices, modern C++ feature usage, and comment standards. Trigger when writing C++ code, reviewing C++ style, or when Google C++ Style Guide is mentioned.
---

# Google C++ Style Guide

## Overview

Enforce the Google C++ Style Guide when writing, reviewing, or refactoring C++ code. This skill ensures all generated C++ code follows Google's conventions, covering naming, formatting, header guards, scoping, class design, modern C++ features, comments, and more.

## When to Use This Skill

Trigger this skill whenever the user:
- Asks to write or generate C++ code
- Requests a code review for C++ code
- Asks to refactor C++ code to match a style guide
- Mentions "Google C++ Style", "Google style", "Google coding standard", or similar

## How to Apply This Skill

### Step 1: Load the Reference

When C++ code needs to be written or reviewed, load the full style guide reference:

```
references/style-guide.md
```

This contains the complete Google C++ Style Guide covering all rules and conventions.

### Step 2: Apply Key Rules by Category

When generating or reviewing C++ code, verify compliance in these areas in order of priority:

#### Naming (Most Visible)
- **Files**: lowercase with underscores, `.cc` / `.h` extensions
- **Types** (classes, structs, enums, type aliases): PascalCase
- **Variables** (local, members): snake_case; member variables: trailing `_`
- **Functions**: PascalCase (verb-like)
- **Constants**: `k` prefix + PascalCase (e.g., `kMaxSize`)
- **Namespaces**: lowercase with underscores
- **Macros**: ALL_CAPS with underscores (avoid macros when possible)
- **Enumerators**: `kEnumName` for unscoped enums; PascalCase for `enum class`

#### Formatting
- 2-space indent, no tabs
- 80 character line limit
- Opening brace on same line as function/class/conditional
- Always use braces for `if`/`for`/`while` (no single-line bodies without braces)
- Pointer/reference: `Type* ptr`, `const Type& ref` (asterisk/ampersand adjacent to type)
- Namespace contents not indented
- Access specifiers indented 1 space, members indented 2 spaces

#### Header Files
- Use `#ifndef`/`#define`/`#endif` guards with `_H_` suffix
- Headers must be self-contained
- Include what you use; no transitive includes
- Avoid forward declarations; prefer `#include`

#### Classes
- Use `explicit` on single-argument constructors
- Declare copy/move operations explicitly (or delete them)
- Use `struct` only for passive data objects (POD)
- Use `class` for everything else
- Make data members `private` with accessors if needed
- Use `override` for all overridden virtual methods
- Use `final` sparingly

#### Functions
- Prefer return values over output parameters
- Use `const&` for input parameters
- Keep functions short (~40 lines max)
- Do not use default arguments in virtual functions

#### Modern C++ Features
- **No exceptions**: Use error codes, `absl::Status`, or `StatusOr`
- **No RTTI**: Do not use `typeid` or `dynamic_cast`
- Use `nullptr`, not `NULL` or `0` for pointers
- Use `auto` only when type is obvious from context
- Use braced initialization `{}` (but never `auto x{5}`)
- Use C++ casts: `static_cast`, `const_cast`, `reinterpret_cast` (no C-style)
- Use `std::make_unique` over `new`

#### Comments
- Use `//` for most comments (even multi-line)
- Every file starts with license boilerplate
- Document classes and non-obvious functions
- Use `TODO(username):` for temporary code

#### Class Declaration Order
```
public:
  - types/aliases
  - static constants
  - factory functions
  - constructors / assignment operators
  - destructor
  - other functions
protected:
  - (same order)
private:
  - (same order)
  - data members last
```

### Step 3: Review Against Complete Checklist

After writing code, perform a final review checking:
1. Naming conventions match (types, variables, functions, constants, files)
2. Formatting is correct (indentation, braces, line length, whitespace)
3. Headers have proper guards and includes
4. No exceptions, no RTTI, no C-style casts
5. Classes have proper access control and member ordering
6. Functions are short, well-named, and properly documented
7. Comments are clear and complete
8. `const` used where appropriate
9. `explicit` on single-argument constructors
10. `override` on all virtual overrides

## Resources

### references/style-guide.md
Complete Google C++ Style Guide reference with all rules, examples, and formatting conventions. Load this when detailed style guidance is needed.

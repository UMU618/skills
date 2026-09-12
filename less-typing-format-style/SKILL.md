---
name: less-typing-format-style
description: >
  Apply the "less typing" C++ formatting style — minimize keystrokes wherever the
  information content is identical. Use when formatting, writing, or reviewing C++
  code, when generating or editing a .clang-format file, or when the user mentions
  "less typing", "敲击少", "格式化风格", "format style", "IndentWidth", "PointerAlignment",
  "BraceWrapping", "BreakAfterReturnType", or "AlignEscapedNewlines". Derives from the
  Google/Chromium style family and enforces 2-space indent, left-aligned pointers,
  and minimal line breaks.
---

# Less Typing Format Style

## Overview

This skill encodes a formatting principle for C++: **minimize keystrokes whenever the
information content is identical**. It is derived from the blog post
《"敲击少"格式化风格》 and filters the well-known C++ styles (LLVM, Google, Chromium,
GNU, Mozilla, Microsoft, WebKit) down to the ones that survive that principle —
**Google** and **Chromium**.

Scope: this is the **lowest-level** topic in coding style — **format only**. It does
**not** govern naming (variables, functions, etc.). Naming should follow readability
first and must not be abbreviated for the sake of typing less.

## When to Use This Skill

Trigger this skill whenever the user:
- Writes, formats, or reviews C++ code and asks for a style decision
- Creates or edits a `.clang-format` file
- Asks to reduce typing/keystrokes in code formatting
- Mentions "less typing", "敲击少", "格式化风格", "format style", or any of the
  specific options below (`PointerAlignment`, `BraceWrapping`, `BreakAfterReturnType`,
  `AlignEscapedNewlines`, `IndentWidth`)

## The Principle

> **Less typing** — but only when the information content is exactly the same.

Two goals motivate it:
1. Protect finger health by typing less.
2. Cut decision fatigue by always resolving "this is fine, that is fine too" the
   same way.

The principle is **only** applied when two options carry identical information. It
never justifies cryptic names, omitted braces, or unreadable code.

## Rules

### Rule 1: Indent with 2 spaces

2 spaces are visually sufficient. 2-space indentation is used by LLVM, Google,
Chromium, GNU, and Mozilla — 4-space by only Microsoft and WebKit. 2 spaces is the
mainstream, and it types fewer characters.

```
IndentWidth: 2
```

### Rule 2: Left-align to save spaces

#### 2.1 Pointer and reference alignment

Four possible styles exist:

- Left: `int* p;`
- Right: `int *p;`
- Middle: `int * p;`
- `int*p;`

Middle and `int*p` are eliminated immediately (no well-known style uses them, and
ClangFormat has no option for `int*p`). Between Left and Right, the deciding case is
a cast:

```cpp
void* p{};
auto i = static_cast<int*>(p);
```

Left alignment formats as:

```cpp
void* p{};
auto i = static_cast<int*>(p);
```

Right alignment formats as:

```cpp
void *p{};
auto i = static_cast<int *>(p);
```

Right alignment adds an extra space in the cast case, violating "less typing".
Therefore choose **Left**.

Reference alignment must match pointer alignment. Also set
`DerivePointerAlignment: false` — if true, ClangFormat infers alignment from the
file's existing usage and uses `PointerAlignment` only as a fallback, which makes
formatting non-deterministic.

```
DerivePointerAlignment: false
PointerAlignment: Left
ReferenceAlignment: Pointer
```

The common objection — `int* p, q;` looks like `q` is also a pointer — is weak:
it only fools beginners, and defining multiple variables per line is discouraged
anyway.

#### 2.2 Align escaped newlines to the left

- `DontAlign`: ugly
- `Left`: as far left as possible — fewer spaces
- `Right`: as far right as possible — prettier but more spaces

Left is used by Google and Chromium. Choose `Left` to save spaces.

```
AlignEscapedNewlines: Left
```

### Rule 3: Break lines as little as possible without hurting readability

Blank lines and comments between statements improve readability and should stay.
But some line breaks do not affect readability at all — those should go.

#### 3.1 BraceWrapping

Indentation already guarantees readability; whether `{` lines up vertically with
`}` no longer matters (short lines matter more). Set as many BraceWrapping options
as possible to `false` / `Never` — Google and Chromium do exactly this:

```
BraceWrapping:
  AfterCaseLabel:  false
  AfterClass:      false
  AfterControlStatement: Never
  AfterEnum:       false
  AfterExternBlock: false
  AfterFunction:   false
  AfterNamespace:  false
  AfterObjCDeclaration: false
  AfterStruct:     false
  AfterUnion:      false
  BeforeCatch:     false
  BeforeElse:      false
  BeforeLambdaBody: false
  BeforeWhile:     false
  IndentBraces:    false
  SplitEmptyFunction: true
  SplitEmptyRecord: true
  SplitEmptyNamespace: true
```

#### 3.2 BreakAfterReturnType

- `None`: automatic
- `All`: always break after the return type (definitions and declarations)
- `TopLevel`: always break after the return type of top-level functions
- `AllDefinitions`: always break after the return type of definitions
- `TopLevelDefinitions`: always break after the return type of top-level definitions

Mainstream styles use `None` (LLVM, Google, Chromium, Microsoft, WebKit). Only
Mozilla (`TopLevel`) and GNU (`AllDefinitions`) differ. Choose `None`.

```
BreakAfterReturnType: None
```

### Rule 4: Initialize with `{}`, and omit default values

Prefer brace initialization `{}` over `= value`. If the value inside the braces is
the type's **default value**, omit it — `{}` alone already produces it. For a class
that has a default constructor, the default constructor already yields the default
value, so no braces and no explicit default value are needed at all.

```cpp
bool is_ok = false;                  // long
bool is_ok{};                        // shorter: value inside {} is the default, omit it

std::shared_ptr<int> p = nullptr;    // long
std::shared_ptr<int> p;              // shorter: default ctor already gives nullptr

int count = 0;                       // long
int count{};                         // shorter
```

Summary:

| Type | Preferred form | Reason |
|------|----------------|--------|
| Built-in (`bool`, `int`, pointer, ...) | `T x{};` | `{}` is required to avoid an uninitialized value |
| Class with a default constructor | `T x;` | The default constructor already initializes it — braces are redundant |
| Non-default initial value | `T x{value};` | The value is real information, so it must be kept |

This rule stays compatible with the "always initialize" rule: `bool is_ok{};`
value-initializes, and `std::shared_ptr<int> p;` default-constructs. Never shorten
`bool is_ok{};` to `bool is_ok;`, which would leave the value indeterminate.

## Reference .clang-format

The full settings required by this skill (all other options may follow Google /
Chromium defaults):

```yaml
Language: Cpp
BasedOnStyle: Google

IndentWidth: 2

DerivePointerAlignment: false
PointerAlignment: Left
ReferenceAlignment: Pointer

AlignEscapedNewlines: Left

BreakAfterReturnType: None

BraceWrapping:
  AfterCaseLabel:  false
  AfterClass:      false
  AfterControlStatement: Never
  AfterEnum:       false
  AfterExternBlock: false
  AfterFunction:   false
  AfterNamespace:  false
  AfterObjCDeclaration: false
  AfterStruct:     false
  AfterUnion:      false
  BeforeCatch:     false
  BeforeElse:      false
  BeforeLambdaBody: false
  BeforeWhile:     false
  IndentBraces:    false
  SplitEmptyFunction: true
  SplitEmptyRecord: true
  SplitEmptyNamespace: true
```

## Conclusion

Under the "less typing" principle, only **Google** and **Chromium** survive among
the well-known styles. Prefer this style family when making formatting decisions.

## Extensions

1. **Pointer null checks**: omit the comparison with `nullptr` when the pointer is
   used as a boolean (e.g. write `if (p)` instead of `if (p != nullptr)`). This is
   consistent with the "less typing" principle.
2. **Line endings**: use `LF`, not `CRLF`. In UTF-8, each line saves one byte, which
   is "less" from the machine's point of view. Configure Git accordingly
   (`core.autocrlf` / `.gitattributes`).

## Applying This Skill

When formatting or reviewing C++ code, verify:
1. Indentation is 2 spaces, no tabs.
2. Pointers/references are left-aligned: `Type* p`, `Type& r`.
3. No unnecessary line breaks before `{` or after the return type.
4. Escaped newlines align left.
5. Initialization uses `{}` with default values omitted; classes with a default
   constructor are declared without braces.
6. Any `.clang-format` produced contains the settings in **Reference .clang-format**.

Formatting choices are only adjusted when the information content is unchanged.
Never trade away naming clarity or readability to type fewer characters.

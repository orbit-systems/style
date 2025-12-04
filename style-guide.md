# Orbit Systems C Language Style Guide

## C Standard

All official Orbit Systems projects written in C should use the **C23 standard**. No, you can't change my mind.

### GNU Extensions

GNU Extensions are allowed, as long as they work both in GCC and Clang.

## Headers

### Ifdef Guards

Use `#ifdef` guards with the form:
```
#ifndef HEADERNAME_H
#define HEADERNAME_H

// ...

#endif // HEADERNAME_H
```

Do not use `#pragma once`.

### Only include something if you use it

Only include a header if you need some functionality from that header.

## Functions

### Use descriptive names for functions, with module information

Always use a descriptive name for a function, like `fe_chain_append_begin`, rather than `chain_begin`.

Include locality information about where this function comes from. If it comes from a project (like Iron), follow the module naming convention, so `fe_chain_append_begin`, rather than `chain_append_begin`.

### Always _always_ include parameter names!

It is not 1983 now. Please include parameter names in your functions!

`void some_function(u8 arg1, void* arg2)` rather than `void some_function(u8, void*)`. This also holds true for function pointers, even if functions pointed to by that pointer do not follow the same naming convention.

### Signatures should match everywhere

Signatures should always match across all c files and headers.

## Macros (don't be scared!)

### Common conventions

Macros should always follow `SCREAMING_CASE`, and should only be used for ease-of-maintainability (e.g X-macros), and simple syntax modification (like vec for). Do not add a hard dependency on your own version of Metalang99.

### X-macros

When using the X macro idiom, do not just define it as "X", give it a descriptive name.

```c
#define OPCODE_LIST \
    OPCODE(ABC, "123") \
    OPCODE(DEF, "456")

char* opcode_mapping[] = {
    #define OPCODE(name, value) [name] = value,
        OPCODE_LIST
    #undef OPCODE
}
```

## Control Flow

### Indentation and style
Indentation should use 4 spaces per level.

Control flow statements should always have their body enclosed in a statement block, with K&R-style bracketing.
```c
// yes!
if (x == y) {
    return 1;
}

// nope
if (x == y)
{
    return 1;
}

// no
if (x == y)
    return 1;

// please no
if (x == y) return 1;
```

`case`s should match the level of their enclosing `switch`,
```c
switch (x) {
case 1:
    x = 100;
    break;
case 2:
    return bar;
case 3:
    x = 200
    [[fallthrough]];
case 4:
    break;

}
```

### Fallthrough

Fallthrough should not be implicit on switch statement cases.

```c
//bad
switch (x) {
case 1:
    x = 100;
    break;
case 2:
    return bar;
case 3:
    x = 200
case 4:
    break;
}

//good
switch (x) {
case 1:
    x = 100;
    break;
case 2:
    return bar;
case 3:
    x = 200
    [[fallthrough]];
case 4:
    break;
}
```



### Layout and complexity
If a large condition is needed, factor it out into a function or indent it like so:
```c
// yes!
if (xxxxxxxxxxx &&
    yyyyyyyyy &&
    zzzzzzzzz
) {
    return 1;
}

// no, hard to tell where the statement block starts
if (xxxxxxxxxxx &&
    yyyyyyyyy &&
    zzzzzzzzz) {
    return 1;
}
```

### Verbosity
When checking for null or for zero inside a condition, always explicitly check, rather than relying on C 'truthiness':
```c
// if (ptr) becomes:
if (ptr != nullptr) {
    return 1;
}

// if (!ch) becomes:
if (ch != '\0') {
    return 1;
}
```

...*unless* a variable is defined in a switch case, then enclose the case in a block with the final `break` outside of the block, and
indent cases one level above the enclosing `switch`.
The closing brace of the block should be in the same level of the `case` label.

If a case returns at the end of the block, the break should be omitted and the return statement should be inside the block.

```c
switch (x) {
    case 1: {
        int y = 100;
    } break;
    case 1: {
        int y = 100;
        return y;
    }
    case 2:
        return bar;
}
```

## Spelling
- Use `snake_case` for variables, functions, and structure fields.
- Use `TitleCase` for new types.
- Use `SCREAMING_SNAKE_CASE` for enum variants and `#define`'d constants.
- Macros should also use `SCREAMING_SNAKE_CASE`, but some exceptions are made for keyword mods like `for_n` that supplement language features, or inline function spoofing.

## Comments
Prefer line comments (`// this`) in almost every case, except when a large
multi-line description is needed.

## Types
Use `common/type.h` types whenever possible, especially in user-defined types.
```rs
i8, i16, i32, i64, isize // integers
u8, u16, u32, u64, usize // unsigned integers
f32, f64 // floating point
```
when the size of an integer doesn't matter, prefer `isize` and `usize` over `int`.

### Strings
For storing or passing strings (unless needed by a stdlib function), always prefer storing or passing a length along with it instead 
of implicitly relying on it being null-terminated.
Use the `string` type from `common/str.h` whenever possible, so that you do not need to remember this.

### Structs/Unions
When declaring a struct or union type, use typedef and include the name in the definition twice:
```c
typedef struct MyStruct {
    // ...
} MyStruct;
```

If it's necessary to refer to that type before the definition is available/complete, use a predeclaration and a simple declarations after:
```c
typedef struct MyStruct MyStruct;

// ...

struct MyStruct {
    // ...
};
```

### Enums
**Always** use `typedef` when creating a new enum type, and **always** give the enum an explicit backing type.
```c
typedef enum : u8 {
    TOKEN_PLUS,
    TOKEN_MINUS,
    TOKEN_NUMBER,
} TokenKind;

typedef struct Token {
    TokenKind kind;
} Token;
```

### Pointers
Pointer types, in declarations and parameters, should have the `*` attached to the base type:
```c
int* xptr = &x;

void modify(int* ptr);
```

## Declarations & Assignments

### Multiple declarations
**Never** declare or assign more than one variable in a single statement.
```c
// nuh uh
int x, y, z = 1;

// better
int x;
int y;
int z = 1;
```

### Readability
Extract long expressions out to variables if they're commonly re-used.
```c
// bad
if (expr.as_binop->rhs.type == expr.as_binop->lhs.type) {
    e = expr.as_binop->rhs;
    t = expr.as_binop->rhs.type;
}

// good
Expr rhs = expr.as_binop->rhs;
Expr lhs = expr.as_binop->lhs;
if (rhs.type == lhs.type) {
    e = rhs;
    t = rhs.type;
}
```

## General programming architecture and other miscellany

### Structure size

Generally adhere to the principles of Data Oriented Design, and try to limit the size of your structures.

### Check error states

Always check a function's return value for error states!

### Avoid raw allocations

Avoid allocating raw with malloc, try to use arena allocation or the allocation schemes built into other data types (like Vec). This is not a strict rule, and more of a guideline.
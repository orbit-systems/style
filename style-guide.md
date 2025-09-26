# Orbit Systems C Language Style Guide

## C Standard

All official Orbit Systems projects written in C should use the **C23 standard**. No, you can't change my mind.

## Control Flow
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
Pointer types, in declarations and parameters, should have the `*` attached to the subtype:
```c
int* xptr = &x;

void modify(int* ptr);
```

## Declarations & Assignments
**Never** declare or assign more than one variable in a single statement.
```c
// nuh uh
int x, y, z = 1;

// better
int x;
int y;
int z = 1;
```

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

## Headers
Use `#ifdef` guards with the form:
```
#ifndef HEADERNAME_H
#define HEADERNAME_H

// ...

#endif // HEADERNAME_H
```

Do not use `#pragma once`.

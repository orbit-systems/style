# orbit systems c language style guide

## indentation
indentation should use 4 spaces per level.

control flow statements with a single statement body should reside on the same line, or inside of a block (preferably the latter if the line is too long):
```c
if (x == y) return 1;

if (x == y) {
    return 1;
}

// no
if (x == y)
    return 1;
```

`case`s should match the level of their enclosing `switch`:
```c
switch (x) {
case 1:
    x = 100;
    break;
case 2:
    return bar;
}
```

## case
- use `snake_case` for variables, functions, and structure fields.
- use `PascalCase` for new types.
- use `SCREAMING_SNAKE_CASE` for `#define`'d constants and enums.
- macros should also use this case, but some exceptions are made for keyword mods like `for_range` that supplement language features, or inline function spoofing

## comments
prefer line comments (`// this`) in almost every case, except when a large
multi-line description is needed.


## types
use `orbit.h` defined types whenever possible, especially in data structures.
```rs
i8, i16, i32, i64, isize // integers
u8, u16, u32, u64, usize // unsigned integers
f16, f32, f64 // floating point
bool // boolean
```
when the size of an integer doesn't matter, prefer `isize` and `usize` over `int`.

### structs
when declaring a struct type, use typedef and include the name in the struct definition:
```c
typedef struct MyStruct {
    // ...
} MyStruct;
```

if it's necessary to refer to that type before the definition is available/complete, use a predeclaration:
```c
typedef struct MyStruct MyStruct;
```

### enums
when storing an enum in a data structure, ***always*** use a sized integer type.

Although not required, enums may have a type name for a better experience in debuggers like GDB.
```c
enum TokenKind {
    TOKEN_PLUS,
    TOKEN_MINUS,
    TOKEN_NUMBER,
};

typedef struct Token {
    // not TokenKind, cause sizeof(TokenKind) == sizeof(int), which wastes space
    u8 kind; 
} Token;
```

### pointers
pointer types, in declarations and parameters, should have the `*` attached to the subtype:
```c
int* xptr = &x;

void modify(int* ptr);
```

## declarations & assignments
never declare or set more than one variable in a single statement.
```c
// nuh uh
int x, y, z = 1;

// better
int x;
int y;
int z = 1;
```

extract long expressions out to variables if they're re-used at all.
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

## headers
headers should use `#pragma once` instead of `#ifdef` guards.
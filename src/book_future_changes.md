# Future Changes

## Lang and KLVM names for statements

Torq derives its kernel language from the Oz Computation Model where a *semantic statement* is a key concept. With that in mind, Torq defines `Stmt` as a semantic statement in agreement with the Oz model. The language parser uses `Sntc` for statements to avoid confusion. The Torq implementation for Rust is changing and Torq for Java will change in the future to match it.

- The KLVM package will use `Instr` instead of `Stmt`
- The language package will use `Stmt` instead of `Sntc`
- The language package will use `Sntc` instead of `Lang`

The language package type hierarchy gets rearranged.

Before:

- `Lang`
  - `Sntc`
  - `Expr`

After:

- `Sntc`
  - `Stmt`
  - `Expr`

## Narrow feature integers from Int64 to Int32

In consideration of value types (Valhalla) and Torq for Rust, we need to narrow the feature domain to the very reasonable range of 0 to 2,147,483,647.

## Import multiple components from same package

Change to be more consistent with other language syntax.

Before:
~~~
import system[ArrayList as JavaArrayList, Cell, ValueIter]
~~~
After:
~~~
import system.[ArrayList as JavaArrayList, Cell, ValueIter]
~~~

## Interactive Debugger

Current thinking is to provide an instrumented version of the KLVM "Machine" class.

## Gradual Typing

Type systems generally fall into two categories: static type systems, where every program expression must have a computed type before execution, and dynamic type systems, where nothing is known about types until execution, when the actual values are available.

Gradual typing lies between static typing and dynamic typing. Some variables and expressions are given types and correctness is checked at compile time (static typing). Other expressions are left untyped and type errors are reported at runtime (dynamic typing).

Torq is predisposed to gradual type checking as it already allows type annotations:
- Invariant variable type: `x::Int32`
- Invariant return type: `func x(x::Int32) -> Int32`

Current thinking:
- Untyped variables default to `Any`
- Type checks such as `Int32 == Any` succeed at compile time but may fail at run time
- Type checks such as `Int32 == Bool` fail at compile time

## Actor Protocols

Gradual typing enables explicit protocol definitions.

## Parametric Types

Allow type parameters on type declarations and type instantiations. For example, allow `ArrayList<Int32>.new()`.

> Do we use `<T>` like most languages or `[T]` like Scala?
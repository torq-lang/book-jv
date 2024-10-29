# Appendix A: Basic Types

Torq is a gradual, nominative, and parametric type system.

> The Torq gradual type system is a work in progress.

## Scalar and composite types

~~~
- Value
    - Comp
        - Obj
        - Rec
            - Tuple
            - Array
    - Literal
        - Bool
        - Eof
        - Null
        - Str
        - Token
    - Num
        - Dec128
        - Flt64
            - Flt32
        - Int64
            - Int32
                - Char
    - Meth
        - Func
        - Proc
~~~

## Actor, label and feature types 

~~~
- Value
    - Comp
        - Obj
            - ActorCfg
    - Feature
        - Int32
        - Literal
    - Meth
        - Func
            - ActorCfgtr
~~~

## Other types

### The dynamic type

Every type is compatible with `Any`. 

> Consider the notion of *strong arrows* as described here: https://elixir-lang.org/blog/2023/09/20/strong-arrows-gradual-typing/

### The `FailedValue` type

A failed value is a pseudo value used to propagate errors across actor boundaries. Any value may contain in its place a `FailedValue`. Accessing a failed value raises an exception that is ultimately returned to requesters or logged.
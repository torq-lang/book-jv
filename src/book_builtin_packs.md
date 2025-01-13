# Appendix B. Builtin Packs

## ArrayList

A wrapper for Java `ArrayList` that can contain zero or more values.

### Class Methods

#### ArrayList.new() -> ArrayList

Create an empty `ArrayList` using the underlying Java `ArrayList()` constructor.

#### ArrayList.new(values::Tuple) -> ArrayList

Create an `ArrayList` using the Java `ArrayList(field_count)` constructor, where `field_count` is the `values` size. Add each value in `values` to the newly created list.

After creation, an `ArrayList` can contain a mix of bound and unbound dataflow variables.

### Object Methods

#### array_list.add(element::Value)

Add `element` to the list using the corresponding Java `add(element)` method.

#### array_list.clear()

Invoke the corresponding Java `clear()` method.

#### array_list.size() -> Int32

Invoke the corresponding Java `size()` method and return its size as an `Int32`.

#### array_list.to_tuple() -> Tuple

Create and return a `Tuple` where each field corresponds in index and value to the elements of `array_list`.

### Iterator Sourcing

* `ValueIter`

---

## Cell

A mutable container with a single value.

### Class Methods

#### Cell.new(initial::Value) -> Cell

Create a `Cell` with an initial value `initial`.

---

## FeatureIter

If a type is a `FeatIter` source, it can be used as an argument in `FeatIter.new(iter_source)` expressions.

---

## FieldIter

A `FieldIter` is a generator. It defines no parameters and each time it is called, it returns either the next field or EOF. Once an iterator returns EOF it will always return EOF.

If a type is a `FieldIter` source, it can be used as an argument in `FieldIter.new(iter_source)` expressions.

### Class Methods

#### FieldIter.new(record::Rec) -> func () -> ([Feat, Value] | Eof)

Fields are returned in feature cardinality order:

1. Integers in ascending order
2. Strings in lexicographic order
3. Booleans in order with `false` before `true`
4. Nulls
5. Tokens in `id` order

---

## HashMap

A wrapper for Java `HashMap` that can contain zero or more mappings.

### Class Methods

#### HashMap.new() -> HashMap

Create an empty `HashMap` using the underlying Java `HashMap()` constructor.

### Object Methods

#### hash_map.get(key::Value) -> Value

Suspend until `key` becomes bound. Return the mapped value at `key`.

#### hash_map.put(key::Value, value::Value)

Suspend until `key` becomes bound. Invoke the underlying `put` method using `key` and `value` arguments.

### Iterator Sourcing

* `FeatIter`
* `FieldIter`
* `ValueIter`

---

## Int32

A utility pack for `Int32` operations.

### Class Methods

#### Int32.parse(num::Str) -> Int32

Parse an `Int32` from the given `num` argument using the `Integer#parseInt()` method.

---

## Int64

A utility pack for `Int64` operations.

### Class Methods

#### Int64.parse(num::Str) -> Int64

Parse an `Int64` from the given `num` argument using the `Long#parseLong()` method.

---

## LocalDate

A wrapper for Java `LocalDate`.

### Class Methods

#### LocalDate.parse(date::Str) -> LocalDate

Parse a `LocalDate` value from the given `date` argument using the underlying `java.time.LocalDate#parse(CharSequence text)` method.

---

## RangeIter

A `RangeIter` is a generator. It defines no parameters and each time it is called, it returns either the next number or EOF. Once an iterator returns EOF it will always return EOF.

### Class Methods

#### RangeIter.new(from::Int32, to::Int32) -> func () -> (Int32 | Eof)

The range includes `from` but excludes `to`.

---

## Rec

### Class Methods

#### Rec.assign(from::Rec, to::Rec) -> Rec

Create a new record as if it was initialized with the `to` fields, and then was assigned `from` fields in a way that new fields are added and existing fields are replaced.

```
var from = { b: 4, c: 5 }
var to = { a: 1, b: 2 }
var result = Rec.assign(from, to)
// result = { a: 1, b: 4, c: 5 }
```

#### Rec.size(rec::Rec) -> Int32

Return the field count in `rec`.

---

## Str

### Instance Methods

#### str.substring(start::Int32) -> Str

#### str.substring(start::Int32, stop::Int32) -> Str

---

## Stream

### Class Methods

#### Stream.new(actor::Actor, request_message::Value) -> func () -> Rec

### Iterator Sourcing

* `ValueIter`

---

## StringBuilder

```
TBD
```

---

## Timer

A Timer generates a stream of ticks as `<period 1>, <tick 1>, ..., <period n>, <tick n>` where each `<period n>` is a delay and each `<tick 1>` is a response.

A timer can be configured with a period and a time unit.

```
var timer_cfg = Timer.cfg(1, 'seconds')
```

Subsequently, a timer is spawned as a publisher.
```
var timer_pub = spawn(timer_cfg)
```

Once spawned, a timer can be consumed using a `Stream` client. The following example iterates over 5 one-second timer ticks. 
```
var timer_pub = spawn(Timer.cfg(1, 'seconds'))
var tick_count = Cell.new(0)
var timer_stream = Stream.new(timer_pub, 'request'#{'ticks': 5})
for tick in Iter.new(timer_stream) do
    tick_count := @tick_count + 1
end
@tick_count
```

A timer can only be used by one requester (subscriber) at a time.

### Configurators

#### Timer.cfg(period::Int32, time_unit::Str) -> ActorCfg

Configure a timer with the given delay period and its time unit.

### Protocol

#### handle stream 'request'#{'ticks': Int32} -> (Int64[] | 'eof'#{more: Bool})

> WARNING: The stream protocol is being enhanced and is a work in progress.

Respond with a stream of ticks where each tick is delayed by the period as configured.

The stream protocol declares a single-request, multiple-response protocol between a publisher and subscriber. Specifically, a stream begins with a single request and ends with the `eof#{'more': false}` response. In between the request and end-of-file are zero to many responses where each response is an array of the stream type `Int64`.

An example of the `handle stream 'request'#{'ticks': Int32} -> Int64` protocol:
1. Send request: `request#{'ticks': 5}`
2. Accept response: `[1731160281000]`
3. Accept response: `[1731160282000]`
4. Accept response: `[1731160283000]`
5. Accept response: `[1731160284000]`
6. Accept response: `[1731160285000]`
7. Accept response: `eof#{'more': false}`

A programmer does not directly implement the stream protocol mentioned above. Instead, the programmer instantiates a stream client to request and consume the timer ticks. For example, the code below instantiates a `Stream` client for a `Timer` publisher and its `'request'#{'ticks': Int32}` message pattern.

```
var timer_stream = Stream.new(spawn(Timer.cfg(1, 'seconds')),
    'request'#{'ticks': 5})
for tick in ValueIter.new(timer_stream) do
    // Do something once per second for 5 seconds
end    
```

> TODO: The `Stream.new(...)` technique above needs to change to the technique below:

```
var timer_ref = spawn(Timer.cfg(1, 'seconds')) 
var timer_stream = timer_ref.stream('request'#{'ticks': 5})
for tick in ValueIter.new(timer_stream) do
    // Do something once per second for 5 seconds
end    
```

---

## Token

A `Token` is an unforgeable value. Typically, tokens are used in-process to secure services. A requester that has been authenticated or authorized is issued a token. Later, the requester must present the token as proof.

### Class Methods

#### Token.new() -> Token

Create a new token.

---

## ValueIter

If a type is a `ValueIter` source, it can be used as an argument in `ValueIter.new(iter_source)` expressions.

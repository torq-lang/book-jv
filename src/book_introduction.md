# Introduction

Welcome to the Torq Programming Language for Java developers.

Big Data analytics, artificial intelligence, and IT/OT convergence are pressuring organizations to be data-centric. The urgent need to manage explosive data growth with meaning, where billions of files and records are distributed over thousands of computers, is a significant challenge.

Torq addresses these challenges by simplifying data-centric programming at scale with a novel programming model. This allows programmers to focus on the application problem rather than the programming problem, leading to efficiencies that lower costs.

The Torq programming model is designed to improve scalability, responsiveness, and faster time-to-market in three problem areas:

1. Data contextualization - enhancing live data meaningfully before it is stored
2. Enterprise convergence - creating meaningful APIs across departments and divisions
3. Situational applications - composing low-cost, low-code, highly-valued applications quickly

<dl>
    <dt>Data contextualization</dt>
    <dd>Modern data fuels analytics and artificial intelligence. To make data specifically meaningful, it needs preprocessing, contextualization, and additional features before reaching storage. Torq enhances existing dataflows efficiently with related data to help reveal insights that would otherwise go unseen.</dd>
    <dt>Enterprise convergence</dt>
    <dd>Enterprise workflows require applications spanning multiple departments and divisions. Initiatives, such as the "Unified Name Space," aim to tear down traditional silos. Torq provides a dynamic platform to unify the enterprise with composable services that integrate departments and divisions.</dd>
    <dt>Situational applications</dt>
    <dd>Recent advancements have enabled solutions that were out of reach only a few years ago, creating a massive demand for new solutions. Low-code platforms help citizen developers reduce software backlogs. Torq is a truly low-code platform that scales and executes efficiently.</dd>
</dl>

Torq is a dynamic, gradually typed, concurrent programming language with novel ease-of-use and efficiency.

## Concurrent programming is hard, really hard

What makes concurrent programming so hard that it would justify a new programming language like Torq?

Mainstream programming suffers an incurable problem: variables alone cannot refer to shared memory values from multiple threads. Shared memory values require multiple threads to explicitly synchronize access, and as [Lee (2006)](book_references.html) formalized, programming over shared memory with threads is a failure. To solve this problem, the industry has adopted models other than shared memory for concurrency, such as message passing, functional programming, and borrow checking. In each of these models, programming is highly structured to avoid shared memory and unsafe access. Unfortunately, these models tend to create complex and tangled code. In the degenerative case, they organize code into concurrency solutions instead of application solutions.

A programming model does exist where multiple threads can share memory without explicit synchronization. Borrowed from logic programming, *Declarative Dataflow* ([Van-Roy, P., & Haridi, S., 2004](book_references.html)) computes over single-assignment variables called dataflow variables. A *dataflow variable* is initially unbound, but once bound, it is immutable. Threads that produce information bind dataflow variables, and threads that consume information suspend until dataflow variables are bound. This *dataflow rule* describes a producer-consumer interaction that implicitly synchronizes access to shared memory, resulting in a natural style void of technical concerns unrelated to the problem. Unfortunately, adopting declarative dataflow is highly disruptive because it requires runtime architectures to redefine the fundamental semantics of the memory variable for all processes.

## Actorflow

Torq is a dynamic programming language based on *Actorflow*, a patented programming model that fuses message-passing actors with a hidden implementation of declarative dataflow. Concurrently executing actors only communicate by sending immutable messages. Requests and responses are correlated with private dataflow variables, bound indirectly by a controller. Actors that send requests may suspend, waiting for a variable bound by a response. Actors that receive messages may resume when a received message binds a waiting variable. This request-response interaction provides synchronization without sharing variables, giving us a naturally sequential programming style. Moreover, we can compose programs using a mix of libraries from other programming languages. All variables, dataflow or otherwise, are hidden.

Consider the following program written as a Torq actor. `ConcurrentMath` calculates the number `7` using three concurrent child actors to supply the operands in the expression `1 + 2 * 3`. This example is an unsafe race condition in mainstream languages. However, in Torq, this sequential-looking but concurrently executing code will always calculate `7` because of the dataflow rule defined previously. Notice that Torq honors operator precedence without explicit synchronization.

```
actor ConcurrentMath() in
    actor Number(n) in
        handle ask 'get' in
            n
        end
    end
    var n1 = spawn(Number.cfg(1)),
        n2 = spawn(Number.cfg(2)),
        n3 = spawn(Number.cfg(3))
    handle ask 'calculate' in
        n1.ask('get') + n2.ask('get') * n3.ask('get')
    end
end
```

## Concurrent Construction

Torq facilitates a concurrent style of programming not possible in mainstream languages. Consider the next example as a slightly modified version of our previous example. Instead of *calculating* concurrently, we *construct* concurrently. The concurrent math calculation `x + y * z` from our first example is replaced with a concurrent data construction `[x, y, z]`, where `x`, `y`, and `z` stand for `n1.ask('get')`, `n2.ask('get')`, and `n3.ask('get')`, respectively.

```
actor ConcurrentMathTuple() in
    actor Number(n) in
        handle ask 'get' in
            n
        end
    end
    var n1 = spawn(Number.cfg(1)),
        n2 = spawn(Number.cfg(2)),
        n3 = spawn(Number.cfg(3))
    handle ask 'calculate' in
        [n1.ask('get'), n2.ask('get'), n3.ask('get')]
    end
end
```

Dataflow variables make concurrent construction possible. Instead of using futures (functions and callbacks), like other programming languages, Torq uses dataflow variables to construct partial data that is complete when concurrent tasks are complete. In essence, futures wait for logic, but Torq waits for data. Implicit synchronization provided by dataflow variables is not the same as the syntactic sugar provided by async-await for futures (sometimes called promises). In essence, Torq is non-blocking-concurrent whereas async-await is non-blocking-sequential.

## Streaming Data

In mainstream programming, vendors provide reactive stream libraries "certified" with an official TCK (Technology Compatibility Kit). In Torq, reactive streams are a natural part of the language. A publisher is simply an actor that can respond multiple times to a single request.

The `IntPublisher` can be configured with a range of integers and an increment.

```
actor IntPublisher(first, last, incr) in
    import system[ArrayList, Cell]
    import system.Procs.respond
    var next_int = Cell.new(first)
    handle ask 'request'#{'count': n} in
        func calculate_to() in
            var to = @next_int + (n - 1) * incr
            if to < last then to else last end
        end
        var response = ArrayList.new()
        var to = calculate_to()
        while @next_int <= to do
            response.add(@next_int)
            next_int := @next_int + incr
        end
        if response.size() > 0 then
            respond(response.to_tuple())
        end
        if @next_int <= last then
            eof#{'more': true}
        else
            eof#{'more': false}
        end
    end
end
```

The `SumOddIntsStream` example spawns and iterates an `IntPublisher`, summing the integers not divisible by two.

```
actor SumOddIntsStream() in
    import system[Cell, Stream, ValueIter]
    import examples.IntPublisher
    handle ask 'sum'#{'first': first, 'last': last} in
        var sum = Cell.new(0)
        var int_publisher = spawn(IntPublisher.cfg(first, last, 1))
        var int_stream = Stream.new(int_publisher, 'request'#{'count': 3})
        for i in ValueIter.new(int_stream) do
            if i % 2 != 0 then sum := @sum + i end
        end
        @sum
    end
end
```

The `MergeIntStreams` example spawns two configurations of `IntPublisher`, one for even numbers and one for odd numbers. The even stream publishes 3 integers at a time and the odd stream publishes 2 integers at a time. After spawning the streams, the example performs an asynchronous, concurrent, non-blocking, merge sort while honoring backpressure.

```
actor MergeIntStreams() in
    import system[ArrayList, Cell, Stream, ValueIter]
    import examples.IntPublisher
    handle ask 'merge' in
        var odd_iter = ValueIter.new(Stream.new(spawn(IntPublisher.cfg(1, 10, 2)), 'request'#{'count': 3})),
            even_iter = ValueIter.new(Stream.new(spawn(IntPublisher.cfg(2, 10, 2)), 'request'#{'count': 2}))
        var answer = ArrayList.new()
        var odd_next = Cell.new(odd_iter()),
            even_next = Cell.new(even_iter())
        while @odd_next != eof && @even_next != eof do
            if (@odd_next < @even_next) then
                answer.add(@odd_next)
                odd_next := odd_iter()
            else
                answer.add(@even_next)
                even_next := even_iter()
            end
        end
        while @odd_next != eof do
            answer.add(@odd_next)
            odd_next := odd_iter()
        end
        while @even_next != eof do
            answer.add(@even_next)
            even_next := even_iter()
        end
        answer.to_tuple()
    end
end
```

These kinds of asynchronous, concurrent, non-blocking, reactive streams cannot be created using mainstream languages and libraries that rely on async-await. The mainstream approach is non-blocking, *sequential* programming, which is insufficient. Reactive streams require non-blocking, *concurrent* programming.

"Asynchronous, Concurrent, Non-blocking, Backpressure"
- Asynchronous: actors don't necessarily wait on other actors
- Concurrent: actors progress independently and opportunistically
- Non-blocking: operating system threads are released while waiting
- Backpressure: publishers cannot produce more than requested

Recall that concurrent is the potential to run in parallel.

## Higher concurrency and throughput

The Torq effect is more than just a natural programming style. The implicit synchronization afforded by dataflow variables can increase concurrency and throughput by reducing synchronization barriers. Consider the following comparison of a simple use-case written in Torq versus Java.

> TODO: Insert the Torq versus Java throughput example--the simple use-case that retrieves a customer order, product, and contact history.

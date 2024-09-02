# Introduction

Welcome to the Torq Programming Language for Java developers.

Big Data analytics, artificial intelligence, and IT/OT convergence are pressuring organizations to be data-centric. The urgent need to manage explosive data growth without losing meaning, where billions of files and records are distributed over thousands of computers, is a significant challenge. Torq addresses several problem areas directly with its simple yet highly concurrent data processing model.

Several problem areas motivated the design of Torq:

1. Data contextualization - enhance live data meaningfully before it is stored
2. Enterprise convergence - create meaningful services spanning departments and divisions
3. Situational applications - compose low-cost highly-valued applications quickly

<dl>
    <dt>Data contextualization</dt>
    <dd>Modern data fuels analytics and artificial intelligence. To make data specifically meaningful, it needs preprocessing, contextualization, and additional features before reaching storage. Torq can enhance existing dataflows efficiently with related data to help reveal insights that would otherwise go unseen.</dd>
    <dt>Enterprise convergence</dt>
    <dd>Enterprise workflows require applications spanning multiple departments and divisions. Initiatives, such as the "Unified Name Space," aim to tear down traditional silos. Torq can help unify the enterprise by composing meaningful services spanning departments and divisions.</dd>
    <dt>Situational applications</dt>
    <dd>Recent advancements have enabled solutions that were out of reach only a few years ago, creating a massive demand for new solutions. Low-code platforms help citizen developers reduce software backlogs. Torq can be used openly and directly to create low-code solutions with intrinsic concurrency. Existing platforms, on the other hand, must implement proprietary processing engines driven by configurations files.</dd>
</dl>

## Concurrent programming is hard, really hard

What makes concurrent programming so hard that it would justify a new programming language like Torq?

Mainstream programming suffers an incurable problem: variables alone cannot refer to shared memory values from multiple threads. Shared memory values require multiple threads to explicitly synchronize access, and as [Lee (2006)](part_0900_references.html) formalized, programming over shared memory with threads is a failure. To solve this problem, the industry has adopted models other than shared memory for concurrency, such as message passing, functional programming, and borrow checking. In each of these models, programming is highly structured to avoid shared memory and unsafe access. Unfortunately, these models tend to create complex and tangled code. In the degenerative case, they organize code into concurrency solutions instead of application solutions.

A programming model does exist where multiple threads can share memory without explicit synchronization. Borrowed from logic programming, *Declarative Dataflow* ([Van-Roy, P., & Haridi, S., 2004](part_0900_references.html)) computes over single-assignment variables called dataflow variables. A *dataflow variable* is initially unbound, but once bound, it is immutable. Threads that produce information bind dataflow variables, and threads that consume information suspend until dataflow variables are bound. This *dataflow rule* describes a producer-consumer interaction that implicitly synchronizes access to shared memory, resulting in a natural style void of technical concerns unrelated to the problem. Unfortunately, adopting declarative dataflow is highly disruptive because it requires runtime architectures to redefine the fundamental semantics of the memory variable for all processes.

## Actorflow

Torq is a new programming language based on *Actorflow*, a patented programming model that fuses message-passing actors with a hidden implementation of declarative dataflow. Concurrently executing actors only communicate by sending immutable messages. Requests and responses are correlated with private dataflow variables, bound indirectly by a controller. Actors that send requests may suspend, waiting for a variable bound by a response. Actors that receive messages may resume when a message received binds a waiting variable. This request-response interaction provides synchronization without sharing variables, giving us a naturally sequential programming style. Moreover, we can compose programs using a mix of libraries from other programming languages. All variables, dataflow or otherwise, are hidden.

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

Torq facilitates a concurrent style of programming not possible in other programming languages. Consider the next example as a slightly modified version of our previous example. Instead of *calculating* concurrently, we *construct* concurrently. The concurrent math calculation `x + y * z` in our first example is replaced with a concurrent data construction `[x, y, z]` in our second example. 

> For clarity, we replaced the `ask` expressions `n1.ask('get')`, `n2.ask('get')`, and `n3.ask('get')` with the symbols `x`, `y`, and `z`, respectively.

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

Dataflow variables make concurrent construction possible. Instead of using functional futures, like other programming languages, Torq uses dataflow variables to construct partial data that are complete when concurrent tasks are complete. On the other hand, functional futures require the programmer to synchronize outside the data. The example above hints that Torq can reduce the number of synchronization barriers and thus increase concurrency. As we will see later, Torq can increase concurrency significantly when returning multiple records or streaming data.

> In many languages, Futures are also known as Promises.

Note that the implicit synchronization provided by dataflow variables is not the same as the syntactic sugar provided by async-await. Torq is non-blocking concurrent whereas async-await is non-blocking sequential.

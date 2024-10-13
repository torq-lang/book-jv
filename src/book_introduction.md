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
    <dd>Recent advancements have enabled solutions that were out of reach only a few years ago, creating a massive demand for new solutions. Low-code platforms help citizen developers reduce software backlogs. Torq is a truly low-code platform that scale and execute efficiently.</dd>
</dl>

## Concurrent programming is hard, really hard

What makes concurrent programming so hard that it would justify a new programming language like Torq?

Mainstream programming suffers an incurable problem: variables alone cannot refer to shared memory values from multiple threads. Shared memory values require multiple threads to explicitly synchronize access, and as [Lee (2006)](book_references.html) formalized, programming over shared memory with threads is a failure. To solve this problem, the industry has adopted models other than shared memory for concurrency, such as message passing, functional programming, and borrow checking. In each of these models, programming is highly structured to avoid shared memory and unsafe access. Unfortunately, these models tend to create complex and tangled code. In the degenerative case, they organize code into concurrency solutions instead of application solutions.

A programming model does exist where multiple threads can share memory without explicit synchronization. Borrowed from logic programming, *Declarative Dataflow* ([Van-Roy, P., & Haridi, S., 2004](book_references.html)) computes over single-assignment variables called dataflow variables. A *dataflow variable* is initially unbound, but once bound, it is immutable. Threads that produce information bind dataflow variables, and threads that consume information suspend until dataflow variables are bound. This *dataflow rule* describes a producer-consumer interaction that implicitly synchronizes access to shared memory, resulting in a natural style void of technical concerns unrelated to the problem. Unfortunately, adopting declarative dataflow is highly disruptive because it requires runtime architectures to redefine the fundamental semantics of the memory variable for all processes.

## Actorflow

Torq is a new programming language based on *Actorflow*, a patented programming model that fuses message-passing actors with a hidden implementation of declarative dataflow. Concurrently executing actors only communicate by sending immutable messages. Requests and responses are correlated with private dataflow variables, bound indirectly by a controller. Actors that send requests may suspend, waiting for a variable bound by a response. Actors that receive messages may resume when a received message binds a waiting variable. This request-response interaction provides synchronization without sharing variables, giving us a naturally sequential programming style. Moreover, we can compose programs using a mix of libraries from other programming languages. All variables, dataflow or otherwise, are hidden.

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

Dataflow variables make concurrent construction possible. Instead of using futures (functions and callbacks), like other programming languages, Torq uses dataflow variables to construct partial data that is complete when concurrent tasks are complete. In essence, futures wait for logic, but Torq waits for data.

> The implicit synchronization provided by dataflow variables is not the same as the syntactic sugar provided by async-await for futures (sometimes called promises). In essence, Torq is non-blocking-concurrent whereas async-await is non-blocking-sequential. For example, a Torq API server may run a single request over multiple hardware threads simultaneously, whereas an API server written with async-await will not.

The Torq effect is more than just a natural programming style. The implicit synchronization afforded by dataflow variables can increase concurrency by reducing synchronization barriers. Consider the following comparison of a simple use-case written in Torq and then Java.

> TODO: Insert a Torq vs Java example. Compare a simple use-case written in Torq and Java that retrieves a customer order, product, and contact history.

> TODO: Insert a streaming example that exploits fewer synchronization barriers
 
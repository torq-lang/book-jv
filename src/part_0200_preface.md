# Preface

Like some of you, I programmed sequentially for years, using the occasional thread here and there. I mostly enjoyed the performance gains as processors became faster and faster. Then, processors stopped becoming faster. Instead, processors started becoming more parallel by adding more cores. I became painfully aware that my programming style was not the future. Moreover, programming applications imperatively with shared state, critical sections, and semaphores had been declared an abject failure. I needed to learn a new programming model.

> [Moore's Law](<https://www.researchgate.net/publication/2999593_Moore's_Law_Past_Present_and_Future>) 
 
> [The Problem With Threads](<https://www2.eecs.berkeley.edu/Pubs/TechRpts/2006/EECS-2006-1.html>)

A colleague introduced me to an early version of Scala. Programming in Scala opened my eyes to a new world of concurrent actors, monads, and multi-paradigm programming. I worked hard and became confident in this new environment, but the applications I produced were suspect. Instead of forming a business solution, my programs formed a concurrency solution. Although I liked the sophistication of this new environment, the results just felt wrong. Writing application code, especially for business applications, should be easy, and the results should express the solution clearly without requiring special knowledge.

While battling the concurrent programming experience, other trends affected my opinions around concurrent programming, such as cloud computing, IT and OT convergence, low-code development, and software composition using web services. Along the way, I studied "Concepts, Techniques, and Models in Computer Programming," paying particular attention to a programming model named *Declarative Dataflow* that greatly simplified concurrent programming. Declarative dataflow introduced a new construct named the dataflow variable, making concurrent programming extremely simple. I began imagining how this dataflow construct could streamline and accelerate the development of modern cloud applications, low-code platforms, and software composition.

In my first experiment with declarative dataflow, I created a dynamic programming language named Ozy that ran as a companion language on the JVM. The experiment was especially fruitful as it exposed a weakness in declarative dataflow--dataflow variables cannot be easily shared with other programming languages--and motivated a novel solution.

> [Ozy: A General Orchestration Container](<https://arxiv.org/abs/1604.07642>)

In 2020, I found myself locked down because of COVID-19, staring at a blank screen with an epiphany for a new programming language. The central idea was an actor construct fusing the message-passing protocol with a hidden implementation of declarative dataflow. Unlike typical actor systems, programs would not be formed as state machines, and unlike declarative dataflow, programs would interoperate without knowing dataflow variables. The name Torq came to me as I imagined a programming language with the power to service millions of requests incrementally and fairly, moving massive amounts of data without stalling.

Torq is the culmination of a personal journey to realize a new programming experience. It consists of a language and a patented programming model named Actorflow. The language is dynamic with optional type annotations, and although interpreted, Torq can be faster than compiled languages by utilizing multiple processors more efficiently. The Actorflow model gives Torq encapsulation power, where actor messages are the inputs and outputs of a hidden dataflow machine.

This book presents Torq by example, where each example builds on previous examples. I hope you enjoy it.

--Glenn Osborne

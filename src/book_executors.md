# Executors

In this section, we use executors to partition our actors into CPU-bound or I/O-bound thread pools. CPU-bound actors do not block threads while waiting and need only a few threads to maintain throughput. I/O-bound actors, on the other hand, block threads while they wait for storage or network I/O to complete. As a consequence, they need more threads to maintain throughput.

By default, Torq actors are CPU-bound. When we create native actors like the Northwind DB and interact with I/O subsystems, we need to use I/O executors with more threads.

## Sizing thread pools

The common practice for allocating a thread pool is to allocate `n + 1` threads where `n` is the number of available hardware threads. Brian Goetz, one of the authors of "Java Concurrency in Practice," suggests a more thoughtful formula: `thread_pool_size = available_hw_threads * target_utilization * (1 + wait_time / compute_time)`

The formula above uses two variables to calculate the base allocation of threads for a thread pool `available_hw_threads * target_utilization`. Next, the formula calculates the ratio `wait_time / compute_time` and uses it to increase the resulting thread pool size. It increases the pool size dramatically if we are I/O-bound.

Consider a CPU-bound ratio of 1 I/O second for every 10 compute seconds, resulting in a one-tenth increase `1 / 10`. Now, consider an I/O-bound ratio of 10 I/O seconds for every 1 compute second, resulting in a 10 times increase `10 / 1`. If we have 32 hardware threads, but only want to utilize 25% of them per thread pool, we get:

- CPU-bound thread pool: `thread_pool_size = 32 * .25 * (1 + 1/10)` = `8.8` or `9` if we round up
- I/O-bound thread pool: `thread_pool_size = 32 * .25 * (1 + 10/1)` = `88`

As we can see, this formula allocates far more threads for I/O bound actors. Our example used the simple ratios `1 / 10` and `10 / 1`, respectively. In practice, actual pool sizes will vary as `wait_time / compute_time` varies.
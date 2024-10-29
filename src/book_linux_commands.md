# Appendix C: Linux Commands

This appendix contains linux commands used in this book.

## List CPU

List CPU information. In this instance, we listed the CPU information for one of our workstations running an AMD Ryzen 9 CPU.

```
lscpu
```

```
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          48 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   32
  On-line CPU(s) list:    0-31
Vendor ID:                AuthenticAMD
  Model name:             AMD Ryzen 9 7950X 16-Core Processor
    CPU family:           25
    Model:                97
    Thread(s) per core:   2
    Core(s) per socket:   16
    Socket(s):            1
    Stepping:             2
    CPU(s) scaling MHz:   25%
    CPU max MHz:          5881.0000
    CPU min MHz:          545.0000
```

## Run on select CPU Cores

Run a Java program constrained to a select set of hardware threads. In this instance, we run `RunNorthwindDb` using cores 0, 1, and 2.

```
taskset -c 0-2 java -XX:+UseZGC -p ~/workspace/torq_jv_runtime -m org.torqlang.examples/org.torqlang.examples.RunNorthwindDb
```

## Display runtime stats

Show cache misses:

```
perf stat -e cache-misses COMMAND
```

Show cache misses while running `RunNorthwindDb` using 3 cores:

```
perf stat -e cache-misses taskset -c 0-2 java -XX:+UseZGC -p ~/workspace/torq_jv_runtime -m org.torqlang.examples/org.torqlang.examples.RunNorthwindDb
```

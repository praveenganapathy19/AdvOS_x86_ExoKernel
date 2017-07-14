# Homework 17

Paper to read for homework assignment: [IX: A Protected Dataplane Operating System for High Throughput And Low Latency (2014)](https://pdos.csail.mit.edu/6.828/2014/readings/osdi14-paper-belay.pdf).

Question: Examine the figure of the network server in the [lab 6 handout](https://pdos.csail.mit.edu/6.828/2014/labs/lab6/index.html). What aspects of the lab6 and the IX design are similar and what aspects are different?

I decided to break down my answer in the sections below. When I refer to JOS, I am talking about the network stack as described in the [lab 6 handout](https://pdos.csail.mit.edu/6.828/2014/labs/lab6/index.html).

## Parallelism & scheduling

IX achieves true parallelization ("no synchronization and coherence traffic in the common case") by using hardware queues on the NICs and by pinning resources to a dataplane (CPU & memory). Specifically, elastic threads are pinned to a core as they are responsible for the lifecycle of a packet (this is where another design aspect, run to completion, comes in handy). Packets received by the NICs, are hashed into separate queues and then processed by a dataplane, in parallel.

JOS on the other hand offers limited parallelism. Access to the E1000 controller is single threaded, via the input and output environments. There are no special packet queues, all packets go to either a receive queue or a transmit queue. The only parallelism happens in the network server, which uses a thread per request model to server client connections. The server does this to avoid blocking entirely while a client waits for packets.

## API

IX and JOS both provide users with library wrappers around their lower level APIs to ease development. IX offers `libix`, which exposes non-blocking POSIX socket operations. Similarly, JOS allows users to operate on sockets instead of using the lower level IPC calls.

## Memory efficiency

JOS uses DMA for communication between the driver and the E1000 chip, thus avoiding a copy operation. However, getting the packets to and from the input and output environments involves a copy because the communication happens over a system call. The driver, which lives in the kernel, cannot make its address space writable by user environments so instead the input and output environments include in the system call argument an address to which the driver can copy the packets.

On the other hand IX avoids copying altogether by sharing memory between the kernel and application and performing all communication over messages stored in memory. The obvious drawback of this is that packets need to remain in the sender's memory until the receiver acknowledges their reception.

## Run to completion

This concept doesn't apply to JOS where packets are handled independently by each component in the networking stack. For example: the driver processes a set of packets and copies them into memory provided by the input environment, which in turn hands them to the network server for protocol processing, which then returns those packets to the application. This sequence doesn't happen sequentially like in IX. Instead each stage is run by a different thread, running in a different address space and once a stage is done with a set of packets it can start on the next set without waiting for the packets it just processed to get sent/received. This decoupling between the application and the protocol processing is done for scheduling flexibility.

IX runs each packet to completion on the same elastic thread using adaptive batching. Run to completion interleaves both protocol and application processing. This is beneficial because as explained in the "successive stages tend to access many of the same data, leading to better data cache locality".

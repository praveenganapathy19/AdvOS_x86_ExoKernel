# Homework 13

Paper read for homework assignment: [Exokernel: An Operating System Architecture for Application-Level Resource Management (1995)](https://pdos.csail.mit.edu/6.828/2014/readings/engler95exokernel.pdf).

Question: In lab 4 you completed building the core of an exokernel-based operating system. In class you studied xv6, a monolithic operating systems. Both are intend to support the UNIX API, but their internal organizations are different. A good example is the virtual memory implementation: JOS implements many virtual-memory-related functions (such as fork) in its library operating system, while xv6 implements them in the kernel. Give an concrete example of what a user program could do in JOS that a user-level program on xv6 is unable to do and why it is useful.

I really enjoyed the exokernel paper! Some of the design choices in the JOS labs make more sense now.

The idea behind the paper is to separate the management and protection responsibilities of the kernel. Monolithic kernels do both, while exokernels only provide protection and leave all resource management to application operating systems. This means that the kernel is no longer forcing applications to use a fixed interface (VM, FS, etc.) and instead applications can now make choices on which abstractions to use based on their domain. Having this choice is beneficial because it enables applications to manage resources in a way that best fits their workload.

For example, an exokernel will let an application pick where to map its own pages in memory. Having pages physically located next to each other improves cache performance. This kind of optimization would not be possible with a monolithic kernel, which would just pick a physical page for you. The exokernel also includes the application in resource revocation. For example, if the kernel is under memory pressure it will ask the application to free up a page. It is then up to the application to pick a page and release it. This is very useful for database applications, which may have workloads that do not fit the default page eviction scheme of a monolithic kernel. Better yet, every application can have a different page eviction policy by including a different library operating system. This flexibility is not possible in a monolithic kernel and is what makes Aegis perform so well, as shown in the paper.
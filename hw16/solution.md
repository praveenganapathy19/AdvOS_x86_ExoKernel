# Homework 16

Paper to read for homework assignment: [Dune: Safe User-level Access to Privileged CPU Features (2012)](https://pdos.csail.mit.edu/6.828/2014/readings/belay-dune.pdf).

Question: One application of Dune is supporting sthread (see Section 5.2). Is it impossible to implement this abstraction in Linux? What is the role of Dune? (Briefly explain your answer.)

To fully understand this paper I had to do some reading on virtualization as I had no background whatsoever. I used the following resources: [CMU presentation on VT-x](http://www.cs.cmu.edu/~412/lectures/L04_VTx.pptx), [presentation on trap-and-emulate virtualization technique](http://www.cs.usfca.edu/~cruse/cs686s07/lesson19.ppt) and [a textbook chapter on virtualization](http://pages.cs.wisc.edu/~remzi/OSTEP/vmm-intro.pdf).

To determine whether `sthread` can be implemented in Linux let's break down its features. `sthread` is a thread with:
- fork-like isolation,
- fast creation,
- access to resources (memory, file descriptors & syscalls) according to a policy.

Fork-like isolation is possible in both Dune and Linux. In both cases, threads get their own IDs, their resources can be constrained via kernel limits (e.g. max number of open file descriptors) and threads can't access memory outside of their own process address space.

Speed of creation is relative. In both Dune and Linux creating threads is faster than creating processes and switching between threads is faster than switching between processes. However, as demonstrated in section 6.3.2 of the paper, creating an `sthread` in Dune is much faster than creating a thread (calling `fork()`) in Linux. This speed increase is possible because of to features only available in Dune. `sthread`s are not created from scratch but are instead recycled from a pool. This is made possible thanks to Dune's ability to avoid TLB flushes at thread creation and thread context switch. In addition, Dune avoids the overhead of a system call for creation because it recycles an already existing thread.

Finally, configuring a thread's access policy is only possible in Dune. For example, with the `sthread` running in ring 3, executing any privileged instruction will trap into a controlled runtime (similar to what is described in the sandboxing section) that is executing in ring 0 and which can then decide whether the call should be allowed to execute. Dune gives processes access to their page tables, which means that an `sthread`'s access to sections of memory can be limited by setting the supervisor bit on those pages.

Even though some of the `sthread` features can be replicated in Linux, Dune allows for finer grained control over the thread's permissions and achieves faster creation and context switching.

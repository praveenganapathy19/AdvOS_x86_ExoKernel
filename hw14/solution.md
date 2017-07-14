# Homework 14

Paper read for homework assignment: [Non-scalable locks are dangerous (2012)](https://pdos.csail.mit.edu/6.828/2014/readings/linux-lock.pdf).

Question: The ticket lock is slightly different than the spinlock in in JOS and xv6. Assuming an invalidation-based coherence scheme as described in the paper, does a ticket lock introduce less inter-CPU communication than a JOS/xv6 spinlock?

The spinlock uses an atomic test-and-set instruction so my answer depends on how that instruction is implemented :) If the value we're trying to set is not being cached locally and the core is manipulating RAM directly, then the JOS/xv6 spinlock causes less traffic than the ticket lock because no cache entries need to be invalidated. On the other hand, if the testing value is cached locally and checked as long as it is marked S (shared), then the JOS/xv6 spinlock causes the same amount of traffic as the ticket lock.

The `xchg` instruction at some point has to lock the bus to get exclusive access to the value in RAM and prevent a race with another core. So even if the value is not being cached, and thus no cache invalidation messages get passed around, the spinlock is still unscalable because of the lock `xchg` places on the bus. As the number of cores trying to acquire the lock grows, each core will have to wait on average longer to get access to the bus, resulting in O(n) runtime.

# Homework 15

Paper read for homework assignment: [RadixVM: Scalable address spaces for multithreaded applications (2013)](https://pdos.csail.mit.edu/6.828/2014/readings/radixvm.pdf).

Question: Refcache re-examines the reference count of an object two epochs later after it inserts the object in a per-core review queue (see page 4, 2nd paragraph, right column). What goes wrong if refcache reviews objects only one epoch later? Is two epochs later just more efficient or is it necessary for correctness? (Briefly explain your answer.)

If the object were re-examined one epoch later then it is possible that not all cores will have flushed their cached counts. The example of this given in the paper is the dirty zero observed in epoch 5. Core 2 flushes at the beginning of the epoch and the increment immediately following will only be flushed in epoch 6. A decrement flushed later in epoch 5 make it look like the global count has dropped to zero when in reality we still haven't accounted for core 2's 'hidden increment'. In essence, waiting two epochs is necessary but not sufficient, as stated in the paper. In the case of a dirty zero, an additional two epoch wait is scheduled. The necessary and sufficient condition is to observe a full epoch with no increments/decrements. This guarantees that the flushed global count is the true count.

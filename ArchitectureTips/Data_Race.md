
# Data Race Free
<span style="font-family: 'Lucida Console';">

In General Relativity, the sequence of two events happened in two different reference system can only be
determined after Lorentz transformations. In another word, if event A happens before event B under one
observer's prespective, event A could happens after event B under another perspective from another reference
system. The absolute sequence between A and B just doesn't exist in the nature. In modern parreller computer
architecture, those load and store operations plays the role of the events, while each process plays the observer
under different reference system. Ensuring the sequence under the perspective of one core, doesn't gurantee the
same sequence is exposed to the other processes. Since most software enginnerings neither have a fever on Physics,
nor on this crazy memory access model, architects have to save them by building a straightforward interface between
the programming model and hardware model.

Mapping the multicore memory access sequence into one timeline has already increased the reflectivity of
programmar's forehead. Having a different timeline for each process could kill most programmars on the Earth. The
voice from programmars is that they need one invariant timeline across all cores, while mantaining the original
relative order within each process. This model is called Sequential Consitency (SC).

Unfortunately this SC model is too expensive to be practical for modern architecture. This is because in most
cases, the operation of store could have a higher latency due to the cache miss and the coherence protocol. So if
there is a load operation after this store, should we stall and wait the previous store to finish? This could
dramatically hurt the parrallelism, especially those store operations are much less likely to be at the critical
path than load operations. If this load operation could bypass the store, the long latency of this store is
hidden. The hardware implementation would be having a FIFO buffer for those store operations in fly. This FIFO
have two following functions: preserving the order of store -> store , and feeding those bypassing load operations
if their address are the same. Assume we also preserve the order of load -> load, and store -> load, this model of
only breaking load -> store is called Total Store Order (TSO). From the perspective of cache, the store
operation is put off after those bypassing load finishes. However, from the core's perspective, the exact order is
preserved. This is because while those stores are waiting at the buffer, they are already visible to this core.

But the problem is that, other cores could only get the memory from the cache and their own buffer, so even if
other cores preserves their memory access order perfectly, the timeline is still skewed due to the extra store
latency from that TSO core. If they issue a load operation to the same address of that delayed store, they will
get a stale data. If programmers still try to mapping the order into one timeline, a loop will forbid them doing
so. A classic example is that: both X and Y's original data is 0. Core A first stores 1 to X and loads the Y,
while core B first stores 1 to Y while loads the X. With TSO, programmars could finally get (X, Y) = (0 ,0) if the
load bypassing happens. If we try to analysis the result with a timeline, the (0 ,0) means both load happens
the store from the other core. And since both store should happen before the load from its own load, a loop
manifests.

And don't forget that here we just relax the order of store -> load. Better performance could be attained if the
other three types of order are relaxed. So how could we avoid those unexpected results like the above example
under those reordering, while maintaining the performance benefit of reordering? Which part of reordering is safe and which might not be? Actually, if there is no memory accesses to the same address of those delayed store from
another core, then everything is fine. Who cares if the data is updated on time if we don't need them? And who
cares if the delayed operation from other core is a load whose effect is never visible to others? What we do care
is one core promises a sequence order, but later their store is delayed, or in other word being bypassed by other
loads or stores. Then from the view of other cores, this core's modified version of memory accesses sequence
distinguishes from the orignal.

One way to fix this is Weak Ordering proposed by Adve and Hill and at 1990. The idea is that we force a sequence
between two accesses from different cores that both access the same location, if at least one of them is a store.
A lock and unlock instruction is added around them, which ensures a consistent sequence of those two accesses. So
how does this work to achieve this? Firstly, no operation is allowed to bypass this lock. Secondly, the lock can only be executed after previous lock has unlocked. A disadvantage of this method is that it could be difficult to
determine whether a lock is needed or not.



</span>
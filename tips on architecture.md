# Tips on Architecture

## VIPT Cache Aliasing
A process runs in virtual address(VA), while the hardware execute memory access for every process in the
Physical Address(PA). This is because processes could run more safely, and share space with other processes
in VA. Specifically, data are stored with their PA in the memory (DRAM), while each core tries to access memory
only sending instructions with VA to the load-store queue to get what they want. So we need to first translate
VA to PA, so that we can access the exact memory location. This type of VA to PA mapping infomation is cached
in the Translation Lookaside Buffer (TLB) to accelerate this process.

Here is an emxample of how two processes (A and B) cooperate with each other by storing and loading the same 
memory, whose PA is X. Assume the VA for X in process A is XA, and in process B is XB. So when each time
process A load a value to XA, it will update the value in PA of X. When B tries to load the value from XB, it
will get the value in PA of X. Everything seems to works well for now.

But never forget about cache! Usually when we try to access a memory, we first search it in cache, which could
have multiple levels. When we fetch a memory from DRAM, we also prefer to put it in the cache for future reuse.
So how do we use VA to access cache? If want to allow multiple processes to access the cache simultaneously,
then we couldn't simply map the VA to cache location, right? Different processes could map one VA to different
PA, which could lead to multiple PAs mapping to the same cache location! However, we might also don't want to
wait the process of VA to PA translation to start our search. So we divide the cache into different sets. We
map VA to different set, and set the search pointer to this set before we know PA. After PA is fetched from
TLB, we start a second round search only in this sepecific set. This greatly reduce the search space, which
gives us better performance and energy efficiency.

The latency is reduced and no two PAs could be mistakenly treated as the same in the cache. Everything is perfect now. Or is it? Hope you still remember the example given above. Two processes could map their own VA
to the same PA, and for memory consistency, we need to update all data replicates belongs this PA when we
updating the memory using VA(either XA or XB). If XA and XB is mapped to different sets(Set SA and Set SB),
horrible things will happen! If process A first access XA, data will be loaded to SA. Then, processes be
loads a value to XB, which will update the memory location X, might leave or modify the data in SB, but it
won't update the data in SA! So when process A loads the value from XA again, it will have a hit in the SA,
which gives process A an stale value. Memory consistency is broken, oops!

So how to fix this? The first way is never allow two processes share a cache. Do a cache Flush every a context
switch happens. This seems to be super expensive. The second way and most popular way is always let memory at
the same PA be mapped into the same set. This could be achieved by only using the bits that don't need to be
translated in VA, which is the page offset. Free lunch? Not a chance. This limits the cache line size to be
smaller than or equal to the page size. As modern processor's capacity grows, while the page size stays the
same, it forces the cache associativity to grow, which is also expensive.

# Cache Aliasing (VIVT, VIPT)
<span style="font-family: 'Lucida Console';">

One day an american student was taking an exam in Cambridge and wanted to borrow an eraser. The rule is very
strict here in England, so the invigilator asked him to first prove that he didn't have one. Then he might start
taking off his pants, or trousers? Not sure which word to use now. Anyway as a noble student majoring in
architecture, I am unwilling to see such things happen, at least in our computer. So let's get rid of Aliasing,
now!

A process runs in Virtual Address (**VA**), while the hardware executes memory access for every process in the
Physical Address (**PA**). This is because processes could run more safely, and share space with other processes
in **VA**. Specifically, data are stored based on their **PA** in the memory (DRAM), while each core tries to
access memory sending only the **VA** to the load-store queue to get what they want. In order to access the exact
memory location, we need to first translate **VA** to **PA**. BTW, **VA** to **PA** mapping information is cached
in the Translation Lookaside Buffer (TLB) to accelerate this process.

Here is an example of how two processes (A and B) cooperate with each other by storing and loading memory both to
**PA** equals X. Assume the **VA** for **X** in process A is XA and in process B is XB. Each time process A loads
a value to XA, it will update the value whose **PA** is X. When B tries to load the value from XB, it will
get the updated value, since its **PA** is also X. Everything seems to works well for now.

But never forget about the cache! Usually the first place to access a memory outside the core is cache, which could
have multiple levels. When we fetch a memory from DRAM, we also prefer to put it in the cache for future reuse.
So how to access the data from the cache? Remember the core is only aware of the **VA** of the data.

So what about simply using **VA** to access the cache, which is called Virtually Indexed Virtually Tagged (VIVT)?
It seems to be the fastest way, but we might get the wrong data which shares the same **VA** but from another
process. But wait, this problem can be solved by searching with the process's index combined, right?
Theoretically, yes. But here comes **VIVT Aliasing**: two VAs from two different processes could point to the same
**PA**. The memory coherence protocol requests that we should gives all cores the illusion that all replicates of
the same data should be updated atomically, while the memory consistency protocol requests that all cores should
agrees on the same sequence of the memory updates. This **VIVT aliasing** easily breaks those two protocols. 
If you can not find all the replicates of a data by only using the **VA**, how are you gona update them atomically
and be exposed to all cores simultaneously? You couldn't.

What about using **PA** to access the cache? Well, we might also don't want to wait the process of **VA** to
**PA** translation to start our search. Pure virtual is fast but not correct, while pure physical is correct but
but slow, so we might want something hybid, which is Virtually Index Physically Tagged (VIPT). First, we divide
the cache into different sets. using **VA** to map the data into those sets. This narrows the search pointer down
to this set before we get the **PA**. After **PA** is fetched from TLB, we only search the data in this sepecific set through a match of **PA**. This greatly reduces the search space, gaining better performance and energy efficiency.

The latency is reduced and no two different data could be mistakenly treated as the same one in the cache.
Everything is perfect now. Or is it? Here finally comes the **VIPT cache aliasing**! Hope you still remember the
example given above. Two processes could map their own **VA** to the same **PA**, and for memory consistency, we
need to update all data replicates belongs this **PA** when we updating the memory with one of its corresponding
**VA** (either XA or XB). If XA and XB is mapped to different sets (Set **SA** and Set **SB**), horrible things
will happen! When process A first accesses XA, data will be loaded to **SA**. Then, process B loads a value to XB,
which will update the memory location X, might leave or modify the data in **SB**, but it won't update the data in
**SA**! This is because when we use the XB to filter the search space in the cache, **SA** will be filtered. So
when process A loads the value from XA again, it will have a hit in the **SA**, which gives process A a stale
value. Memory consistency is broken, oops!

So how to fix this? The first way is to never allow two processes to share a cache. This method actually could
fix all aliasing problem, since if all people are speaking american English, that student might never have that
precious experience. But it could be expensive to execute a cache Flush every time a context switch happens.The
second and most popular way is always to let memory at the same **PA** be mapped into the same set. This could be
achieved by only using those bits that don't need to be translated in **VA**. Since we usually take 4KB as the mapping granularity, those 12 least important bits representing the page offset will not change during the
translation, and could be used for cache set mapping. Free lunch? Not a chance. This limits the cache line size to
be smaller than or equal to the page size. As modern processor's capacity grows, while the page size stays the
same, it forces the cache associativity to grow, which is also expensive (cache associativity * cache line size =
cache capacity).

</span>
# WindowsHeap.io

Windows 10 segment(native) heap

### Definitions
- Segment:
 A segment is a large contiguous region of memory that is allocated by the system and divided into smaller blocks. A segment can have multiple pages and each page can have diffrent size.
 A segment can be either committed or decommitted, meaning that it is either backed by physical memory or not.

- Block:
A block is a small region of memory that is allocated from a segment. a block can have one of the predefined sizes, ranging from 8 bytes to 16KB. A bloc kcan be either free or busy, means that it is either available for allocation or already allocated.

- Chunk:
A chunk is a group of blocks that have the same size and are located in the same heap page. A chunk can have upto 64 blocks, depending on the block size.

- LFH:
LFH stands for Low Fragmentation Heap, which is a policy that applications can enable for their heap to reduce heap fragmentation.
Heap fragmentation is a state in which available memory is broken in certain small, non-contigous blocks, which can cause memory allocation failures.
When the LFH is enabled, the system allocates memory in certain predetermined sizes, and uses a best-fit search algo to find the most suitable block for each request

### Segment heap architecture comprises of:
- Low fragment heap ( allocation request <= 16,368 bytes)
- Variable Size allocation (allocation request for <= 128KB). uses backend to create the vs subsegment.
- Backend(LFH and VS allocation talk to backend) / (segment allocation) / allocation requests for > 128KB to 508KB.
- Large Block Allocation (Block allocation) / (allocation request > 508KB) Uses virtual memory functions provided by  NT memory manager for allocation and freeing.
- NT Memory manager 

### About segment heap
* Signature of segment heap
    ```
    0:009> dt ntdll!_SEGMENT_HEAP 235fafc0000
    +0x000 EnvHandle        : RTL_HP_ENV_HANDLE
    +0x010 Signature        : 0xddeeddee
    ``` 
* The meta-data of segment heap is seperated from the allocated memory unlike NT heap, providing security.
* Segment heap also has LFH like NT heap but their internal working is very different.
* Segment Heap is currently opt-in feature
* Windows apps are opted in by default
  * Apps from windows store
  * Microsoft Edge
* Executables are default are also opted in by default
  * csrss.exe
  * lsass.exe
  * runtimebroker.exe
  * services.exe
  * smss.exe
  * svchost.exe
* NT heap (oplder heap implementation) is still the default for traditional applications

* Open Edge in Windbg and use !heap command to check heap type
  * we will get sement heap and NT heap addresses
 
### NOTE:
even if segment heap is enabled  in a process , not all heaps created by the process will be managed by the segment heap as there are specific type of heap that
still need to be managed by the NT heap.


## Heap Creation
- If the segment heap is enabled, the heap created by heapcreate() will be managed by the segment heap unless the dwmaximumSize argument passed to it is not zero (means the heap is not growable).
- if the rtlcreateheap() API is directly used to create heap, all of the following should be true for the segment heap to manage the created heap:
 - **heap should be growable**: Flag argument passed to RtlCreateHeap() should have HEAP_GROWABLE set.
 - Heap memory should be pre-allocated: HeapBase argument passed to RtlCreateHeap() should be NULL.
 - If a Paramenter argument is passed to RtlCreateHeap(), the following paramenter fields should be set to 0/NULL: SegmentReserve, SegmentCommit, VirtualMemoryThreshold and CommitRoutine.
 - The lock argument passed to rtlCreateHeap() should be NULL.

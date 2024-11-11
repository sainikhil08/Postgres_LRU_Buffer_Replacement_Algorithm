
Changes made in these file

bufmgr.c 
freelist.c
buf_init.c
buf_internals.h

Implementation:

1. Implemented a Doubly linked list for maintaining an LRU Queue by adding next and prev fields in BufferDesc Structure to link buffers in Queue, initialised 
   next and prev fields of a bufferdesc to -2.

2. Added first and last fields to StrategyControl structure to track head and tail of an LRU Queue, initialised these values to -1.

3. When page hit occurs in bufferLookup table, we look for buffer in buffer pool and if it's in LRU queue, remove the buffer from queue and return it.

4. if page misses in bufferlookup table then on StrategyGetBuffer() we get buffer from freelist if we have any or evict a least recently used buffer
   which is head of our LRU queue.
   
5. When refcount becomes zero for a buffer we add it to tail of the LRU queue by calling a helper routine addBufferToLRU() in unPinBuffer() method.


Helper routines for LRU buffer replacement algorithm

/* it helps to add the buffer to LRU Queue 
 * it's called from unPinBuffer() when refcount of a buffer is zero
 */
void addBufferToLRU(BufferDesc *buf)


/* it removes the buffer from LRU Queue 
 * it's called from StrategyFreeBuffer() and BufferAlloc()
 * when a buffer is freed or pinned by a process respectively
 */
void removeBufferFromLRU(BufferDesc *buf)


/*
 * LRU algorithm for buffer replacement 
 * its called when a buffer needs to be evicted.
 * it returns the head of the LRU
 */
BufferDesc *LRU(BufferAccessStrategy strategy, uint32 *buf_state)
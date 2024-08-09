## Index Concurrency Control

### Overview

Concurrency control in database management systems (DBMS) is essential to ensure that concurrent operations on shared objects produce correct results. This is particularly crucial for index structures within a database, which play a key role in efficient data retrieval.

### Correctness Criteria

Concurrency control protocols need to meet certain correctness criteria to ensure the integrity and consistency of the data. These criteria can be categorized into two types:

1. **Logical Correctness**: Ensures that the operations performed by a thread produce the expected logical results.
    
    - A thread should be able to read values that it has written previously.
    - Example: If a thread writes a value `x` into an index and then reads it back, it should read `x`.
2. **Physical Correctness**: Ensures the internal integrity of the data structure.
    
    - The internal representation of the object should be sound.
    - Example: There should not be invalid pointers in the data structure that cause threads to read incorrect memory locations.

### Focus on Logical Correctness

In the context of this discussion, the focus is on the logical correctness of index operations. This involves ensuring that:

- Index operations (such as insertions, deletions, and searches) by multiple threads do not interfere with each other in a way that produces incorrect results.
- The index reflects a consistent state that would be expected if the operations were performed serially (one after another).

### Concurrency Control Protocols for Indexes

To achieve logical correctness in indexes, various concurrency control protocols can be employed:

1. **Lock-Based Protocols**: Use locks to control access to the index.
    
    - **Exclusive Locks**: Prevent other threads from reading or writing while a thread holds the lock.
    - **Shared Locks**: Allow multiple threads to read but prevent writing.
2. **Optimistic Concurrency Control (OCC)**: Assume conflicts are rare and validate transactions at commit time.
    
    - Threads proceed without locking resources and validate their operations at the end.
    - If a conflict is detected, the transaction is rolled back.
3. **Timestamp Ordering Protocols**: Assign timestamps to transactions and ensure they execute in timestamp order.
    
    - Ensures serializability by ordering transactions based on their timestamps.
4. **Multi-Version Concurrency Control (MVCC)**: Maintain multiple versions of data items.
    
    - Allows readers to access the most recent committed version without blocking writers.

- Allows readers to access the most recent committed version without blocking writers.

## Locks vs. Latches

When discussing how a Database Management System (DBMS) protects its internal elements, it is essential to distinguish between **locks** and **latches**. These two mechanisms serve different purposes and operate at different levels of the DBMS.

### Locks

**Locks** are higher-level, logical primitives used to protect the contents of a database, such as tuples, tables, or entire databases, from other transactions. Here are key characteristics of locks:

- **Duration**: Locks are held for the entire duration of a transaction. This ensures that the transaction has exclusive or shared access to the locked data until it completes.
- **Purpose**: Locks ensure the consistency and isolation properties of transactions, crucial for maintaining ACID (Atomicity, Consistency, Isolation, Durability) properties in databases.
- **User Exposure**: Database systems can expose the locks being held to users, allowing them to see which transactions are holding locks and potentially blocking other operations.
- **Rollback Capability**: Locks need to support the ability to roll back changes. If a transaction fails or is aborted, the changes made under the protection of the lock must be undone to maintain data consistency.

### Latches

**Latches** are low-level protection primitives used for critical sections of the DBMS’s internal data structures, such as regions of memory or specific data structures. Key characteristics of latches include:

- **Duration**: Latches are held only for the duration of the specific operation being performed. They are short-lived compared to locks.
- **Purpose**: Latches ensure the physical integrity of internal data structures, preventing concurrent threads from causing inconsistencies or corruption in the memory regions they protect.
- **Rollback Capability**: Latches do not need to support rollback capabilities. Their purpose is to protect data structures during operations, not to maintain transaction-level consistency.
- **Modes**:
    - **READ Latch**: Multiple threads are allowed to read the same item concurrently. A thread can acquire a read latch even if another thread already holds it.
    - **WRITE Latch**: Only one thread can access the item. A thread cannot acquire a write latch if another thread holds the latch in any mode. A write latch also prevents other threads from acquiring a read latch.



## Latch implementations 
- Latches in database management systems (DBMS) are crucial for ensuring synchronization and mutual exclusion. Given the nature of concurrent operations, latches must be both efficient and effective in preventing race conditions. Here are three common latch implementations that utilize atomic compare-and-swap (CAS) instructions:

1. **Spinlock:**
    
    - **Description:** A spinlock is a simple form of a lock where the thread repeatedly checks (spins) on the latch until it becomes available. It uses CAS to acquire the lock.
    - **Implementation:**
        
```cpp
class Spinlock {
private:
    std::atomic<bool> locked;
public:
    Spinlock() : locked(false) {}
    
    void lock() {
        while (locked.exchange(true, std::memory_order_acquire)) {
            // busy-wait (spin)
        }
    }
    
    void unlock() {
        locked.store(false, std::memory_order_release);
    }
};


        
```
- **Trade-offs:**
    - **Pros:** Simple to implement and low overhead when lock contention is low.
    - **Cons:** Can waste CPU cycles due to busy-waiting, leading to poor performance under high contention.
1. **Ticket Lock:**
    
    - **Description:** A ticket lock addresses some of the issues of spinlocks by providing a fairer locking mechanism. Each thread takes a ticket and waits for its turn to acquire the lock.
    - **Implementation:**
  
```cpp
class TicketLock {
private:
    std::atomic<int> next_ticket;
    std::atomic<int> now_serving;
public:
    TicketLock() : next_ticket(0), now_serving(0) {}

    void lock() {
        int my_ticket = next_ticket.fetch_add(1, std::memory_order_relaxed);
        while (now_serving.load(std::memory_order_acquire) != my_ticket) {
            // busy-wait (spin)
        }
    }

    void unlock() {
        now_serving.fetch_add(1, std::memory_order_release);
    }
};

```
      
        
- **Trade-offs:**
    - **Pros:** Ensures fairness as each thread is served in the order they arrive, reducing starvation.
    - **Cons:** Still uses busy-waiting, which can lead to wasted CPU cycles under high contention.
1. **MCS Lock (Mellor-Crummey and Scott Queue Lock):**
    
    - **Description:** MCS locks maintain a queue of waiting threads, which reduces the performance penalties of busy-waiting and provides scalable mutual exclusion.
    - **Implementation:**
        
```cpp
struct MCSNode {
    std::atomic<MCSNode*> next;
    std::atomic<bool> waiting;
};

class MCSLock {
private:
    std::atomic<MCSNode*> tail;
public:
    MCSLock() : tail(nullptr) {}

    void lock(MCSNode* node) {
        node->next.store(nullptr, std::memory_order_relaxed);
        node->waiting.store(true, std::memory_order_relaxed);

        MCSNode* prev = tail.exchange(node, std::memory_order_acquire);
        if (prev != nullptr) {
            prev->next.store(node, std::memory_order_release);
            while (node->waiting.load(std::memory_order_acquire)) {
                // busy-wait (spin)
            }
        }
    }

    void unlock(MCSNode* node) {
        MCSNode* next = node->next.load(std::memory_order_acquire);
        if (next == nullptr) {
            if (tail.compare_exchange_strong(node, nullptr, std::memory_order_release)) {
                return;
            }
            while ((next = node->next.load(std::memory_order_acquire)) == nullptr) {
                // busy-wait (spin)
            }
        }
        next->waiting.store(false, std::memory_order_release);
    }
};

```
        
- **Trade-offs:**
    - **Pros:** Reduces contention on the tail and scales well with the number of threads. Each thread spins on its local variable, which reduces cache coherence traffic.
    - **Cons:** More complex to implement and requires additional memory per thread.



### Blocking OS Mutex

#### Description:

A blocking OS mutex, such as the `std::mutex` in C++ or futex in Linux, is an operating system-provided mutual exclusion mechanism. It operates in two stages:

1. **User-Space Spin Latch:** Initially, a thread attempts to acquire a lightweight spin latch in user space.
2. **OS-Level Mutex:** If the spin latch is unavailable, the thread escalates to an OS-level mutex, which involves kernel intervention. If this also fails, the thread gets descheduled by the OS, waiting until the mutex becomes available.

#### Example:

Using `std::mutex` in C++:
```cpp
#include <mutex>

std::mutex mtx;

void critical_section() {
    std::unique_lock<std::mutex> lock(mtx);
    // Critical section code here
}

```

#### Advantages:

- **Simplicity:** Easy to use as it abstracts the complexity of synchronization mechanisms.
- **Reliability:** Leverages the robustness and debugging tools provided by the OS.

#### Disadvantages:

- **Performance Overhead:** High overhead due to OS scheduling and context switching, typically around 25 ns per lock/unlock operation.
- **Non-Scalable:** Under high contention, the performance can degrade significantly as threads are frequently descheduled and rescheduled by the OS.

### Test-and-Set Spin Latch (TAS)

#### Description:

A Test-and-Set (TAS) spin latch is a low-level synchronization primitive controlled by the DBMS itself. It involves a memory location that threads attempt to update using atomic compare-and-swap (CAS) instructions. If a thread fails to acquire the latch, it can either retry in a loop (busy-wait) or decide to be descheduled.

#### Example:

Using `std::atomic<bool>` in C++:



```cpp
#include <atomic>

class TASSpinLatch {
private:
    std::atomic<bool> locked;

public:
    TASSpinLatch() : locked(false) {}

    void lock() {
        while (locked.exchange(true, std::memory_order_acquire)) {
            // busy-wait (spin)
        }
    }

    void unlock() {
        locked.store(false, std::memory_order_release);
    }
};

TASSpinLatch latch;

void critical_section() {
    latch.lock();
    // Critical section code here
    latch.unlock();
}

```
#### Advantages:

- **Efficiency:** Latch/unlatch operations are fast (single instruction for lock/unlock).
- **Control:** The DBMS has more control over the latch behavior, deciding whether to retry or be descheduled.

#### Disadvantages:

- **Scalability:** Under high contention, multiple threads repeatedly executing CAS can lead to significant overhead.
- **Cache Coherence Issues:** Frequent CAS operations by multiple threads can cause cache coherence problems, leading to performance degradation.

### Comparison and Trade-offs

- **Performance:**
    
    - **OS Mutex:** Higher overhead due to kernel involvement and context switching, leading to slower performance, especially under contention.
    - **TAS Spin Latch:** Faster lock/unlock operations as they are handled in user space without kernel intervention, but performance can degrade under high contention due to busy-waiting.
- **Scalability:**
    
    - **OS Mutex:** Less scalable because of the overhead of OS scheduling and context switches.
    - **TAS Spin Latch:** Can handle low to moderate contention well but struggles with high contention due to excessive CAS retries and cache coherence traffic.
- **Complexity:**
    
    - **OS Mutex:** Easier to implement and manage since the OS handles most of the complexity.
    - **TAS Spin Latch:** More complex to implement and manage, requiring careful handling to avoid performance pitfalls.
- **Use Cases:**
    
    - **OS Mutex:** Suitable for scenarios with low to moderate contention where simplicity and reliability are prioritized over raw performance.
    - **TAS Spin Latch:** Suitable for low contention scenarios where performance is critical, and the DBMS can benefit from fine-grained control over the locking mechanism.




Latching in hash tables is a crucial aspect of ensuring safe concurrent access, particularly in multi-threaded environments. Here's a more detailed breakdown of the concepts mentioned:

### Static Hash Table Latching

Static hash tables are simpler when it comes to concurrent access because:

1. **Deterministic Traversal**: All threads traverse the hash table in a predictable manner (e.g., top-down).
2. **Single Slot/Page Access**: Threads access one slot/page at a time, reducing the complexity of potential conflicts.
3. **Deadlock-Free**: Since no two threads can hold latches that the other needs, deadlocks are avoided.
4. **Global Latch for Resizing**: When resizing the table, a global latch is taken to ensure safe resizing without concurrent modifications.

### Dynamic Hash Table Latching

Dynamic hashing (e.g., extendible hashing) introduces more complexity due to the need for updating shared state:

1. **Granularity of Latches**: Two main approaches exist for latching based on granularity:
    
    - **Page Latches**:
        - **Reader-Writer Latch**: Each page has a latch that can be held in read or write mode.
        - **Lower Parallelism**: Only one thread can write to a page at a time, though multiple threads can read.
        - **Efficiency for Single Thread**: A thread can access multiple slots in a page with one latch acquisition.
    - **Slot Latches**:
        - **Individual Slot Latches**: Each slot has its own latch.
        - **Higher Parallelism**: Multiple threads can access different slots on the same page concurrently.
        - **Increased Overhead**: More storage and computational overhead due to the need to acquire a latch for each slot.
2. **Latch-Free Linear Probing with CAS**:
    
    - **Compare-and-Swap (CAS)**: Utilizes CAS instructions to manage slot updates without explicit latches.
    - **Insertion Logic**: Attempts to insert a value by comparing and swapping a special "null" value with the desired value.
    - **Probing**: If CAS fails (indicating the slot is occupied), the thread probes the next slot until insertion succeeds.
    - **Benefits**: Reduces the need for latches, potentially increasing performance in highly concurrent environments.

### Considerations

1. **Trade-Offs**:
    
    - **Page Latches**: Better for scenarios where a single thread needs to access multiple slots quickly. Less parallelism but simpler latch management.
    - **Slot Latches**: Better for high concurrency with many threads accessing different slots. More parallelism but higher overhead.
2. **Performance Impact**:
    
    - **Page Latches**: Can lead to bottlenecks if many threads frequently access the same page.
    - **Slot Latches**: Can lead to higher latency due to the overhead of managing multiple latches.
3. **Use Cases**:
    
    - **Static Hash Tables**: Suitable for applications with predictable access patterns and less frequent resizing.
    - **Dynamic Hash Tables**: Suitable for applications that require frequent resizing and have unpredictable access patterns.

Choosing the right latching mechanism depends on the specific requirements of the application, including the expected concurrency level, access patterns, and performance considerations.




### B+Tree Latching

B+Tree latching is essential for ensuring thread-safe operations in concurrent environments. The primary challenges are:

1. **Concurrent Modifications**: Preventing multiple threads from modifying the same node simultaneously.
2. **Node Splits/Merges During Traversal**: Ensuring that a thread traversing the tree is not disrupted by another thread splitting or merging nodes.

### Latch Crabbing/Coupling Protocol

Latch crabbing or coupling is a protocol designed to allow safe concurrent access and modification of B+Trees. The basic steps are:

1. **Latch Parent**: Acquire a latch on the parent node.
2. **Latch Child**: Acquire a latch on the child node.
3. **Release Parent Latch if Safe**: A node is considered "safe" if:
    - It will not split during insertion (i.e., it is not full).
    - It will not merge during deletion (i.e., it is more than half full).

### Basic Latch Crabbing Protocol

- **Search**:
    - Start at the root.
    - Acquire a latch on the child.
    - Release the latch on the parent if the child is safe.
- **Insert/Delete**:
    - Start at the root.
    - Acquire exclusive (X) latches as needed.
    - Check if the child is safe. If safe, release latches on all ancestors.

**Note**:

- Read latches do not require the "safe" check.
- The definition of "safe" depends on the operation (insertion or deletion).

### Improved Latch Crabbing Protocol

To enhance parallelism, the improved protocol assumes that node splits/merges are rare:

- **Insert/Delete**:
    - Acquire shared (READ) latches down to the leaf node.
    - Check if the leaf node is safe.
    - If not safe, release all latches and restart using exclusive latches.

### Deadlock Considerations

- **Top-Down Latching**: Threads acquire latches from top to bottom, avoiding deadlocks because they do not hold latches on nodes above the current one.
- **Leaf Node Scans**: Potential for deadlocks exists since threads may acquire latches in opposite directions (left-to-right and right-to-left).

### Handling Deadlocks in Leaf Node Scans

Programmers must implement a "no-wait" mode for acquiring latches on leaf nodes:

- **No-Wait Mode**: If a latch on a leaf node is unavailable, the thread should immediately abort its operation, release any held latches, and restart.

### Summary

- **Basic Protocol**: Ensures safe concurrent access with minimal assumptions about node splits/merges.
- **Improved Protocol**: Optimizes for performance by assuming splits/merges are rare, enhancing parallelism by using shared latches initially.
- **Deadlock Avoidance**: Achieved by top-down latching in general tree traversal and a no-wait protocol for leaf node scans.

These protocols ensure that B+Tree operations remain thread-safe while maximizing concurrency and minimizing potential deadlocks.



# Task 06 – Deadlock and Inter-Thread Communication

**Bloom's Level:** K4 – Analyse

## Problem Context – Hostel Wallet Transfers and Mess Token Counter

**Part A:** Students transfer money between hostel wallet accounts. Each transfer locks both accounts.

**Part B:** A mess counter prepares meal tokens (producer) and students collect them (consumers) from a counter that can hold at most 5 tokens.

## Aim

To understand deadlock prevention and inter-thread communication in Java by implementing hostel wallet transfers and a producer-consumer token counter using multiple threads.

# Part A – Deadlock Analysis and Prevention

## 1. Program That Can Cause Deadlock

In a hostel wallet system, two students transfer money between two accounts.

- Thread 1 locks Account A and waits for Account B.
- Thread 2 locks Account B and waits for Account A.
- Both threads wait forever because each thread holds a lock required by the other.

The CountDownLatch makes both threads acquire their first locks before attempting to acquire the second locks. This reliably creates the circular waiting condition.

The threads are daemon threads so that the demonstration does not prevent the program from terminating.

## 2. Four Necessary Conditions for Deadlock

1. **Mutual Exclusion:** Only one thread can hold a particular account lock at a time.
2. **Hold and Wait:** A thread holds one account lock while waiting for another account lock.
3. **No Preemption:** A lock cannot be forcibly taken away from a thread.
4. **Circular Wait:** Thread 1 waits for Account B while Thread 2 waits for Account A.

All four conditions are present, resulting in deadlock.

## 3. Deadlock Prevention Using Consistent Lock Ordering

To prevent deadlock, all threads must acquire account locks in the same order.

For example, every transfer locks the account with the smaller ID first and the account with the larger ID second.

### Explanation

- Every thread locks the account with the smaller ID first.
- The account with the larger ID is locked second.
- Therefore, threads cannot form a circular waiting chain.
- This prevents deadlock while allowing multiple transfers to execute safely.

# Part B – Producer-Consumer Problem

## 1. Producer-Consumer Implementation

The mess counter can hold a maximum of 5 tokens.

- The producer prepares tokens and adds them to the counter.
- Consumers collect tokens from the counter.
- If the counter is full, the producer waits.
- If the counter is empty, consumers wait.

The producer method uses `put()` and the consumer method uses `take()`.

## 2. Why Must wait() Be Used Inside a while Loop?

The `wait()` method must be used inside a while loop because:

- A thread may wake up even without receiving the expected notification. This is called a spurious wakeup.
- Another thread may change the counter before the awakened thread gets the lock again.
- Therefore, the condition must be checked again before accessing the buffer.
- The while loop prevents inserting into a full counter or removing from an empty counter.

Example:

```java
while (count == 0) {
    wait();
}
```

## 3. Why Must wait() Be Called Inside a Synchronized Block?

- The wait() method must be called by a thread that owns the object's monitor.
- The synchronized method provides that monitor ownership.
- When wait() is called, the thread releases the monitor and enters the waiting state.
- After notification, it must reacquire the monitor before continuing.

Calling wait() outside synchronized code causes an `IllegalMonitorStateException`.

## 4. Difference Between notify() and notifyAll()

- `notify()` wakes up only one waiting thread.
- `notifyAll()` wakes up all threads waiting on the same object's monitor.
- With multiple producers and consumers, notify() may wake a thread that cannot currently proceed.
- This can leave the correct thread waiting and may cause the system to stop making progress.
- notifyAll() allows all waiting threads to recheck their conditions, so it is safer for this producer-consumer design.

### Why Use notifyAll() in This Program?

There is one producer and two consumers. With this particular arrangement, notify() may work because only the consumers wait when the buffer is empty and only the producer waits when it is full. However, notifyAll() is more reliable if the program is extended to use multiple producers or additional consumers.

## 5. Counter Size Over Time

The counter size changes according to the producer and consumer operations.

Example sequence:

- Producer adds token 1: counter size = 1.
- Producer adds token 2: counter size = 2.
- Producer adds token 3: counter size = 3.
- Consumer removes token 1: counter size = 2.
- Producer adds token 4: counter size = 3.
- Producer adds token 5: counter size = 4.
- Producer adds token 6: counter size = 5.
- Producer waits because the counter is full.
- Consumer removes a token: counter size = 4.

The counter size always remains between 0 and 5 because:

- The producer waits when count == 5.
- The consumer waits when count == 0.

## 6. Difference Between sleep() and wait()

### sleep()

- Pauses a thread for a specified time.
- Does not release the object's lock.
- Belongs to the Thread class.
- Can be called without a synchronized block.

### wait()

- Makes a thread wait until notification or interruption.
- Releases the object's monitor lock.
- Belongs to the Object class.
- Must be called inside synchronized code.

## 7. What Happens If while Is Changed to if?

Original code:

```java
while (count == 0) {
    wait();
}
```

Modified code:

```java
if (count == 0) {
    wait();
}
```

Similarly, for the producer:

```java
if (count == buffer.length) {
    wait();
}
```

### Possible Problems

- A thread may wake up without the condition becoming true.
- Another consumer may remove the available token before the awakened consumer continues.
- The awakened consumer may then attempt to remove a token from an empty counter.
- A producer may also attempt to insert into a full counter.
- This can result in incorrect data, overwritten tokens, or invalid counter values.

**Conclusion:** Using while is necessary because it checks the condition again whenever a thread wakes up. Changing it to if may cause errors, although the errors may not appear in every execution.

# Alternative Concepts

## 1. Using ArrayBlockingQueue

Instead of manually implementing wait() and notifyAll(), Java provides ArrayBlockingQueue.

It is a thread-safe bounded queue that automatically handles producer-consumer synchronization.

### Advantages

- Automatically handles synchronization.
- Prevents the counter from exceeding its capacity.
- put() waits when the queue is full.
- take() waits when the queue is empty.
- Reduces manual coding errors.

## 2. Using tryLock() with Timeout for Wallet Transfers

tryLock() attempts to acquire a lock without waiting forever.

- If the lock is available, the thread acquires it.
- If the lock is unavailable within the specified timeout, the thread can release any lock it already holds and retry.
- This helps prevent threads from waiting indefinitely.

If the second lock cannot be acquired, the first lock is released in the finally block. The transfer returns false, allowing the caller to retry or report failure.

# Time and Space Complexity Analysis

## Producer-Consumer Using wait() and notifyAll()

- put() operation: O(1) when space is available.
- take() operation: O(1) when a token is available.
- Waiting time depends on other threads and their execution.
- Space complexity: O(5), because the buffer stores at most 5 tokens.

## Producer-Consumer Using ArrayBlockingQueue

- put() operation: O(1) under normal conditions.
- take() operation: O(1) under normal conditions.
- Waiting may occur when the queue is full or empty.
- Space complexity: O(5), because the queue has a fixed capacity of 5.

## Wallet Transfer Using Synchronized Locks

- Time complexity: O(1) for a transfer involving two accounts, excluding lock waiting time.
- Space complexity: O(1) additional space per transfer.

## Wallet Transfer Using tryLock()

- Time complexity: O(1) for each lock acquisition attempt, excluding timeout and retries.
- Repeated retries may increase total execution time.
- Space complexity: O(1) additional space per transfer.

# Which Design Is Less Error-Prone?

- ArrayBlockingQueue is less error-prone for producer-consumer problems because it handles synchronization internally.
- tryLock() with timeout helps prevent indefinite waiting and allows the application to handle unsuccessful lock acquisition.
- Consistent lock ordering is also a simple and effective way to prevent deadlock.
- Manual wait() and notifyAll() require careful condition checking and synchronization.

# Joint Conclusion

Deadlock and producer-consumer problems both involve multiple threads competing for shared resources.

- Deadlock occurs when threads wait indefinitely for locks held by one another.
- Producer-consumer problems occur when threads coordinate access to a shared buffer.
- Deadlock is prevented by using a consistent lock-ordering rule.
- Producer-consumer coordination is achieved using wait(), notifyAll(), and condition checking with while loops.
- ArrayBlockingQueue provides a simpler and safer alternative for managing shared buffers.

Therefore, proper synchronization, lock management, and inter-thread communication ensure data consistency and help threads make progress without deadlock or buffer errors.

# Task 05 – Race Conditions in a Cinema Seat-Booking System

## Aim
To demonstrate race conditions in a cinema seat-booking system, understand how multiple threads can book the same seat simultaneously, and compare different synchronization techniques to prevent double booking.

## 1. Booking Without Synchronization
Multiple threads attempt to book the same seat without synchronization. Since checking and booking are separate operations, two threads may access the same seat at the same time.

### Observation
- Ten seats are available.
- One hundred users attempt to book seats.
- Ten users compete for each seat.
- Multiple threads may pass the availability check before another thread marks the seat as booked.
- Therefore, bookingsMade may become greater than 10.

The exact number of duplicate bookings can vary between runs.

## 2. Booking Using a Synchronized Method
A synchronized method allows only one thread at a time to execute that method on the same object.

### Observation
- The synchronized keyword protects the entire booking method.
- Only one thread can execute the method on the same screen object at a time.
- Other threads must wait until the current thread releases the lock.
- Therefore, duplicate booking is prevented.

## 3. Booking Using a Synchronized Block
A synchronized block protects only the critical section instead of locking the entire method.

### Observation
- The synchronized block protects the check-and-book operation.
- The lock is released automatically after the block finishes.
- Code outside the synchronized block can execute without acquiring that lock.
- It prevents duplicate booking while allowing more flexibility in method design.

## 4. Race Condition Explanation
A race condition occurs when multiple threads access and modify shared data at the same time, and the final result depends on the order of execution.

### Step-by-step example
1. Thread A checks whether seat 5 is available. It finds that the seat is free.
2. Before Thread A marks the seat as booked, Thread B checks seat 5.
3. Thread B also finds that the seat is free.
4. Thread A marks seat 5 as booked and increases the booking count.
5. Thread B also marks seat 5 as booked and increases the booking count.
6. Both threads report a successful booking for the same seat.

This results in double booking.

### Exact Critical Section
```java
if (!booked[seat]) {
    booked[seat] = true;
    bookingsMade++;
}

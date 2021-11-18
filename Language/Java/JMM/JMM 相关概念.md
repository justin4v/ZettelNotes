# Shared variables/Heap memory
- Memory that **can be shared between threads is called shared or heap memory**.
- All instance fields, static fields and array elements are stored in heap memory. 
- We use the term *variable*  to refer to both fields and array elements. 
- *Variables local to a method are never shared between threads*.

# Inter-thread Actions
- An inter-thread action is **an action performed by a thread that could be detected by or be directly influenced by another thread**. 
- **Inter-thread actions include reads and writes of shared variables and synchronization actions**, such as locking or unlocking a monitor, reading or writing a volatile variable, or starting a thread.

# Program Order
Among all the inter-thread actions performed by each thread t, **the program order of *t* is a total order that reflects the order in which these actions would be performed according to the intra-thread semantics of *t***.

# Intra-thread semantics
Intra-thread semantics are **the standard semantics for single threaded programs, and allow the complete prediction of the behavior of a thread based on the values** seen by read actions within the thread.

Simply put, **intra-thread semantics are what determine the execution of a thread in isolation**.

# Synchronization Actions
- **All inter-thread actions other than reads and writes of normal and final variables are synchronization actions**. 
- These *include locks, unlocks, reads of and writes to volatile variables, actions that start a thread, and actions that detect that a thread is done*.

# Synchronization Order 
In any execution, there is a **synchronization order which is a total order over all of the synchronization actions of that execution.** 

For each thread t, the synchronization order of the synchronization actions in t is consistent with the program order of t.

# Happens-Before relationship
Two actions can be ordered by a happens-before relationship. If one action happens before another, then the first is visible to and ordered before the second.

参见：[[Java 同步语义#Happens-Before Relationship 和 Happens-Before Order]]

# Happens-Before Edges
If we have two actions x and y, we **use $$x{\overset{hb}{\longrightarrow}}y$$ to mean that x happens before y**.
If x and y are actions of the same thread and x comes before y in program order, then $$x{\overset{hb}{\longrightarrow}}y$$

Synchronization actions also induce *happens-before edges*. We call the resulting edges--*synchronized-with edges*, and they are defined as follows:
- There is a happens-before edge from **an unlock action on monitor *m* to all subsequent lock actions on *m*(where subsequent is defined according to the synchronization order). **
- There is a happens-before edge from **a write to a volatile variable *v* to all subsequent reads of *v* by any thread (where subsequent is defined according to the synchronization order).** 
- There is a happens-before edge from **an action that starts a thread to the first action in the thread it starts. **
- There is a happens-before edge **between the final action in a thread T1 and an action in another thread T2 that detects that T1 has terminated.** T2 may accomplish this by calling T1.isAlive() or doing a join action on T1. 
- If thread T1 interrupts thread T2, there is a happens-before edge from **the interrupted by T1 to the point where any other thread (including T2) determines the T2 has been interrupted** (by having an InterruptedException thrown or by invoking Thread.interruped or Thread.isInterrupted). 


In addition, we have two other rules for *generating happens-before edges*. 
- There is a happens-before edge **from the write of the default value (zero, false or null) of each variable to the first action in every thread.** 
- Happens-before is transitively closed. In other words, if $$x{\overset{hb}{\longrightarrow}}y$$ and $$y{\overset{hb}{\longrightarrow}}z$$, then $$x{\overset{hb}{\longrightarrow}}z$$.


It should be noted that *the presence of a happens-before relationship between two actions does not necessarily imply that they have to take place in that order in an implementation*.

However, they have to appear to share an ordering to the rest of the program. If it cannot be caught, it is not illegal. 

For example, the write of a default value to every field of an object constructed by a thread need not occur before the beginning of that thread, as long as no read ever observes that fact.


# 访问冲突
Conflicting Accesses：
- **Two accesses (reads of or writes ) to the same shared field or array element are said to be conflicting if at least one of the accesses is a write.**
- 对于同一变量的两个并行的访问（读或者写）中如果至少有一个是写入操作，则称它们是冲突的。

# 数据竞争

^7a5c5d

- *there is a write in one thread*, 
- *a read of the same variable by another thread*, 
- and the write and read are not ordered by synchronization.

When a program **contains two conflicting accesses that are not ordered by a happens-before relationship**, it is said to contain a **data race**. 

# Visibility
- If an action in one thread is *visible* to another thread, then the result of that action can be observed by the second thread；
- In order to guarantee that the results of one action are observable to a second action, then the first must *happen before* the second。

```java
class LoopMayNeverEnd { 
	boolean done = false; 

	void work() { 
		while (!done) {
			// do work 
		}
	} 
	
	void stopWork() {
		done = true; 
	}
}
```

Consider the code in Figure 4. Now imagine that two threads are created, and that one thread calls work(), and at some point, the other thread calls stopWork(). 

Because there is no happens-before relationship between the two threads, the thread in the loop may never see the update to *done* performed by the other thread. 

In practice, this may happen if the compiler detects that no writes are performed to done in the first thread; the compiler may hoist the read of *done* out of the loop, transforming it into an infinite loop. 

To ensure that this does not happen, there must be a happens-before relationship between the two threads. 

In LoopMayNeverEnd, this can be achieved by *declaring done to be volatile*. Conceptually, all actions on volatiles happen in a single order, and each write to a volatile field happens before any read of that volatile that occurs later in that order.


# Ordering
Ordering constraints govern the order in which multiple actions are seen to have happened. The ability to perceive ordering constraints among actions is only guaranteed to actions that share a happens-before relationship with them.


# Atomicity
If an action is (or a set of actions are) atomic, its result must be seen to happen “all at once”, or indivisibly.

Atomicity can also be enforced on a sequence of actions. A program can be free from data races without having this form of atomicity. However, enforcing this kind of atomicity is frequently as important to program correctness as enforcing freedom from data races. Consider the code in Figure 6. Since all access to the shared variable balance is guarded by synchronization, the code is free of data races.

# Sequential Consistency
Sequential consistency is a very strong guarantee that is made about visibility and ordering in an execution of a program. Within a sequentially consistent execution, there is a total order over all individual actions (such as a read or a write) which is consistent with program order.


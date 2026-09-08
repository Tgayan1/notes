- Current Pintos Uses Round Robin Technic

- **Appendix** : Round-Robin Scheduling
	- Provide specific time period for a thread to be execute. When the time period/ time *quantum* is expired, it yields the CPU for the next Thread to be run.
	- ![[Pasted image 20260908140546.png|455]]

- On the current Setup, Interrupt Handling works Like this.
![[timer_interrupt_flow_no_priority.png|364]]

- Thread A, stops keeping the `eip`/`eflags` of the `intterupt_handler()` and eventually unwind the stack into its normal stop point of `baz()`. (like returning from a typical function)
- So, `interrupt_handler()` works within the thread stack,  just a stack frame like a function frame of a typical C program. So to get back normal execution, we need to clear that frame (+ sum other things)


### Priority Scheduling.

- `ready_list` gonna edit by basically two main functions.

1. `thread_yield()` - giving up the CPU for next thread to be run.(`RUNNING` - `READY`)

```run-c
void
thread_yield (void)
{
  struct thread *cur = thread_current ();
  enum intr_level old_level;

  ASSERT (!intr_context ());

  old_level = intr_disable ();

  if (cur != idle_thread)
    list_push_back (&ready_list, &cur->elem);
  cur->status = THREAD_READY;

  schedule ();

  intr_set_level (old_level);

}
```


2. `thread_unblock()` - makes a block thread back to ready. (`BLOCKED` - `READY`)

```run-c
void
thread_unblock (struct thread *t)

{
  enum intr_level old_level;

  ASSERT (is_thread (t));
  
  old_level = intr_disable ();
  
  ASSERT (t->status == THREAD_BLOCKED);

  list_push_back (&ready_list, &t->elem);
  t->status = THREAD_READY;

  intr_set_level (old_level);

}
```

- Using above functions, `Semaphore`,`Lock`,`Conditional Variables` etc. will eventually edit the ready list.


### Problem -> Implementing Priority Scheduling

```run-c
/** Thread priorities. */

#define PRI_MIN 0                       /**< Lowest priority. */
#define PRI_DEFAULT 31                  /**< Default priority. */
#define PRI_MAX 63                      /**< Highest priority. */
```

- In order to implement this, we need to fulfill 3 requirements. 

##### REQ-1 : When a thread is added to the ready list that has a higher priority than the currently running thread, the current thread should immediately yield the processor to the new thread

**Solution** : 
- Convert Normal Ready List into High Priority Queue. `solves selecting highest priority`
	- We can implement comparator function for select the greater one and use it in both `thread_yield()` and `thread_unblock()` for `list_insert_ordered()` method.

```run-c
bool priority_grater(const struct list_elem *a, const struct list_elem *b, void *aux UNUSED)
{
  struct thread *ta = list_entry(a, struct thread, elem);
  struct thread *tb = list_entry(b, struct thread, elem);

  return ta->priority > tb->priority;
}
```

- When we try to solve Immediate Yield, We have to consider what context the CPU currently have.

- **Appendix:** `Interrupt Context` vs `Thread Context` 
	- **Thread context** — the CPU is executing a thread's own code directly; nothing is interrupting it right now.
	- **Interrupt context** — the CPU is executing interrupt-handling code that has been layered on top of whichever thread happened to be running when the interrupt fired — using that thread's stack, with an obligation (the hardware contract) to finish and `iret` back to resume that same thread, unless a yield gets triggered along the way. 

- **If thread requires to obtain the CPU within the interrupt context (ex: waking up sleeping thread) we cannot use `thread_yield()`.** Because *within the interrupt context, if we try to switch context without letting the hardware completion on the interrupt, it will crash the OS.* 

- So, If we need to give the priority and yield CPU in `interrupt_context`, we have to use `intr_yield_on_return()` method. It will safely completes the hardware completion and then do the context switch via `schedule()`.

- **Appendix:** `intr_yield_on_return()`
	- A Boolean flag set to `TRUE`, When the interrupt context successfully completes and yielding is required within the interrupt context. It is gonna check before windup/clear the entire interrupt handling frame from the current thread. If it `TRUE`, It will perform the context switch via `shedule()`.
- So we only need add this preemption check to `thread_unblock()`. (`thread_yield()` calls by ruining thread so preemption check is not making sense).

```run-c
void thread_yield(void)
{
  struct thread *cur = thread_current();
  enum intr_level old_level;

  ASSERT(!intr_context());

  old_level = intr_disable();
  
  if (cur != idle_thread)
    list_insert_ordered(&ready_list, &cur->elem, priority_grater, NULL);

  cur->status = THREAD_READY;

  schedule();

  intr_set_level(old_level);
}

void thread_unblock(struct thread *t)
{
  enum intr_level old_level;

  ASSERT(is_thread(t));

  old_level = intr_disable();

  ASSERT(t->status == THREAD_BLOCKED);
  
  list_insert_ordered(&ready_list, &t->elem, priority_grater, NULL);

  t->status = THREAD_READY;
// compare priority with current running thread vs new unblocked thread
  if (t->priority > thread_get_priority())

  {
    if (intr_context())

    {
      intr_yield_on_return();
    }
    else
    {
      thread_yield();
    }
  }
  intr_set_level(old_level);
}

```


##### REQ2: When threads are waiting for a lock, semaphore, or condition variable, the highest priority waiting thread should be awakened first

- **Appendix**: Synchronization Primitives.
1. **Semaphores** 
	- An `unsigned` integer value with a list of `waiters`.
	- If a sharable resource need concurrency, we can assign a semaphore to that resource. 
	- Important Operations
		- `sema_down()` -> acquires the semaphore by decrementing it's value. if already acquired by a thread, put current thread into it's `waiters` list. **Don't use in interrupt context** (interrupt handler is not a `thread`)
		- `sema_try_down()` -> acquires the semaphore and returns if that acquire is success full or not via a`bool`. **Can use within interrupt context**.
			- ex: `KeyboardInterrupt` can ask to acquire a resource to write. if can't we can program to write the data temporally in a buffer in the `handler()`.
		- `sema_up()` -> Releases the semaphore by incrementing it's value
2. **Locks**
	- A **Binary Semaphore** which only capable of allocate a resource for a one thread at a time
3. **Conditional Variables**
	- If a resource need to kept locked on more that one reason, we use them.



- In order to pick the highest priority thread to give the chance, we use the same mechanism. convert waiter lists in to higher priority queues.

```run-c
void
sema_down (struct semaphore *sema)

{
  enum intr_level old_level;

  ASSERT (sema != NULL);
  ASSERT (!intr_context ());
  
  old_level = intr_disable ();

  while (sema->value == 0)
    {
      list_insert_ordered(&sema->waiters,&thread_current()->elem,priority_grater,NULL);

      thread_block ();
    }

  sema->value--;

  intr_set_level (old_level);
}
```

- Since, we have already implemented the preemption check (compares current thread with unblocked new thread) at `thread_unblock()`,we don't require to change `sema_up()`. 
- Current Pintos Uses Round Robin Technic
- **Appendix** : Round-Robin Scheduling
	- Provide specific time period for a thread to be execute. When the time period/ time *quantum* is expired, it yields the CPU for the next Thread to be run.
	- ![[Pasted image 20260908140546.png|455]]

- On the current Setup, Interrupt Handling works Like this.
![[timer_interrupt_flow_no_priority.png|544]]
- Thread A, stops keeping the `eip`/`eflags` of the `intterupt_handler()` and eventually unwind the stack into its normal stop point of `baz()`.
- So, `interrupt_handler()` works within the thread stack, as a normal thread-frame. so to get back normal execution, we need to clear that frame (+ sum other things)


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
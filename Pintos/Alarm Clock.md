#### Problem

```run-c
// timer.c
void

timer_sleep (int64_t ticks)

{

  int64_t start = timer_ticks ();

  ASSERT (intr_get_level () == INTR_ON);

  while (timer_elapsed (start) < ticks)

    thread_yield ();

}
```

- Current `timer_sleep(int64_t ticks)` only moves the callee thread in to **Ready-Queue**
```run-c
  // thread.c
static struct list ready_list;
```

- by calling `thread_yeild()`. 
- Then CPU will unnecessarily picks that process again and again and checks the condition `while (timer_elapsed (start) < ticks)` and put it back to the ready queue. Which wastes CPU cycles. (*Busy Waiting Issue*)

#### Solution

- Instead of putting sleeping thread in to READY_QUEUE, we implements SLEEP_QUEUE to put sleeping threads. 
- Since `timer_sleep()` is running with interrupts on mode ` ASSERT (intr_get_level () == INTR_ON)` (otherwise it cant track the tick count `static int64_t ticks`), we cannot use standard `malloc()` to allocate heap segments. 
- **Note:** `malloc()` cannot be continue from a stopping point safely. if it stopped while allocating space, it might corrupt the data written by the kernel or interrupt handler.
- So, We gonna use **Intrusive Linked List** structure.
- **Appendix** : Intrusive Linked List
	- Instead wrapping data or our target object (*thread* ) around Node,
	- ```run-c
	  struct Node{
		  int data;
		  Node* prev;
		  Node* next;
	  };
	  ```
	- We will introduce linking data to that object.
	- ```run-c
	  struct thread
	  {
	    /* ... object properties */

	    struct list_elem allelem; // for All threads
	    struct list_elem elem; // We are using This   
    
		/* ... object properties */        
	  };
	  ```
	  
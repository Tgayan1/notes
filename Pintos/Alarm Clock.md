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
	- Instead wrapping data or our target object (*thread* ) around Node;
	- ```run-c
	  struct Node{
		  int data;
		  Node* prev;
		  Node* next;
	  };
	  ```
	
	- We will introduce linking data to that object directly.
	- ```run-c
	  struct thread
	  {
	    /* ... object properties */

	    struct list_elem allelem; // for All threads
	    struct list_elem elem; // We are using This   
    
		/* ... object properties */        
	  };
	  ```

- In this way, We don't need `malloc()` to create `Node` object for every object that we need to insert in to linked list.

```run-c
/** List element. */

struct list_elem
  {
    struct list_elem *prev;     /**< Previous list element. */
    struct list_elem *next;     /**< Next list element. */
  };
  
/** List. */

struct list
  {
    struct list_elem head;      /**< List head. */
    struct list_elem tail;      /**< List tail. */
  };
  
/** For List Operations, See src/lib/list.h for definitions and src/lib/list.c for implementations */

```

- **Note**: *Since, Now `elem`* points out to the middle of the actual object, Pintos provides a *macro* which uses `offsetof(STRUCT,PROPERTY)` {retrieves the relative location of property to the object starting point} in `stddef.h`  for retrieve the object data/ get the pointer back to the actual object starting point.

- ```run-c
  // src/lib/list.h
  
  #define list_entry(LIST_ELEM, STRUCT, MEMBER)           \

        ((STRUCT *) ((uint8_t *) &(LIST_ELEM)->next     \

                     - offsetof (STRUCT, MEMBER.next)))
  ```

- **Appendix:** Threads

```run-c
//src/threads/thread.h

/** States in a thread's life cycle. */
enum thread_status
  {
    THREAD_RUNNING,     /**< Running thread. */
    THREAD_READY,       /**< Not running but ready to run. */
    THREAD_BLOCKED,     /**< Waiting for an event to trigger. */
    THREAD_DYING        /**< About to be destroyed. */
  };

/**Important Operations on Threads */

void thread_yield (void); /** put back callee thread in to ready queue and call shedule()*/ 

void thread_block (void); /** change the state to Blocked, and call shedule() */
void thread_unblock (struct thread *); /** change the state from Blocked to Ready */

void thread_exit (void) NO_RETURN;  /** make thread status into dying for make it clearable */

/** For more Details See src/threads/thread.h and src/thread/thread.c */  
```

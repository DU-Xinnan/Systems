## System Programming
<br>

### Processes  

**find the exit value of the child**

	int status;
	pid_t child = fork();
	if (child == -1) return 1; // Failed
	if (child > 0) { /* I am the parent - wait for the child to finish */
	    pid_t pid = waitpid(child, &status, 0);
	    if (pid != -1 && WIFEXITED(status)) {
		int low8bits = WEXITSTATUS(status);
		printf("Process %d returned %d" , pid, low8bits);
	    }
	} else { /* I am the child */
	    // do something interesting
	    execl("/bin/ls", "/bin/ls", ".", (char *) NULL); // "ls ."
	}
	

### Signals  

signals allows asynchronous signal transfer across processes  

|Name|	Default Action|	Usual Use Case|
|---|---|---|
|SIGINT|	Terminate Process (Can be caught)|	Tell the process to stop nicely
|SIGQUIT|	Terminate Process (Can be caught)|	Tells the process to stop harshly
|SIGSTOP|	Stop Process (Cannot be caught)|	Stops the process to be continued
SIGCONT|	Continues a Process|	Continues to run the process
SIGKILL|	Terminate Process (Cannot be caught)|	You want your process gone

We can send signal to rocess using `kill` command in shell:  

	>kill -SIGSTOP 403
	>kill -SIGCONT 403

we can also send the signal in `c` code using `kill` POSIX call:

	kill(child, SIGUSR1); // Send a user-defined signal
	kill(child, SIGSTOP); // Stop the child process (the child cannot prevent this)
	kill(child, SIGTERM); // Terminate the child process (the child can prevent this)
	kill(child, SIGINT); // Equivalent to CTRL-C (by default closes the process)

We can pause a process (including child process), if succeeds it won't be allocate more CPU time.
`SIGCONT` will be sent to the process when we try to resume it.

**handle Ctrl-C**  
some system calls and library are not 'async-signal-safe', for example, if the program is executing `malloc`, and we interrupt the program, then the memory structure won't be in a consistant state, if our signal handling program contains system calls that uses `malloc` (e.g. malloc), it might result in program crash and 'deadlock'

Usually, a bollean flag is set to be read occasionally as part of the normal running program.

example:

	// int pleaseStop;
	volatile sig_atomic_t pleaseStop;
	void handle_sigint(int signal) {
	    pleaseStop = 1;
	}

	int main() {
	    signal(SIGINT, handle_sigint);
	    pleaseStop = 0;
	    while (! pleaseStop) { 
		/* application logic here */ 
	    }
	    /* cleanup code here */
	}
	
`volatile` make sure compiler doesn't make unnecessary optimization, `sig_atomic_t` implies all the bits of the variable will be read and write as an 'atomic' operation.	


### Memory and Allocations

**malloc**  
it will return `NULL` when malloc fails, a program should check that before further operation.
memory in heap will remain allocated until it is freed by the same pointer

By calling `sbrk` we can increase the size of the heap

if we write multi-threaded programs, we will need multiple stacks but only ever one heap. Typically, the heap is part of `Data Segmenrt` and are placed just above code and global variables.

	void *top_of_heap = sbrk(0); // return where the current heap ends
	malloc(16384);
	void *top_of_heap2 = sbrk(0);
	printf("The top of heap went from %p to %p \n", top_of_heap, top_of_heap2);

output: `The top of heap went from 0x4000 to 0xa000`

**calloc**
similar  to malloc but it initialize the memory contents to 0, naive version of the implementation is:

	void *calloc(size_t n, size_t size)
	{
		size_t total = n * size; // Does not check for overflow!
		void *result = malloc(total);
	
		if (!result) return NULL;
	
		// If we're using new memory pages 
		// just allocated from the system by calling sbrk
		// then they will be zero so zero-ing out is unnecessary,

		memset(result, 0, total);
		return result; 
	}

The memeory returned by `sbrk` is initialized to 0, because if it doesn't do so, the current program could read information that previous program leaves and could be a security loop hole. But `malloc` doesn't initialize memory to 0 in order to have better performance

**realloc**
`realloc` could resize an existing memory allocation that was previously allocated on the heap.

 a naive implementation:

	void * realloc(void * ptr, size_t newsize) {
	  // Simple implementation always reserves more memory
	  // and has no error checking
	  void *result = malloc(newsize); 
	  size_t oldsize =  ... //(depends on allocator's internal data structure)
	  if (ptr) memcpy(result, ptr, newsize < oldsize ? newsize : oldsize);
	  free(ptr);
	  return result;
	}
	
**heap memory allocation strategy**  

|strategy name|meaning|pros&cons|
|---|---|---|
|perfect-fit strategy| finds the smallest hole that is of sufficient size| it will not evaluate all possible placements and therefore be faster|
|worst-fit strategy|finds the largest hole that is of sufficient size|targets the largest unallocated space, it is a poor choice if large allocations are required.|
|first-fit strategy|finds the first available hole that is of sufficient size|usually used in practice, Hybrid approaches and many other alternatives exist|

challenges:

- Need to minimize fragmentation (i.e. maximize memory utilization)
- Need high performance
- Fiddly implementation (lots of pointer manipulation using linked lists and pointer arithmetic)

### Memory Allocator

We can tihink of the heap memory as a list of blocks with each block contains *size* information about the block
if we have a pointer `p` that points to the start of the block, then the `next_block` will be at `((char *) p) + *(size_t *) p`,
`char *` means the pointer is calculated in bytes, `size_t` ensures the memory at `p` is read as size value. 
a

The simplist way we can do a search along all the blocks until we find one with enough size. When the block was found, we will create 2 entries in our implicit list. We can mark the lowest bit in the size as inuse.  

	// Assumes p is a reasonable pointer type, e.g. 'size_t *'.
	isallocated = (*p) & 1;
	realsize = (*p) & ~1;  // mask out the lowest bit
	 
**memory alignment**  

Many architectures expect multi-byte primitives to be aligned to some multiple of 2^n, if multi-byte primitives are not stored on a reasonable boundary (for example starting at an odd address) then the performance can be significantly impacted.

The glibc `malloc` uses the heuristic that will guarantee us to be aligned. On GNU systems, the address is always a multiple of eight on most systems, and a multiple of 16 on 64-bit systems.

To calculated how many 16-bytes units do we need:   

	int s = (requested_bytes + tag_overhead_bytes + 15) / 16

**free**
we just need to mark it as unused  

	*p = (*p) & ~1; // Clear lowest bit 
	
However, if the previous or next block are also unused, we need to coaleasce them into a single free block. In order to do that, we also store the block size at the end of the block.

**performance**  

Allocating memory is a worst-case linear time operation (search linked lists for a sufficiently large free block)

**Explicit Free Lists allocators**  
Better performance can be achieved by implementing an explicit doubly-linked list of free nodes. In that case, we can immediately traverse to the next free block and the previous free block. And we could stored the next and previous avaliable block in the block. if the links are maintained from largest to smallest, then this produces a 'Worst-Fit' placement strategy.

**Segragated Allocator**  
A segregated allocator is one that divides the heap into different areas that are handled by different sub-allocators dependent on the size of the allocation request. Sizes are grouped into classes (e.g. powers of two) and each size is handled by a different sub-allocator and each size maintains its own free list.

If there are no free blocks of size 2^n, go to the next level and steal that block and split it into two. If two neighboring blocks of the same size become unallocated, they can be coalesced back together into a single large block of twice the size

The main disadvantage of the Buddy allocator is that they suffer from internal fragmentation, because allocations are rounded up to the nearest block size. For example, a 68-byte allocation will require a 128-byte block.


### Thread (Lightweight Process)

Similar to process except there is no copying. What this allows is for a process to share the same address space, variables, heap, file descriptors and etc. `clone` is used instead of `fork`. 

**pthread**  
  include `pthread.h` AND you need to compile with -pthread (or -lpthread) compiler option.  
  to create a thread:  
  
  	int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
  
  The argument void *(*start_routine) (void *) means a pointer that takes a void * pointer and returns a void * pointer.
  
  Simple example:  
  
	#include <stdio.h>
	#include <pthread.h>
	// remember to set compilation option -pthread

	void *busy(void *ptr) {
	// ptr will point to "Hi"
	    puts("Hello World");
	    return NULL;
	}
	int main() {
	    pthread_t id;
	    pthread_create(&id, NULL, busy, "Hi");
	    void *result;
	    pthread_join(id, &result); // result will be bull because *busy returns null
	}
  
  
  **exit()** and **pthread_exit**  pthread_exit
  
  `exit()` will exit the whole process, it's the same as return in main method.  
  `pthread_exit` only exit a process. Calling pthread_exit in the the main thread is a common way for simple programs to ensure that all threads finish.
  
  Example:
  
	int main() {
	  pthread_t tid1, tid2;
	  pthread_create(&tid1, NULL, myfunc, "Jabberwocky");
	  pthread_create(&tid2, NULL, myfunc, "Vorpel");
	  exit(42); //or return 42;
	  // the two thread probably wont have time to be started
	  // No code is run after exit
	}

	int main() {
	  pthread_t tid1, tid2;
	  pthread_create(&tid1, NULL, myfunc, "Jabberwocky");
	  pthread_create(&tid2, NULL, myfunc, "Vorpel");
	  pthread_exit(NULL); 

	  // No code is run after pthread_exit
	  // However process will continue to exist until both threads have finished
	}
	
	// we can also join the thread to wait until it returns
	int main() {
	  pthread_t tid1, tid2;
	  pthread_create(&tid1, NULL, myfunc, "Jabberwocky");
	  pthread_create(&tid2, NULL, myfunc, "Vorpel");
	  // wait for both threads to finish :
	  void* result;
	  pthread_join(tid1, &result);
	  pthread_join(tid2, &result); 
	  return 42;
	}
	
**Race condition**  
 run prints out 1 7 8 8 8 8 8 8 8 10
 
	#include <pthread.h>
	void* myfunc(void* ptr) {
	    int i = *((int *) ptr);
	    printf("%d ", i);
	    return NULL;
	}

	int main() {
	    // Each thread gets a different value of i to process
	    int i;
	    pthread_t tid;
	    for(i =0; i < 10; i++) {
		pthread_create(&tid, NULL, myfunc, &i); // ERROR
	    }
	    pthread_exit(NULL);
	}

to overcome this, we can create a struct and give them their own data area  

	struct T {
	  pthread_t id;
	  int start;
	  char result[100];
	};
	struct T *info = calloc(10 , sizeof(struct T)); // reserve enough bytes for ten T structures
	pthread_create(&info[i].id, NULL, func, &info[i]);
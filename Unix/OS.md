## System Programming

<br>
###Processes
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

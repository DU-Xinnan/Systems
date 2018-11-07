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

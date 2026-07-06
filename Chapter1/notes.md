# Chapter 1: Operating system interfaces 

## The job of an OS is to share a computer among multiple programs 

This includes:
* abstracting low-level hardware, for example so programs don't need to care what type of disk is being used
* shares hardware among many programs so they appear to run at the same time
* provide controlled ways for programs to interact, share data, and work together.

An OS provides these services to programs through an interface. 
Interface neeeds to balance simplicity vs. feature quantity. Ideally, only a few mechanisms that provide much generality should be surfaced.

Unix provides a narrow interface with much generality, and has been copied by BSD, Linux, Mac, Solaris, and sort of Windows.

## Components of Unix 

* Kernel: A special program that provides services to running programs.
* processes: A running program containing the following
    * Instructions: implement the program's logic
    * Data the variables on which the logic acts
    * Stack: organizes the program's procedure calls
* System Calls: the operating system's interface to invoke kernel functionality
* Shell: Unix's command-line UI, a *User Program* that reads commands from the user and executes them.

Processes alternate between executing in user space and kernel space as they make system calls.

Kernels leverage **CPU-provided** mechanisms to ensure that each process in user space can access only its own memory. (Kernel runs with system hardware priveliges that user programs do not have.)

# Processes and memory

xv6 processes consist of
* User-space memory (instructions, data, stack)
* per-process state (private to the kernel)

xv6 time-shares processes: 
* Switches available CPUs among set of queued processes
* Saves and restores process' CPU registers when switching
* associates PID (process identifier) with each process


## Fork system call
* used by processes to create child processes
* child processes have the same memory conents as the calling process
* Fork returns child's PID to parent, returns 0 to child. 

Example: prints child's PID if a parent process.
```
int pid = fork();
    if(pid > 0){
    printf("parent: child=%d\n", pid);
```

# All xv6 system calls
Each call returns 0 for success and -1 for error unless stated otherwise.
 
* fork(): Creates a process, returns child PID.
* exit(int status) Terminate the current process, report status to wait()
* wait(int * status) Wait for a child to exit, **returns child's PID**
* kill(int pid) kill process PID
* getpid() returns the current process's PID
* sleep(int n) pause for n clock ticks
* exec(*file, *argvp[]) Load and execute a file with arguemnts, **only returns if error.**
* char* sbrk(int n): grow process's memory by n bytes. **returns start of new memory**
* open(char *file, int flags): Open a file. **Returns file descriptor**








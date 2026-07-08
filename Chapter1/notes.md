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
* write(int fd, buf, n) write N bites from buf to file descriptor **returns n*
* pipe(int p[]) Create a pipe, put read/write file descriptors in p[0] and p[1].

The exit call causes the calling process to stop executing and releases its memorry and open files. 

Child process have a *copy* of the same memory as the parent, but modifying child variables won't modify parent variables and vice versa

*Exec* replaces the calling process's memory with a new memory image loaded froma file stored in fs. The file must be in ELF format , which specifies where instructions, data, and start location. Instead of returning to the calling program, Exec begins running the instructios at the file start addres.

inside shell, fork, wait, and exec are separate lines of code to allow for optimizations such as IO redirection.

For example:
```
char *argv[2];
argv[0] = "cat";
argv[1] = 0;
if(fork() == 0) {
    close(0);
    open("input.txt", O_RDONLY);
    exec("cat", argv);
}
```
# Pipes 
A pipe is a small kernel buffer exposed to processes as a pair of file descriptors-  One for reading, one for writing. 

Pipes provide a way for processes to communicate

Pipes have 4 advantages over temporary files:
* Clean themselves up
* Can pass arbitrarily long streams of data
* Allow for parallel execution of pipeline stages
* Pipes' blocking reads and writes are more efficient than non blocking files

# File system

Filesystem provides
* Data files: uninterpreted byte arrays
* Directories: Named refferences to data files and other directories

Directories
- form a tree, starting at *root*, which is a special directory
- mkdir crreates new directories
- mknod creates new *device* file

```
mkdir("/dir");
fd = open("/dir/file", O_CREATE|O_WRONLY);
close(fd);
mknod("/console", 1, 1);
```
Files
* name is distinct from file itself
* one underlying file (called inode) can have multiple names, called *links*
* each link is an entry in a directory, each entry contains filename and reference to inode
* Inode holds metadata about a file
    * type (file, directory, device)
    * length
    * location of content on disk
    * number of links to the file

**fstat**
* retrieves info from the inode that a FD refers to
* fills in the struct fstat  

```
struct stat {
int dev; // File system’s disk device
uint ino; // Inode number
short type; // Type of file
short nlink; // Number of links to file
uint64 size; // Size of file in bytes
};
```




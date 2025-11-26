# Signals

A signal is an asynchronous event delivered by the kernel to a process. Delivery interrupts execution and forces a reaction based on the signal’s default action or the process’s installed handler.

Signals provide a direct control path inside Unix: the kernel sends an integer representing the signal, and the target process stops, continues, terminates, or runs a handler.

## Listing Available Signals

```bash
trap -l
```

```bash
kill -l
```

Both commands print all signal names with their numeric identifiers.

## Core Principles

Each process maintains an internal signal table.  
This table defines three properties per signal: ignored, default action, or custom handler.  
The kernel checks this table whenever it delivers a signal.

## Process Signal Masks

`/proc/<PID>/status` exposes the process’s signal masks. cat me to see me  
Each mask is a hexadecimal bitmap.  
Bit index `0` maps to signal `1`, index `1` to signal `2`, and so on.

### SigBlk
Signals blocked by the process. Delivery is deferred and queued.

### SigIgn
Signals ignored by the process. The kernel discards them immediately.

### SigCgt
Signals with custom handlers installed by the process.

### Example
If `SigIgn = 0000000000001000` (hex), the single set bit is at index `3`.  
Index `3` corresponds to signal `4`.  
The process ignores signal `4` (`SIGILL`).

## Common POSIX Signals

- SIGHUP (1): the kernel treats this as a session-loss event. Current systems keep the same rule: if the session that started the process ends, the kernel sends SIGHUP to all processes in that session. Default action is termination. Daemons repurpose it as a reload trigger by installing a handler.
- SIGINT (2): the kernel delivers an interrupt request. Default action: terminate. Foreground programs stop immediately unless they install a handler.  
- SIGQUIT (3): the kernel delivers a quit request. Default action: terminate and produce a core dump. Intended for debugging or forced diagnostic exits.
- SIGKILL (9): the kernel unconditionally removes the process. No handler is checked, no cleanup runs, no blocking is allowed. The process is destroyed at the kernel level. 
- SIGTERM (15): the kernel requests a graceful termination. Default action: terminate. Processes may trap it to run cleanup steps or shutdown routines before exiting. 

## Custom Handlers

The `trap` builtin binds commands to signals.  
When the shell receives a trapped signal, it runs the associated command before continuing or exiting.

## Synchronization

`wait` blocks until a specific PID ends.  
Without arguments, it waits for all child processes.  
This provides basic synchronization for background work.

## Runtime Identifiers

- `$$`: PID of the current shell  
- `$!`: PID of the most recent background job  

## Job-Control Tools

- `jobs`: list of job states  
- `bg`: resume a stopped job in the background  
- `fg`: bring a job to the foreground  

## Process Inspection

- `ps -eLf`: list processes with thread information  
- `ps -aux --forest`: tree view of process hierarchy  

## Termination Tools

- `kill -9 PID`: send SIGKILL  
- `kill -s KILL PID`: explicit signal form  
- `killall <name>`: send a signal to all matching process names  
- `kill -9 $!`: kill the last background process  


## Keyboard signals
These are common signals injected by the terminal driver, not by the shell. The kernel watches specific control characters and converts them into signals for the foreground process group.

### Core mappings

- `Ctrl+C → SIGINT`: The kernel interrupts the foreground program. Default action: terminate.
- `Ctrl+\ → SIGQUIT`: The kernel sends a quit request. Default action: terminate with a core dump.
- `Ctrl+Z → SIGTSTP`: The kernel stops the foreground process. Not a termination. The process is suspended until continued by fg or bg.

## Tasks

### 1. Analyze the script `nekoncim.sh`

#### Script
```bash
#!/bin/bash
echo "Moje PID je $$"

trap 'echo "Ja nekoncim!"' SIGINT

echo "Uspat ma stale mozes pomocou CTRL+Z"
while :
do
    echo zijem
    sleep 3
done
```

#### Analysis
The script prints its PID, installs a SIGINT trap, and enters an infinite loop that prints “zijem” every three seconds.

`trap 'echo "Ja nekoncim!"' SIGINT` overrides the default behavior of SIGINT. Normally, pressing Ctrl+C terminates a script. Here, Ctrl+C does not stop execution. Instead, the trap runs and prints “Ja nekoncim!”, then the loop continues.

SIGTSTP (Ctrl+Z) is not trapped, so the script can still be suspended. Suspension stops execution, allowing the user to terminate it manually using `kill`.

### 2. Run `nekoncim.sh` and try to stop it using `CTRL+C`

- SIGINT is delivered to the script.
- The trap executes and prints “Ja nekoncim!”.
- The infinite loop resumes.
- The script cannot be ended with Ctrl+C.
- It can be stopped with:
  - Ctrl+Z (SIGTSTP)
  - `kill -TERM <PID>`
  - `kill -KILL <PID>`

### 3. Analyze the script `losovac.sh`

#### Script
```bash
#!/bin/bash

echo "Losovac caka na vyzvu k losovaniu."

function losuj()
{
    echo -n $(($RANDOM%10+1))
}

trap 'losuj' USR1

while :
do
    sleep infinity &
    wait
    #skuste zakomentovat nasledujuce dva riadky a sledujte ps aux pri posielani signalu USR1
    kill -9 $!
    wait $! 2> /dev/null
    echo " je vylosovane cislo"
done
```
#### Analysis
The script prints a waiting message, then defines a function `losuj()` that prints a random number between 1 and 10.

`trap 'losuj' USR1` registers the function as a handler for SIGUSR1. When USR1 arrives, the function prints a number immediately.

Inside the infinite loop:

- `sleep infinity &` starts a background sleep that never ends.
- `wait` waits for *any* child process state change, normally blocking forever.
- When SIGUSR1 is delivered:
  - `wait` is interrupted.
  - The trap runs and prints the random number.
  - After the trap, the script kills the background sleep using `kill -9 $!`.
  - `wait $!` reaps the sleep process.
  - The script prints `" je vylosovane cislo"`.
  - The loop repeats with a new sleep.

If the cleanup lines (`kill -9` and the second `wait`) are removed, the script leaves behind many orphaned `sleep infinity` processes.


### 4. Run `losovac.sh` in one terminal and send it a signal from another terminal

**Terminal 1:**
```bash
./losovac.sh
```

Output:
```
Losovac caka na vyzvu k losovaniu.
```

**Terminal 2:**
```bash
ps aux | grep losovac.sh
kill -USR1 <PID>
```

Each signal produces output like:
```
5 je vylosovane cislo
8 je vylosovane cislo
3 je vylosovane cislo
```

The process:
- `wait` is interrupted by USR1.
- `losuj` prints a random number.
- Sleep is killed and reaped.
- The script prints `" je vylosovane cislo"`.

### 5. Repeat the previous task in a single terminal

Start the script in the background:
```bash
./losovac.sh &
```

The shell prints the job number and PID. `$!` contains the PID.

Send signals directly:
```bash
kill -USR1 $!
```

Same output sequence appears:
```
9 je vylosovane cislo
2 je vylosovane cislo
10 je vylosovane cislo
```

Job control replaces the need for a second terminal.  
Signals function identically.

### 6. Analyze the script `vyberac.sh`

#### Script

```bash
#!/bin/bash
echo
echo "PID vyberaca je $$"

./losovac.sh &
tmp=$!
echo "PID losovaca je $tmp"

trap 'kill -9 $tmp' SIGINT

while :
do
    #skuste zakomentovat nasledujuci sleep a zistite ci program bude losovat rychlejsie
    sleep 5
    echo
    echo "Volam losovaca..."
    kill -USR1 $tmp
    sleep 5
done
```

#### Analysis

1. **Startup and PID printing**

   ```bash
   echo
   echo "PID vyberaca je $$"
   ```

   - Prints an empty line and then the PID of the `vyberac.sh` process (`$$`).
   - This is the controller script PID.

2. **Starting `losovac.sh`**

   ```bash
   ./losovac.sh &
   tmp=$!
   echo "PID losovaca je $tmp"
   ```

   - `./losovac.sh &` starts the lottery script in the background.
   - `$!` captures the PID of the **last background process**, which is `losovac.sh`.
   - This PID is stored in `tmp` and printed.
   - From now on, `vyberac.sh` knows exactly which process it controls.

3. **Trap on SIGINT (Ctrl+C)**

   ```bash
   trap 'kill -9 $tmp' SIGINT
   ```

   - This installs a handler for SIGINT (signal 2).
   - Normally, Ctrl+C would terminate `vyberac.sh`.
   - With this trap:
     - When `vyberac.sh` receives SIGINT, it runs `kill -9 $tmp`.
     - This sends SIGKILL to `losovac.sh`.
   - After the trap, `vyberac.sh` itself continues the loop (it is not told to exit).

4. **Main control loop**

   ```bash
   while :
   do
       sleep 5
       echo
       echo "Volam losovaca..."
       kill -USR1 $tmp
       sleep 5
   done
   ```

   - Infinite loop.
   - `sleep 5`: waits 5 seconds before each trigger.
   - Prints a blank line, then `"Volam losovaca..."` (“Calling the lottery…”).
   - `kill -USR1 $tmp`: sends SIGUSR1 to `losovac.sh`.
     - This activates the trap in `losovac.sh` (its `losuj` function).
     - `losovaca.sh` prints a random number followed by `je vylosovane cislo`.
   - Final `sleep 5`: space between two draws.
   - If you comment out the first `sleep 5`, the loop calls `losovac` more often, so numbers appear faster.


### 7. Run `vyberac.sh`. Check whether sending a signal to `vyberac.sh` can stop `losovac.sh`

#### Can a signal sent to `vyberac.sh` stop `losovac.sh`?

Yes, specifically SIGINT can.

- When you press **Ctrl+C** (or send SIGINT with `kill -INT <PID_vyberac>`):
  - The kernel delivers SIGINT to `vyberac.sh`.
  - Instead of exiting, `vyberac.sh` runs the trap:
    ```bash
    kill -9 $tmp
    ```
  - This sends **SIGKILL** to `losovac.sh` using its stored PID.
  - SIGKILL cannot be ignored or trapped, so `losovac.sh` is immediately terminated.

What happens next:

- `losovac.sh` stops permanently.
- `vyberac.sh` keeps running its loop, but:
  - Further `kill -USR1 $tmp` calls will fail because the target PID no longer exists.
  - You will typically see error messages like `kill: (PID) - No such process`.

Summary:

- `vyberac.sh` acts as a controller.
- It periodically tells `losovac.sh` to draw a number via SIGUSR1.
- Sending SIGINT to `vyberac.sh` causes it to forward a SIGKILL to `losovac.sh`.
- Therefore, **a signal to `vyberac.sh` can indeed stop `losovac.sh`**, because `vyberac.sh` explicitly kills it in its SIGINT trap.


### 8. [Homework] Modify `losovac.sh` to store its PID in `/tmp/losovac.PID`
### 9. [Homework] Modify `vyberac.sh` to read the PID of `losovac.sh` from `/tmp/losovac.PID`
### 10. [Homework] Modify `losovac.sh` to create a named pipe `/tmp/losovac.PID`
S### Bonus
Ensure the exchange of at least five different signals between two processes.
For each exchanged signal, print what is happening from the perspective of the processes.
After that, print one user-defined message taken as the next argument from the script’s argument list.

# Processes, Threads, and Synchronization

## Processes

A **process** is an instance of a program in execution.  
Key properties:

- Has its own **virtual address space** (memory is isolated from other processes).
- Has its own **PID**, open file descriptors, and signal handlers.
- Switching between processes is relatively **expensive** (context switch).
- Inter-process communication (IPC) is required to exchange data.

Typical lifecycle:
1. Created with `fork()` or similar call.
2. Replaces its code with a new program using `exec()` (optional).
3. Runs until it exits, crashes, or is terminated by a signal.
4. Parent collects exit status using `wait()` / `waitpid()`.

## Threads

A **thread** is a lightweight execution unit inside a process.

- Threads in the same process:
  - Share **the same address space** (global variables, heap, code).
  - Share open file descriptors and other process-wide resources.
- Each thread has:
  - Its own **stack** and **register state**.
  - Its own thread ID (pthread_t in POSIX).

Advantages:

- Cheaper context switches compared to processes.
- Easier to share data (same memory space).

Risks:

- Data races if threads access shared variables without synchronization.
- One buggy thread can corrupt the whole process.

## Synchronization Between Processes

Because processes do not share memory by default, synchronization is done via IPC mechanisms:

- **Signals**
  - Asynchronous notifications (like you used with `losovac.sh` and `vyberac.sh`).
  - Used for control (stop, reload, wake up), not good for large data.

- **Pipes / FIFOs**
  - Unidirectional byte streams between processes.
  - Anonymous pipes: usually between parent and child.
  - Named pipes (FIFOs): special files visible in the filesystem, multiple processes can open them.
  - Good for simple producer–consumer patterns.

- **Message Queues**
  - Kernel-managed queues of discrete messages.
  - Allow sending structured messages with priorities.

- **Shared Memory**
  - Multiple processes map the same memory region.
  - Very fast, but you must add your own synchronization (semaphores, mutexes).

- **Sockets**
  - Local (UNIX domain) or network sockets.
  - Good for client-server architectures and distributed systems.


## Synchronization Inside a Process (Threads)

Threads share memory, so they need synchronization when accessing shared data:

- **Mutex (mutual exclusion)**
  - Only one thread can lock a mutex at a time.
  - Protects critical sections.

- **Semaphores**
  - Counting resource control (e.g., N available slots).
  - Can be used across processes (POSIX semaphores) or within a single process.

- **Condition Variables**
  - Threads wait for a condition to become true.
  - Always used together with a mutex and a predicate.

- **Barriers**
  - All threads wait until everyone reaches the same point.

---

# Provided C Files and Build System

We will work with the following sources:

- `1.c`
- `2.c`
- `3.c`
- `4.c`
- `5.c [Homework]`
- `6.c [Homework]`
- `7.c [Homework]`
- `8.c [Homework]`
- `os_base.c [Homework]` 
- `os_base.h [Homework]`
- `Makefile`

---

## Makefile Explanation

```make
SOURCES = $(wildcard [1-9].c)
BINS = $(patsubst %.c, %.exe, $(SOURCES))

%.exe : %.c os_base.c os_base.h
	@echo "building $@"
	@gcc os_base.c $< -lpthread -o $@

.PHONY: all
all : $(BINS)
	@

.PHONY: clean
clean:
	@rm *.exe
```
#### SOURCES
```
SOURCES = $(wildcard [1-9].c)
```
`wildcard` expands to all files named `1.c` through `9.c`.
Example: `1.c 2.c 3.c 4.c 5.c 6.c 7.c 8.c`.

#### BINS
```
BINS = $(patsubst %.c, %.exe, $(SOURCES))
```
`patsubst` converts every `X.c` into `X.exe`.
Example: `1.c` → `1.exe`.

#### Build Rule
```
%.exe : %.c os_base.c os_base.h
	@echo "building $@"
	@gcc os_base.c $< -lpthread -o $@
```
Pattern rule describing how to compile each `.exe`:

- Target: `%.exe`
- Depends on:
  - corresponding source file `%.c`
  - `os_base.c`
  - `os_base.h`
- `$@` is the target file (e.g., `1.exe`)
- `$<` is the matched `.c` file (e.g., `1.c`)
- Compilation step:
  - Builds `os_base.c` + the matching `X.c`
  - Links with pthread (`-lpthread`)
  - Produces the executable.

`@` suppresses printing of the command itself.

#### all Target
```
all : $(BINS)
```
Building `all` builds every `*.exe` derived from `*.c`.

#### clean Target
```
clean:
	@rm *.exe
```
Deletes all compiled executables.  
`@` hides the `rm` command from being echoed.

#### purpose of Makefile
It automates building the binaries, and to removes the need to run gcc manually for every file.



## 1.c

```c
#include <stdio.h>
#include <sys/syscall.h>
#include <unistd.h>

//man syscall
//man fork
//man getpid

#define gettid() syscall(SYS_gettid)

int main (int argc, char * argcv[])
{
    int indefinite = argc > 1;
    pid_t pid = fork();
    if (pid == 0)
    {
        do {
            printf("executing child:  pid - %d; tid - %ld\n", getpid(), gettid());
            sleep(3);
        } while(indefinite);
    }
    else
    {
        do {
            printf("executing parent: pid - %d; tid - %ld\n", getpid(), gettid());
            sleep(2);
        } while(indefinite);
    }

    return 0;
}
```

---

### Purpose of the program

- Demonstrate how `fork()` creates a **new process**.
- Show the relationship between:
  - **PID** (process ID)
  - **TID** (thread ID)
- Show how the scheduler may interleave output from parent and child.
- Give a simple example of using a **system call** (`syscall`) to obtain the thread ID.

If you start the program without arguments:

```bash
./1.exe
```

it runs **one iteration** in both parent and child and then exits.

If you start it with any argument:

```bash
./1.exe x
```

`indefinite` becomes `1` and both processes print in a loop until you kill them.

---

#### What is a syscall?

A **system call** is the controlled entry point from user space into the kernel.

- Normal function calls stay inside your process.
- A `syscall` switches the CPU into kernel mode and requests a service from the OS
  (create a process, read a file, get time, etc.).
- On Linux, many low-level functions (like `gettid()`) are exposed only as syscalls.

In this code:

```c
#define gettid() syscall(SYS_gettid)
```

- `syscall` is a generic wrapper.
- `SYS_gettid` is the numeric ID of the `gettid` system call.
- `gettid()` here is a macro that directly asks the kernel for the current thread ID.

---

#### What is `fork()`?

`fork()` creates a **new process** by duplicating the calling process.

- The original process becomes the **parent**.
- The new process becomes the **child**.
- Both processes continue from the **same place** in the code (just after `fork()`).

Return values:

- In the **parent**, `fork()` returns the **child’s PID** (`pid > 0`).
- In the **child**, `fork()` returns `0`.
- On error, `fork()` returns `-1` in the parent and no child is created.

Usage in this program:

```c
pid_t pid = fork();
if (pid == 0) {
    // child path
} else {
    // parent path
}
```

This splits execution into two flows:
- One for the parent process.
- One for the child process.

#### Why use `fork()`?

Example reasons:

- Parallelize work: each process handles one part of the problem.
- Servers: parent accepts connections, then `fork()`s a child process for each client.
- The OS itself starts with an initial process and then **forks** new processes during boot
  (e.g. `init` and its descendants).

Processes are **isolated**:
- Each has its own address space.
- One process cannot directly modify another’s memory.

---

#### What is `getpid()`?

`getpid()` returns the **process ID** of the caller.

- Always unique per running process.
- Same in all threads within the same process.

In the code:

```c
printf("... pid - %d ...\n", getpid(), ...);
```

This prints the PID of:
- The parent process in the parent path.
- The child process in the child path.

---

#### What is `gettid()` here?

On Linux, each thread has a **thread ID** (TID).

- The program defines:
  ```c
  #define gettid() syscall(SYS_gettid)
  ```
- This calls the `gettid` syscall and returns the **thread ID** of the current thread.

In **this program** there is **exactly one thread per process**.  
So:

- PID == TID for each process.
- Output shows identical values for PID and TID in both parent and child.

This matches the note:

> Pid a Tid su rovnake, kazdy ma len jeden thread ... Takze je rovnaky ako proces id  
> (PID and TID are the same because each process has only one thread.)

If you had multiple threads inside one process, `getpid()` would be the same for them,
but `gettid()` would differ per thread.

---

## 2.c 

```c
#include <pthread.h>
#include <stdio.h>
#include <sys/syscall.h>
#include <unistd.h>

#define gettid() syscall(SYS_gettid)
int indefinite;

void* child(void *args)
{
    do {
        printf("executing child: pid - %d tid - %ld\n", getpid(), gettid());
        sleep(3);
    } while(indefinite);
}

int main (int argc, char * argcv[])
{
    pthread_t p1;
    pthread_create(&p1, NULL, child, NULL);
    indefinite = argc > 1;

    do {
        printf("executing parent: pid - %d tid - %ld\n", getpid(), gettid());
        sleep(2);
    } while(indefinite);

    pthread_join(p1, NULL);

    return 0;
}
```

---

### Purpose of the Program

This program demonstrates:

- Creating a **thread** inside a process using pthreads.
- Showing that all threads share the **same PID**.
- Showing that each thread has a **different TID** (Lightweight Process ID).
- Demonstrating concurrency inside one process.
- Showing how shared global variables can cause race conditions.

---

### pthread Basics

`pthread_create()` is used to create a new thread:

```c
pthread_t p1;
pthread_create(&p1, NULL, child, NULL);
```

It provides:

- `pthread_t` – a handle used to refer to a thread.
- Ability to specify:
  - thread attributes (here `NULL` meaning default)
  - the function to execute (`child`)
  - one argument passed to the thread (`NULL` here)

Thread functions must be of type:

```c
void* function_name(void *arg)
```

This matches the signature required by `pthread_create()`.

---

### PID vs TID

Inside the program:

```c
printf("pid - %d tid - %ld\n", getpid(), gettid());
```

Important facts:

- All threads share **one PID**.
- Each thread has a unique **TID**.
- `gettid()` is accessed using a **syscall**, because glibc does not provide a wrapper.

This matches the note:

> Parent has some PID… child has the same PID…  
> Threads have different TID… each thread is a lightweight process (LWP).

You can view thread IDs using:

```
ps -eLf
```

`LWP` column = thread ID.

---

### Global Variables and Race Conditions

Global variable:

```c
int indefinite;
```

Shared among **all threads**.

Risk:

- The child thread begins executing **immediately** after `pthread_create()`.
- The main thread sets:

```c
indefinite = argc > 1;
```

but this happens *after* thread creation.

Because globals default to zero, the child thread may see:

- `indefinite == 0` → loop runs once and exits.

This is why the note says:

> Add sleep(3) after pthread_create and child may run only once.

To make this safe, you would need synchronization (mutexes, semaphores, barriers).

---

### Scheduler Behavior

The parent thread sleeps 2 seconds, child sleeps 3 seconds.  
There is no guaranteed order. The scheduler may output:

```
executing parent...
executing child...
executing parent...
executing parent...
executing child...
```

Execution order is nondeterministic.

---

### Thread Cleanup: `pthread_join`


At the end:

```c
pthread_join(p1, NULL);
```

This ensures:

- Main thread **waits** until the child thread finishes.
- The process does not exit prematurely.
- All threads finish properly.

Notes:

> Without join, the program could exit while other threads are still running.

Ctrl+C ends the process and all its threads.

---

### Observing Threads

### `ps aux --forest`

Shows one process, but inside it threads are visible depending on ps version.

### `ps -eLf`

Shows:

- PID: shared by all threads.
- LWP: unique thread ID.
- THREAD count grows to two: main thread + child thread.

---

### Summary

`2.c` demonstrates:

- Creating threads with POSIX pthreads.
- Shared PID, separate TIDs.
- Concurrent execution inside one process.
- Risks of shared global variables.
- Importance of `pthread_join`.
- How to observe threads using ps.


## 3.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int cnt = 0;
int indefinite = 0;

void printed(int num)
{
    printf("hello from child process ");
    printf("(%d) ", getpid());
    //fflush(stdout);  //skus odkomentovat :)
    //sleep(1);
    printf("num %d, cnt %d ", num, cnt);
    printf("\n");
    cnt = cnt+1;
    sleep(5);
}

int main (int argc, char * argv[])
{
    int number = atoi(argv[1]);
    int indefinite = argc > 2;
    for (int i = 0; i < number; i++)
    {
        pid_t pid = fork();
        if (pid == 0)
        {
            do {
                printed(i);
            } while(indefinite);
            return 0; //???
        }
    }

    printf("Zomrel som\n");
    return 0;
}
```

---

### High-level behavior

- Global variables:
  - `cnt` – per-process counter, starts at 0.
  - `indefinite` – global, but **shadowed** by local `indefinite` in `main`.
- `main` expects **at least one argument**: `argv[1]` is converted to `number`.
- The program calls `fork()` in a `for` loop, creating `number` **child processes**.
- Each child calls `printed(i)`:
  - once, if there is only one argument
  - in an infinite loop, if there are at least two arguments
- The parent exits the loop, prints `Zomrel som` (“I have died”) and terminates.

---

### Argument handling and segfault

```c
int number = atoi(argv[1]);
```

If you run:

```bash
./3.exe
```

there is **no** `argv[1]`. Accessing it is out of bounds:

- `argv[1]` points to an invalid address.
- `atoi()` reads from memory that is not yours.
- Result: **segmentation fault**.

This matches the note:

> Run without arguments → segfault.  
> We accessed memory we do not own (invalid address).

Correct usage:

```bash
./3.exe <count>          # finite run
./3.exe <count> anything # infinite run in children
```

---

### Finite run: one argument

Example:

```bash
./3.exe 3
```

- `number = 3`, `argc = 2`, so local `indefinite = 0`.
- Loop `for (i = 0; i < 3; i++)`:

For each `i`:

```c
pid_t pid = fork();
if (pid == 0) {
    do {
        printed(i);
    } while(indefinite);
    return 0;
}
```

- `fork()` duplicates the process.
- In the **child** (`pid == 0`):
  - It calls `printed(i)` exactly **once** because `indefinite == 0`.
  - Then `return 0;` ends the child process.
- In the **parent**:
  - `pid > 0`, so it skips the `if` body and continues the loop.

After the loop ends:

```c
printf("Zomrel som
");
return 0;
```

- Only the **parent** executes this line.
- Parent prints `Zomrel som` and exits.
- Child processes only print from `printed`, never `"Zomrel som"`.

#### Why did the shell prompt appear before the child output?

Sequence:

1. Parent finishes the loop quickly.
2. Parent prints `Zomrel som` and exits.
3. Shell (`bash`) gets the terminal back and prints a new prompt.
4. Meanwhile, the children are still running, printing their messages with `sleep(5)`.

This matches:

> The parent died, passed the terminal back to bash,  
> then the child processes continued printing.

---

### What happens inside `printed()`

```c
void printed(int num)
{
    printf("hello from child process ");
    printf("(%d) ", getpid());
    //fflush(stdout);
    //sleep(1);
    printf("num %d, cnt %d ", num, cnt);
    printf("
");
    cnt = cnt+1;
    sleep(5);
}
```

Per child process:

- Prints:
  - static text
  - its PID
  - `num` = index `i` from the loop
  - `cnt` = value of the **child’s own** `cnt`
- Increments `cnt` after printing.
- Sleeps 5 seconds.

Key point:

- `cnt` is **global per process**, but after `fork()` each child gets its **own copy**.
- In the finite case (`indefinite == 0`):
  - `printed` is called once per child.
  - Each child prints `cnt` as `0`, then increments to `1`, but never prints again.
  - You never see `cnt` change on screen.

This answers:

> Will `cnt` increment for itself only, or influence others?

It increments **only inside that child process**.  
No child sees another child’s changes.

---

### Infinite run: two or more arguments

Example:

```bash
./3.exe 3 loop
```

Now:

```c
int indefinite = argc > 2;
```

- `argc = 3`, so local `indefinite = 1`.
- Each child executes:

```c
do {
    printed(i);
} while(indefinite);
```

with `indefinite == 1`, so they loop **forever**.

Behavior:

- Parent still exits after the loop and prints `Zomrel som`.
- Children keep printing in 5-second intervals.
- `cnt` is per-child, so each child’s `cnt` increments in its own memory.

#### Why “I cannot kill it with Ctrl+C”?

After the parent exits:

- Children continue running in the background.
- The shell has regained control and may have its own foreground process group.
- Ctrl+C now sends SIGINT to the **shell’s** foreground group, not to those orphaned children.
- The children often do **not** receive SIGINT from your terminal anymore.

Result:

- You cannot stop them with a simple Ctrl+C.
- You must use `kill` or `killall`:

```bash
ps aux --forest   # see the tree
killall 3.exe     # kill all 3.exe children
# or
kill -9 PID1 PID2 PID3 ...
```

This matches:

> The parent died, children got a replacement parent (PID 1).  
> You need `killall` or manual `kill -9` to remove them.

---

### Orphaned children and new parent

When the parent process exits:

- Its still-running children become **orphans**.
- The kernel reassigns them to a system process (traditionally PID 1).

This matches the note:

> Parent PID becomes 1 because the original parent died.  
> The system gives them a new parent.

---

### Shadowing of `indefinite`

Global:

```c
int indefinite = 0;
```

Local in `main`:

```c
int indefinite = argc > 2;
```

- The local `indefinite` **shadows** the global one inside `main`.
- `printed()` uses the **global** `indefinite`, but it never reads it.
- The loop in child uses the **local** `indefinite` on the stack at the time of `fork()`.

So the control flag used in the `do { ... } while(indefinite);` inside the child is:

- A **copy** of the local `indefinite` at fork time.
- Not the global one, and not shared between parent and child.

---

### Buffering and `fflush(stdout)`

The commented line:

```c
//fflush(stdout);  //skus odkomentovat :)
```

matters because:

- `printf` output is **buffered**.
- Without `fflush(stdout)` and with `sleep()`, buffers may not flush immediately or may be duplicated across processes after `fork()`.

Uncommenting `fflush(stdout)` makes sure each line is pushed to the terminal immediately, making interleaving more visible and consistent.

---

### Summary

`3.c` shows:

- How `fork()` in a loop creates multiple child processes.
- How each child gets its own **copy** of global variables (`cnt`, `indefinite`).
- Why arguments must be validated (segfault with missing `argv[1]`).
- How the parent can exit early while children continue running.
- How orphans are re-parented and why Ctrl+C may no longer stop them.
- How to use `ps` and `killall`/`kill` to clean up remaining child processes.




## 4.c
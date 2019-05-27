---
title: Characterizing the behavior of Python multiprocessing pools
layout: post
use_code: true
use_toc: true
excerpt: The behavior of Python multiprocessing pools depends on largely on the target functions.
last_updated: May 27, 2019
---

- [Background](#background)
  - [Python's `multiprocessing` package](#pythons-multiprocessing-package)
  - [Calling system commands in Python](#calling-system-commands-in-python)
  - [Process groups and signals](#process-groups-and-signals)
- [Behavior of `multiprocessing.Pool`](#behavior-of-multiprocessingpool)
  - [Memory footprint](#memory-footprint)
  - [Handling `SIGINT` signals](#handling-sigint-signals)
  - [`pool.terminate()`](#poolterminate)
- [Summary](#summary)
- [Appendix](#appendix)
  - [`multiprocessing.Pool` versus `multiprocessing.pool.Pool`](#multiprocessingpool-versus-multiprocessingpoolpool)
  - [Life of a pool process](#life-of-a-pool-process)
  - [Obtaining process (group) IDs and sending signals](#obtaining-process-group-ids-and-sending-signals)
  - [`multiprocessing.dummy` module](#multiprocessingdummy-module)
  - [Topics not discussed (may be added later!)](#topics-not-discussed-may-be-added-later)

# Background

## Python's `multiprocessing` package

One of the most useful APIs introduced by Python's `multiprocessing` package is the process [`Pool`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.pool.Pool), which "offers a convenient means of parallelizing the execution of a function across multiple input values, distributing the input data across processes (data parallelism)." [[`multiprocessing` documentation](https://docs.python.org/3/library/multiprocessing.html)]

By creating child processes, the `multiprocessing` package sidesteps the [Global Interpreter Lock](https://docs.python.org/3/glossary.html#term-global-interpreter-lock) and allows full utilization of multiple processors. However, depending on the target function, memory footprint can be significant, and achieving desired behavior over process and pool termination can be tricky.

## Calling system commands in Python

This post refers collectively to binary executables (such as `ls` or `wget`) and [shell builtins](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html) (such as `pwd`) as "system commands." Several standard ways of calling system commands in Python are as follows:
  - [`os.system(<command>)`](https://docs.python.org/3/library/os.html#os.system)
    - Calls the `system()` system call, which according to the [Linux man-pages](http://man7.org/linux/man-pages/man3/system.3.html),
      > uses fork(2) to create a child process that executes the shell command specified in command using execl(3) as follows:
      >  ```
      >  execl("/bin/sh", "sh", "-c", command, (char *) NULL);
      >  ```
      >  `system()` returns after the command has been completed.
      >  
      >  During execution of the command, `SIGCHLD` will be blocked, and `SIGINT` and `SIGQUIT` will be ignored, in the process that calls `system()`. (These signals will be handled     according to their defaults inside the child process that executes command.)
  - [`os.exec*()`](https://docs.python.org/3/library/os.html#os.execl) family of functions
    - Call the corresponding `exec*()` system calls.
  - [`subprocess.Popen(<command>, **kwds)`](https://docs.python.org/3/library/subprocess.html#popen-constructor)
    - Implemented using [`fork()`](https://github.com/python/cpython/blob/1b85f4ec45a5d63188ee3866bd55eb29fdec7fbf/Modules/_posixsubprocess.c#L686) and [`execv()`/`execve()`](https://github.com/python/cpython/blob/1b85f4ec45a5d63188ee3866bd55eb29fdec7fbf/Modules/_posixsubprocess.c#L511) system calls.
    - Nonblocking constructor that returns a `subprocess.Popen` object immediately, which can be waited upon using `subprocess.Popen.wait()`.
    - The `<command>` can be executed directly or through a shell by passing the `shell=True` argument.
    - `subprocess.Popen` objects *cannot* be picked.
  - [`subprocess.run(<command>, **kwds)`](https://docs.python.org/3.7/library/subprocess.html#subprocess.run)
    - Calls `subprocess.Popen()` and blocks until the command finishes.
    - Returns a [`CompletedProcess`](https://docs.python.org/3.7/library/subprocess.html#subprocess.CompletedProcess) object, which *can* be pickled.

## Process groups and signals

In Python, child processes created using `multiprocessing` package or any of the methods above (with default arguments) for calling system commands will inherit the process group ID of the parent Python process. `subprocess.Popen()` and `subprocess.run()` additionally support a `subprocess.CREATE_NEW_PROCESS_GROUP` flag that specifies "that a new process group will be created." [[`subprocess` documentation](https://docs.python.org/3/library/subprocess.html)] By default, Python translates `SIGINT` signals into `KeyboardInterrupt` exceptions. [[`signal` documentation](https://docs.python.org/3/library/signal.html)]

In shells (e.g., `sh` and `bash`), child processes' process group IDs depend on whether the shell is run interactively (i.e., in a terminal) or not (e.g., with the `-c <command>` option). When run by an interactive shell, child processes are assigned their own unique process group IDs. When run by a non-interactive shell, child processes inherit the process group ID of the shell. In either mode, the shell catches and handles `SIGINT` signals such that it waits for the current command to finish and "breaks out of any executing loops." [[Bash manual](https://www.gnu.org/software/bash/manual/html_node/Signals.html)]

Signals generated by keyboard interrupts (e.g., `SIGINT`, Ctrl+C; `SIGTSTP`, Ctrl+Z; `SIGQUIT`, Ctrl+\\) are sent to the foreground process group. [[Wikipedia](https://en.wikipedia.org/wiki/Process_group)]

# Behavior of `multiprocessing.Pool`

For brevity and consistency, we will use the following conventions and assumptions:
- `mp` refers to the `multiprocessing` package (e.g., assume that the line `import multiprocessing as mp` has been run).
- "Parent process" refers to the Python process that created the pool (i.e., the process in which `pool = mp.Pool()` was run).

## Memory footprint

On Unix-based systems, the `multiprocessing` package defaults to the `'fork'` method to start processes. Depending on when garbage collection routines are run, the physical memory occupied by the child processes may be quite high. Memory footprint of child processes can be significantly reduced by using the `'spawn'` start method (`mp.set_start_method('spawn')`), which launches each child process as a fresh Python interpreter that only inherits resources as necessary.

Upon initialization (e.g., in the constructor), a `pool` object creates a pool of processes, each of whose target is the [`mp.pool.worker()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L93) function, which waits until a task (consisting of a function to run and arguments to call it with, such as provided to `pool.apply_async()`) arrives in a task queue. We consider memory footprint with 3 types of task functions:

1. Does not call the `fork()` or `exec()` family of system calls.
   - Example: `pool.apply_async(time.sleep, (10,))`
   - Process hiearchy
     - parent process
       - pool process
   - Memory footprint depends on the process start method (`'fork'` or `'spawn'`).
2. Uses the `fork()` system call to create a child process to run the actual command.
   - Example: `pool.apply_async(os.system, ('sleep 10',))`
   - Process hiearchy
     - parent process
       - pool process
         - [shell process]
           - command process
   - Memory footprint is relatively high and depends in part on the process start method. A Python pool process, and possibly a shell process, is required to run alongside the desired command process.
3. Uses the `exec()` family of system calls *before/without calling `fork()`* to run the actual command
   - Example: `pool.apply_async(os.execvp, ('sleep', ('10',)))`
   - Process hiearchy
     - parent process
       - pool process (whose memory space, originally occupied by a Python process, has been cannibalized to run the command)
   - Memory footprint is low, since there are no additional Python or shell processes required to run alongside the desired command process.

## Handling `SIGINT` signals

Consider the following set of processes:
- parent process (Python interpreter)
- idle pool process (idle, waiting for a task)
- active pool process (running task function)
- shell process (may or may not exist, depending on the task function)
- system command process (may or may not exist, depending on the task function)

We then consider sending a `SIGINT` signal to any one process. For the parent process, idle pool process, shell process, and system command process, the behavior is the same regardless of the task function:
- Parent process: The Python interpreter raises a `KeyboardInterrupt` exception, which usually just means that the prompt is cleared and `KeyboardInterrupt` is printed to the terminal. The `pool` is not affected, since the various threads (worker handler, task handler, and result handler) managing the pool are not stopped, in part because [there is no direct way to stop a thread in Python](https://stackoverflow.com/a/325528).
- <a name="SIGINT_idle_pool_process"></a>Idle pool process: The process exits with [exitcode 1](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L310). The pool worker handler then repopulates the pool with a new process.
  - Details: The `mp.pool.worker()` function does not handle any `KeyboardInterrupt` exceptions raised while the pool process is [blocked (idling) waiting for a task](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L110), so the exception is passed up one stack frame to the pool process object's [`_boostrap()` method](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L297), which catches the `KeyboardInterrupt`, prints out a traceback, and returns. The next stack frame up is the `_launch()` method of a `Popen` object (note: the `Popen` classes defined in the `mp` package and the `subprocess.Popen` class are distinct) created by the pool process object. Within `_launch()`, `os._exit()` is called, killing the pool process.
- Shell processes: The shell waits for any currently executing system command to finish, then breaks out of any executing loops.
- System command processes: The behavior of the system command upon receiving a `SIGINT` signal will depend on whether a signal handler was installed, either by the system command program, or by the [`trap` shell builtin](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html).

For the active pool process, the behavior depends on the task function:
- A Python function that does not call the `fork()` or `exec()` family of system calls.
  - Unless a `SIGINT` handler was installed, the task function is interrupted, and the pool process puts a result value of [`(False, KeyboardInterrupt)`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L125) in the pool results queue. The pool process then waits for the next task if it has not yet reached its `maxtasks` limit, or exits otherwise (and is replaced with a new pool process by the pool worker handler).
- `os.system()`
  - The `SIGINT` signal is ignored (queued) until the `system()` system call returns, after which the pool process puts a result value of [`(False, KeyboardInterrupt)`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L125) in the pool results queue. The pool process then waits for the next task if it has not yet reached its `maxtasks` limit, or exits otherwise (and is replaced with a new pool process by the pool worker handler).
- `subprocess.run()`
  - The pool process exits before placing any result in the pool results queue and is replaced with a new pool process by the pool worker handler. If `subprocess.run()` was called with the default `shell=False` argument, then the system command is killed. If the argument `shell=True` was set, then the shell is killed, but the system command continues to run.
    - Details: It is unclear why no result is added to the pool results queue. `subprocess.run()` blocks until the system command exits, or an exception is raised. In the [case of a `KeyboardInterrupt` exception](https://github.com/python/cpython/blob/v3.7.3/Lib/subprocess.py#L480), the system command is killed, and the `KeyboardInterrupt` exception is re-raised (so `subprocess.run()` ends but never returns). The pool process should [catch this exception](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L122) and add `(False, KeyboardInterrupt)` to the pool results queue, but that is not the observed behavior.
- `subprocess.Popen()`
  - The pool process exits and is replaced with a new pool process by the pool worker handler. Any shell and/or system command continues to run.
    - <a name="subprocess_popen_result"></a>Details: Since `subprocess.Popen()` is nonblocking, it immediately returns, and the pool process tries to put the `subprocess.Popen` object into the pool results queue. However, `subprocess.Popen` objects cannot be pickled, so the pool process puts a `multiprocessing.pool.MaybeEncodingError` exception in the pool results queue and waits for the next task if it has not yet reached its `maxtasks` limit, or exits otherwise (and is replaced with a new pool process by the pool worker handler). Any `SIGINT` signal is likely to be received by the pool process when it is waiting for its *next* task. The behavior is then analogous to that of an [idle pool process receiving a `SIGINT` signal](#SIGINT_idle_pool_process).
- `os.exec*()`
  - The pool process becomes the system command process, so the behavior is defined by the signal handler (if any) installed by the system command. Assuming the default signal handler, the system command exits. The pool is repopulated by the worker handler.

Note that keyboard interrupts (e.g., Ctrl-C) are sent to the entire process group, and the overall behavior can be understood as if `SIGINT` signals are sent to each process as discussed above.

## `pool.terminate()`

[`pool.terminate()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L544) calls the [`terminate()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L119) method of each pool process, sending them `SIGTERM` signals. As before, we consider sending the effect of `pool.terminate()` on any one process.

- Parent process: not relevant / no effect
- Idle pool process: exits and is not replaced
- Active pool process: exits without putting any result in the pool results queue, and is not replaced
  - The only exception is if the task function is `subprocess.Popen()`, in which case the pool process will [put a result with an exception](#subprocess_popen_result) in the pool results queue unless `pool.terminate()` is run before the pool process finishes its nonblocking creation of a subprocess.
  - If task function is `os.exec*()`, the pool process becomes the system command process, so the behavior is defined by the signal handler (if any) installed by the system command. Assuming the default signal handler (i.e., immediate termination upon receiving a `SIGTERM` signal), the system command process exits.
- Shell process: not affected
- System command process: not affected, unless the task function is `os.exec*()` (see above)

# Summary

In many cases, the standard
```
pool = mp.Pool()
pool.apply_async(<target_function>)
```
usage is sufficient. However, the optimal use of `multiprocessing` pools depends on the target functions one wants to run in the pool. Target functions can be split into two types:

1. Python functions that do not call the `fork()` or `exec()` family of system calls. To reduce memory utilization at the expense of speed of launching processes, you can set the process start method to `'spawn'` using `mp.set_start_method('spawn')` at the start of your main Python program.
2. System commands. If you want to be able to terminate the system commands with `pool.terminate()`, you have to use `os.exec*()` to launch them. However, this precludes getting any return value (not even an exit code) from the pool processes.

To run system commands with the ability to terminate them, check on their status, and retrieve their returned exit codes, you will have to create your own thread/process pool class, either by extending the existing `multiprocessing.pool.Pool` class, or [rolling your own altogether](https://github.com/bentyeh/scripts/blob/5a74eb0b22688d4226b87cf4c196024ef7a1be68/Python/utils.py#L257).

# Appendix

## `multiprocessing.Pool` versus `multiprocessing.pool.Pool`

`multiprocessing.Pool` is an alias of `multiprocessing.pool.Pool`. Upon [importing](https://chrisyeh96.github.io/2017/08/08/definitive-guide-python-imports.html) the multiprocessing package (running [`__init__.py`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/__init__.py)), the [global namespace of the package is updated](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/__init__.py#L22) with names in [`_default_context`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/context.py#L313), which is an isntance of the [DefaultContext](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/context.py#L225) class that inherits the [BaseContext](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/context.py#L30) class, which defines [`Pool`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/context.py#L114) as a wrapper around the [`multiprocessing.pool.Pool`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L146) constructor.

## Life of a pool process

Below is the hierarchy of some of the key functions and methods involved in creating a pool process. Each function / method is linked to the source code of the method being called, not where it was called in the parent stack frame.

[`pool = mp.Pool()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L155)
- [`pool._repopulate_pool()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L227)
  - [`w = pool.Process()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L152)
    - [`pool._ctx.Process()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L72) - on Unix-based systems, the BaseProcess subclass that is instantiated is [`mp.context.ForkProcess`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/context.py#L272)
  - [`w.start()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L101)
    - [`_popen = self._Popen(self)`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/popen_fork.py#L16)
      - [`_popen._launch()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/popen_fork.py#L67)
        - [`os.fork()`](https://docs.python.org/3/library/os.html#os.fork)
          - If in the child process:
            - [`w._bootstrap()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L276)
              - [`w.run()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L94)
                - `w._target()` - where `_target` is [`mp.pool.worker()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L93)
                  - `inqueue.get()` - blocks until a task is received
                  - `func(*args, *kwargs)` - `func` is the function one desires to run in parallel, as passed to `pool.apply_async(func)`
            - [`os._exit()`](https://docs.python.org/3/library/os.html#os._exit)

## Obtaining process (group) IDs and sending signals

Python
- Current process ID: `os.getpid()`
- Current process group ID: `os.getpgrp()`, or `os.getpgid(0)`
- Current process's parent's process ID: `os.getppid()`
- Process group ID of a process with process ID `<PID>`: `os.getpgid(<PID>)`
- Sending a signal to a process: `os.kill(<PID>, <SIGNAL>)`
- Sending a signal to a process group: `os.killpg(<PGID>, <SIGNAL>)`

Shell
- Current process ID: `echo $$`
- Process group ID, parent process ID, process ID: `ps -o pgid,ppid,pid,cmd <PID>`
- Sending a signal to a process: `kill -s <SIGNAL> <PID>`
- Sending a signal to a process group: `kill -s <SIGNAL> -- -<PGID>`

## `multiprocessing.dummy` module

The [`multiprocessing.dummy`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing.dummy) module "replicates the API of `multiprocessing` but is no more than a wrapper around the [`threading`](https://docs.python.org/3/library/threading.html) module." [[`multiprocessing` documentation](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing.dummy)] Specifically, ["dummy processes"](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/dummy/__init__.py#L34) inherit the `threading.Thread` class. Even though a thread-based pool system (such as that implemented by `multiprocessing.dummy`) is limited to a single processor by the Global Interpreter Lock, it can still be useful in the following contexts:
- Running multiple I/O-bound tasks simultaneously
- Lanching subprocesses (e.g., system commands)

The `multiprocessing.dummy` module has some notable limitations, however.
- Starting dummy processes with a target of `os.exec*()` will fail with an `OSError` exception: `[Errno 12] Cannot allocate memory`.
- `multiprocessing.pool.ThreadPool.terminate()` does not terminate workers.
  - Pools created using `multiprocessing.dummy.Pool()` are instances of the `multiprocessing.pool.ThreadPool` class, which inherits the `mulitprocessing.pool.Pool` class and shares the same `terminate()` method, which calls `_terminate()`, which calls `_terminate_pool()`, which includes the [lines](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/pool.py#L597)
    ```
    if pool and hasattr(pool[0], 'terminate'):
      util.debug('terminating workers')
      for p in pool:
          if p.exitcode is None:
              p.terminate()
    ```
    where `pool` is a list of process objects. [Real processes](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L63) have a [`terminate()`](https://github.com/python/cpython/blob/v3.7.3/Lib/multiprocessing/process.py#L119) method, so `hasattr(pool[0], 'terminate')` evaluates to `True`, and the `terminate()` method is called on each process object. However, threads and dummy processes do not have a `terminate()` method or attribute, so they are not actually stopped.

## Topics not discussed (may be added later!)

- Using `os.spawn()` as a means of launching system commands.
- Exactly how virtual and physical memory are managed across `fork()` system calls in Python.

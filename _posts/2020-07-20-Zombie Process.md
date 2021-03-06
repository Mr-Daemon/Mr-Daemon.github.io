---
title: "Zombie Process"
---

###  Zombie Process

This article we will talk about the zombie process: defination of it and how to avoid it.

#### Def

What is zombie process? When you use `fork()` , a child process was created. As parent process, if the child process terminated, you may need to know its terminated status — if there are some errors occurred. C library provide functions such as `waitpid()`, `wait()` to get the child processes’ terminated status. But if parent process don’t wait for its child, the terminated child process will become zombie process or defunct process, which can’t be killed by `SIGKILL` and will still have an entry in the process table. That may cause serious resource leak.  

And noticed that if a parent process terminated and some of its’ child processes are still running, the active processes will be adopt by the ==init process(pid 1)== , and will be reaped automatically after terminating.

#### Handle it !

To avoid our children become zombie, we can use `singal()` to handle our terminated children and don’t need wait them terminated.

Below I give a simple example to handle the `SIGCHLD` signal. It is worth noting that , we must use atomic function in the signal-handle function. Because we may be interrupted by other singal in this signal -handle fun. In below code block, the `s_print_s` and `s_print_i` I use the `write()` syscall to print info to `stdout`.

```c
void signal_handler(int singal)
{
    s_print_s("caught singal: ");
    s_print_i(singal);
    s_print_s("\n");
    int count = 0;
    int stat_loc;
    int pid;
    while ((pid = waitpid(-1, &stat_loc, WNOHANG | WUNTRACED)) > 0)
    {
        if (!stat_loc)
        { 
            s_print_s("child process ");
            s_print_i(pid);
            s_print_s(" is killed\n");
        }
    }
}
```


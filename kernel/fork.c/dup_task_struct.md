dup_task_struct
========================================

dup_task_struct来建立父进程task_struct的副本。用于子进程的新的task_struct实例可以在任何空闲的
内核内存位置分配.父子进程的task_struct实例只有一个成员不同：新进程分配了一个新的核心态栈，
即task_struct->stack。通常栈和thread_info一同保存在一个联合中，thread_info保存了线程所需的
所有特定于处理器的底层信息。

Arguments
----------------------------------------

path: kernel/fork.c
```
static struct task_struct *dup_task_struct(struct task_struct *orig)
{
```

tsk_fork_get_node
----------------------------------------

```
    struct task_struct *tsk;
    struct thread_info *ti;
    int node = tsk_fork_get_node(orig);
    int err;
```

https://github.com/novelinux/linux-4.x.y/tree/master/kernel/kthread.c/tsk_fork_get_node.md

alloc_task_struct_node
----------------------------------------

```
    tsk = alloc_task_struct_node(node);
    if (!tsk)
        return NULL;
```

https://github.com/novelinux/linux-4.x.y/tree/master/kernel/fork.c/alloc_task_struct_node.md

alloc_thread_info_node
----------------------------------------

```
    ti = alloc_thread_info_node(tsk, node);
    if (!ti)
        goto free_tsk;
```

https://github.com/novelinux/linux-4.x.y/tree/master/kernel/fork.c/alloc_thread_info_node.md

arch_dup_task_struct
----------------------------------------

```
    err = arch_dup_task_struct(tsk, orig);
    if (err)
        goto free_ti;
```

https://github.com/novelinux/linux-4.x.y/tree/master/kernel/fork.c/arch_dup_task_struct.md

setup_thread_stack
----------------------------------------

```
    tsk->stack = ti;
#ifdef CONFIG_SECCOMP
    /*
     * We must handle setting up seccomp filters once we're under
     * the sighand lock in case orig has changed between now and
     * then. Until then, filter must be NULL to avoid messing up
     * the usage counts on the error path calling free_task.
     */
    tsk->seccomp.filter = NULL;
#endif

    setup_thread_stack(tsk, orig);
```

set_task_stack_end_magic
----------------------------------------

设置栈结束标志防止栈溢出.

```
    clear_user_return_notifier(tsk);
    clear_tsk_need_resched(tsk);
    set_task_stack_end_magic(tsk);

#ifdef CONFIG_CC_STACKPROTECTOR
    tsk->stack_canary = get_random_int();
#endif

    /*
     * One for us, one for whoever does the "release_task()" (usually
     * parent)
     */
    atomic_set(&tsk->usage, 2);
#ifdef CONFIG_BLK_DEV_IO_TRACE
    tsk->btrace_seq = 0;
#endif
    tsk->splice_pipe = NULL;
    tsk->task_frag.page = NULL;
    tsk->wake_q.next = NULL;

    account_kernel_stack(ti, 1);

    return tsk;

free_ti:
    free_thread_info(ti);
free_tsk:
    free_task_struct(tsk);
    return NULL;
}
```

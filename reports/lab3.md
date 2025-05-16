= Chapter 3

== 简单总结

1. 对于`_trace_request`为0，将`_id`转换为`*const u8`的指针，然后直接获取改地址的值，并转换为`isize`返回。

2. 对于`_trace_request`为1，将`_id`转换为`*mut u8`指针`addr`，并且将`_data`转换为`u8`赋值给`addr`指针，然后返回`0`。

3. 对于`_trace_request`为2，在`task.rs`中为`TaskControlBlock`添加一个`task_syscall_record`数组，用于维护`task`的系统调用记录。并且为`task`实现`record_syscall_times`和`get_syscall_times`。前者用于调用时让`task_syscall_record[syscall_id] += 1`，并且在每次`syscall`前调用；后者用于获取`task_syscall_record[syscall_id]`的值，并且在`_trace_request`等于2时调用。

== 浅浅记录一下坑:

对于`task/task.rs`中, `TaskControlBlock.syscall_times`需要是`[u32; MAX_SYSCALL_ID]`, 不能是`[usize; MAX_SYSCALL_ID]`

== 简答作业

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容（运行 三个 bad 测例 (ch2b_bad_*.rs) ）， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

> [kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003a4, kernel killed it.
> [kernel] IllegalInstruction in application, kernel killed it.
> [kernel] IllegalInstruction in application, kernel killed it.
> os报错如上，机器拒绝执行指令。



2. 深入理解 `trap.S` 中两个函数 `__alltraps` 和 `__restore` 的作用，并回答如下问题:
  1. L40：刚进入 `__restore` 时，`sp` 代表了什么值。请指出 `__restore` 的两种使用情景。
  > 刚进入`__restore`时, `sp` 代表了当前进程的内核栈，`__restore` 用于：1. 从中断/异常返回时，将栈上保存的寄存器信息恢复到cpu上。2. 设置一些初试的值，用于直接从内核启动一个进程。

  2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

  ```asm
  ld t0, 32*8(sp)
  ld t1, 33*8(sp)
  ld t2, 2*8(sp)
  csrw sstatus, t0
  csrw sepc, t1
  csrw sscratch, t2
  ```

  > 处理了 `sstatus、sepc、sscratch` 这几个寄存器，分别保存了进入内核态前的当前状态、pc指向的地址、用户态栈起始地址。

3. L50-L56：为何跳过了 `x2` 和 `x4`？

```asm
ld x1, 1*8(sp)
ld x3, 3*8(sp)
.set n, 5
.rept 27
   LOAD_GP %n
   .set n, n+1
.endr
```

> `x2`寄存器是`sp`寄存器，指向了当前内核栈，如果立即执行，则会切换到另一个栈，所以不能立即执行，需要等恢复操作结束后，再切换栈。`x4`则是对进程无用，无须恢复或者保存。

4. L60：该指令之后，`sp` 和 `sscratch` 中的值分别有什么意义？
```asm
csrrw sp, sscratch, sp
```

> 在改指令结束之后，`sp` 和 `sscratch` 的值交换，指令之前 `sp -> kerne stack` 、`sscratch -> user stack`，指令之后 `sp -> user stack`、`sscratch -> kerne stack`。

5. `__restore`：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

> 在 `sret` 指令之后，`ret` 用于函数调用后返回，`sret` 用于从 S 态函数返回 U 态，指令会将 `sepc` 寄存器的值恢复到 `pc` 寄存器，会根据 `sstatus` 恢复状态到用户态。

6. L13：该指令之后，`sp` 和 `sscratch` 中的值分别有什么意义？
```asm
csrrw sp, sscratch, sp
```

> 在这条指令后，`sp` 和 `sscratch` 的值交换，指令之前 `sp -> user stack`、`sscratch -> kerne stack`，指令之后 `sp -> kerne stack` 、`sscratch -> user stack`。

7. 从 U 态进入 S 态是哪一条指令发生的？

> 通过系统调用 `syscall` 触发，系统调用是由 `ecall` 指令完成。

== 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与以下各位就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：
> 无

2. 此外，我也参考了以下资料，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：
> Qwen3、Deepseek、Gpt、Gemini

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。


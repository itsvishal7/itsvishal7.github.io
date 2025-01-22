### 💡 Linux kernel patch ideas
---

<details><summary> 🖱️ Global cpumasks tracking online, active, present and possible CPUs are not needed in CONFIG_SMP=N systems </summary>

\
Move the global CPU mask variables i.e. `__cpu_online_mask`, `__cpu_active_mask`,
`__cpu_present_mask`, and `__cpu_possible_mask` inside the CONFIG_SMP macro.
This change is important because when CONFIG_SMP is set to N (meaning SMP
is disabled), these CPU masks aren’t needed. By doing this, we can remove
or simplify the #ifdef conditional blocks in the code. The main benefit is
that it saves memory by not allocating these CPU masks on systems that
don’t use SMP.
        

```c
/* kernel/cpu.c
 *
 * Activate the first processor.
 */
void __init boot_cpu_init(void)
{
        int cpu = smp_processor_id();

        /* Mark the boot cpu "present", "online" etc for SMP and UP case */
        set_cpu_online(cpu, true);
        set_cpu_active(cpu, true);
        set_cpu_present(cpu, true);
        set_cpu_possible(cpu, true);

#ifdef CONFIG_SMP
        __boot_cpu_id = cpu;
#endif
}
```
---

</details>





### 🐟 Interesting concepts
---
#### jump label [kernel documentation](https://docs.kernel.org/staging/static-keys.html)
  
Using the ‘asm goto’, we can create branches that are either taken or not taken by default, without the need to check memory. Then, at run-time, we can patch the branch site to change the branch direction.

For example, if we have a simple branch that is disabled by default:

```
if (static_branch_unlikely(&key))
        printk("I am the true branch\n");
```

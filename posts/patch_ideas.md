### Linux kernel patch ideas


Move globally declared cpumasks variables i.e. __cpu_online_mask,
__cpu_active_mask, __cpu_present_mask and __cpu_possible_mask under
CONFIG_SMP macro, reason being when CONFIG_SMP=N then these cpumasks should
not be needed at all anywhere and as a side effect, it will help us remove
#ifdef block (see below) altogether or move entire thing inside the #ifdef
block

Real saving are coming from memory that we save by not allocating masks in
non SMP systems.

```
/*
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

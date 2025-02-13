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

<details><summary> 🖱️ convert to static key </summary>

\
Can `cpu_hotplug_offline_disabled` and `cpu_hotplug_disabled` be converted to static keys?
- since hotplug is a rare event, this code path isn't exercised much, hence does it make sense to use static key here

```c
/* kernel/cpu.c */
 static int cpu_down_maps_locked(unsigned int cpu, enum cpuhp_state target)
  {
          struct cpu_down_work work = { .cpu = cpu, .target = target, };

          /*
           * If the platform does not support hotplug, report it explicitly to
           * differentiate it from a transient offlining failure.
           */
          if (cpu_hotplug_offline_disabled)
                  return -EOPNOTSUPP;
          if (cpu_hotplug_disabled)
                  return -EBUSY;

          /*
           * Ensure that the control task does not run on the to be offlined
           * CPU to prevent a deadlock against cfs_b->period_timer.
           * Also keep at least one housekeeping cpu onlined to avoid generating
           * an empty sched_domain span.
           */
          for_each_cpu_and(cpu, cpu_online_mask, housekeeping_cpumask(HK_TYPE_DOMAIN)) {
                  if (cpu != work.cpu)
                          return work_on_cpu(cpu, __cpu_down_maps_locked, &work);
          }
          return -EBUSY;
  }
```
---

</details>

<details><summary> 🖱️ nvim lua script to fold ftrace function_graph </summary>

[something like this](https://github.com/torvalds/linux/blob/09fbf3d502050282bf47ab3babe1d4ed54dd1fd8/Documentation/trace/function-graph-fold.vim)  but in lua
Will need to understand: 
- vim folds and how do they work
- ftrace:
  - collect a trace, matching parenthesis part needs to be verified
  - just enable ftrace function_graph globally, because ftrace will start tracing in the middle, proper folds will not be possible
  - or, maybe just trace a given function, here, ftrace will start tracing only when a function is hit, stops tracing the it exits, this would work, i suppose

---

</details>


<details><summary> 🔎 ptrace selftests failing on powerpc </summary>

\
Build and run `ptrace` selftests in linux kernel
```bash
make O=/tmp/ptrace headers
make -C tools/testing/selftests/ O=/tmp/ptrace TARGETS=ptrace run_tests
```

```
make: Entering directory '/root//linux/tools/testing/selftests'
make[1]: Entering directory '/root//linux/tools/testing/selftests/ptrace'
CFLAGS = -std=c99 -pthread -Wall -isystem /tmp/ptrace/usr/include
KHDR_INCLUDES = -isystem /tmp/ptrace/usr/include
  CC       set_syscall_info
  CC       peeksiginfo
  CC       vmaccess
  CC       get_set_sud
make[1]: Leaving directory '/root//linux/tools/testing/selftests/ptrace'
make[1]: Entering directory '/root//linux/tools/testing/selftests/ptrace'
CFLAGS = -std=c99 -pthread -Wall -isystem /tmp/ptrace/usr/include
KHDR_INCLUDES = -isystem /tmp/ptrace/usr/include
TAP version 13
1..5
# timeout set to 45
# selftests: ptrace: get_syscall_info
# TAP version 13
# 1..1
# # Starting 1 tests from 1 test cases.
# #  RUN           global.get_syscall_info ...
# #            OK  global.get_syscall_info
# ok 1 global.get_syscall_info
# # PASSED: 1 / 1 tests passed.
# # Totals: pass:1 fail:0 xfail:0 xpass:0 skip:0 error:0
ok 1 selftests: ptrace: get_syscall_info

# timeout set to 45
# selftests: ptrace: set_syscall_info
# TAP version 13
# 1..1
# # Starting 1 tests from 1 test cases.
# #  RUN           global.set_syscall_info ...
# #            OK  global.set_syscall_info
# ok 1 global.set_syscall_info
# # PASSED: 1 / 1 tests passed.
# # Totals: pass:1 fail:0 xfail:0 xpass:0 skip:0 error:0
ok 2 selftests: ptrace: set_syscall_info

# timeout set to 45
# selftests: ptrace: peeksiginfo
# PASS
ok 3 selftests: ptrace: peeksiginfo

# timeout set to 45
# selftests: ptrace: vmaccess
# TAP version 13
# 1..2
# # Starting 2 tests from 1 test cases.
# #  RUN           global.vmaccess ...
# #            OK  global.vmaccess
# ok 1 global.vmaccess
# #  RUN           global.attach ...
# # attach: Test terminated by timeout
# #          FAIL  global.attach
# not ok 2 global.attach
# # FAILED: 1 / 2 tests passed.
# # Totals: pass:1 fail:1 xfail:0 xpass:0 skip:0 error:0
not ok 4 selftests: ptrace: vmaccess # exit=1

# timeout set to 45
# selftests: ptrace: get_set_sud
# TAP version 13
# 1..1
# # Starting 1 tests from 1 test cases.
# #  RUN           global.get_set_sud ...
# # get_set_sud.c:43:get_set_sud:Expected ret (-1) == 0 (0)
# # get_set_sud: Test terminated by assertion
# #          FAIL  global.get_set_sud
# not ok 1 global.get_set_sud
# # FAILED: 0 / 1 tests passed.
# # Totals: pass:0 fail:1 xfail:0 xpass:0 skip:0 error:0
not ok 5 selftests: ptrace: get_set_sud # exit=1
make[1]: Leaving directory '/root//linux/tools/testing/selftests/ptrace'
make: Leaving directory '/root//linux/tools/testing/selftests'
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

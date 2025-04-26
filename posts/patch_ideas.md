### 💡 Linux kernel patch ideas

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

<details><summary>🕳️ Adding SCHED_WARN_ON in pick_eevdf</summary>

There is a possibility that `pick_eevdf()` can still return `NULL` causing kernel panic in `pick_next_entity()`
```diff
diff --git a/kernel/sched/fair.c b/kernel/sched/fair.c
index 1c0ef435a7aa..be8e3ba38d63 100644
--- a/kernel/sched/fair.c
+++ b/kernel/sched/fair.c
@@ -964,6 +964,12 @@ static struct sched_entity *pick_eevdf(struct cfs_rq *cfs_rq)
        if (!best || (curr && entity_before(curr, best)))
                best = curr;

+       /*
+        * The return value of pick_eevdf gets dereferenced
+        * without a NULL check in pick_next_entity.
+        * Add SCHED_WARN_ON to catch it.
+        */
+       SCHED_WARN_ON(!best);
        return best;
 }
```
</details>

<details><summary> 🤔 why so many execve?</summary>
        
if a process/program executes more than one `execve`, because the binary was not present. perhaps it's a good idea to use file system api to figure out where a binary exists.
```
18:35:50.335560 2583    2582    /usr/libexec/gcc/x86_64-redhat-linux/14/cc1 -quiet setuid.c -quiet -dumpdir a- -dumpbase setuid.c -dumpbase-ext .c -mtune=generic -march=x86-64 -o /tmp/ccAr4gUY.s
18:35:50.455417 2584    2582    as --64 -o /tmp/ccaUs46A.o /tmp/ccAr4gUY.s
18:35:50.455674 2584    2582    as --64 -o /tmp/ccaUs46A.o /tmp/ccAr4gUY.s
18:35:50.455989 2584    2582    as --64 -o /tmp/ccaUs46A.o /tmp/ccAr4gUY.s
18:35:50.456219 2584    2582    as --64 -o /tmp/ccaUs46A.o /tmp/ccAr4gUY.s
18:35:50.456415 2584    2582    as --64 -o /tmp/ccaUs46A.o /tmp/ccAr4gUY.s
18:35:50.456563 2584    2582    as --64 -o /tmp/ccaUs46A.o /tmp/ccAr4gUY.s
18:35:50.475399 2585    2582    /usr/libexec/gcc/x86_64-redhat-linux/14/collect2 -plugin /usr/libexec/gcc/x86_64-redhat-linux/14/liblto_plugin.so -plugin-opt=/usr/libexec/gcc/x86_64-redhat-linux/14/lto-wrapper -plugin-opt=-fresolution=/tmp/ccc7g5Gk.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --build-id --no-add-needed --eh-frame-hdr --hash-style=gnu -m elf_x86_64
18:35:50.491106 2586    2585    /usr/bin/ld -plugin /usr/libexec/gcc/x86_64-redhat-linux/14/liblto_plugin.so -plugin-opt=/usr/libexec/gcc/x86_64-redhat-linux/14/lto-wrapper -plugin-opt=-fresolution=/tmp/ccc7g5Gk.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --build-id --no-add-needed --eh-frame-hdr --hash-style=gnu -m elf_x86_64
```
```
# cat pid.2584
execve("/root/.local/bin/as", ["as", "--64", "-o", "/tmp/ccaUs46A.o", "/tmp/ccAr4gUY.s"], 0x3ac35fc0 /* 38 vars */) = -1 ENOENT (No such file or directory)
execve("/root/bin/as", ["as", "--64", "-o", "/tmp/ccaUs46A.o", "/tmp/ccAr4gUY.s"], 0x3ac35fc0 /* 38 vars */) = -1 ENOENT (No such file or directory)
execve("/usr/local/sbin/as", ["as", "--64", "-o", "/tmp/ccaUs46A.o", "/tmp/ccAr4gUY.s"], 0x3ac35fc0 /* 38 vars */) = -1 ENOENT (No such file or directory)
execve("/usr/local/bin/as", ["as", "--64", "-o", "/tmp/ccaUs46A.o", "/tmp/ccAr4gUY.s"], 0x3ac35fc0 /* 38 vars */) = -1 ENOENT (No such file or directory)
execve("/usr/sbin/as", ["as", "--64", "-o", "/tmp/ccaUs46A.o", "/tmp/ccAr4gUY.s"], 0x3ac35fc0 /* 38 vars */) = -1 ENOENT (No such file or directory)
execve("/usr/bin/as", ["as", "--64", "-o", "/tmp/ccaUs46A.o", "/tmp/ccAr4gUY.s"], 0x3ac35fc0 /* 38 vars */) = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```
Note: next statement/instruction after a successful execve is not should never be reached.
</details>

---

### 🐟 Interesting concepts
#### jump label [kernel documentation](https://docs.kernel.org/staging/static-keys.html)
  
Using the ‘asm goto’, we can create branches that are either taken or not taken by default, without the need to check memory. Then, at run-time, we can patch the branch site to change the branch direction.

For example, if we have a simple branch that is disabled by default:

```
if (static_branch_unlikely(&key))
        printk("I am the true branch\n");
```

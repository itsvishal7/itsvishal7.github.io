#### stack trace

when hotplug is done from 

```
echo 0 > /sys/devices/system/cpu/cpu7/online
```

```bash
$ strace -e t=/open,/write chcpu -d 7
openat(AT_FDCWD, "/sys/devices/system/cpu", O_RDONLY|O_CLOEXEC) = 3
openat(3, "cpu7/online", O_WRONLY|O_CLOEXEC) = 4
write(4, "0", 1)                        = 1
write(1, "CPU 7 disabled\n", 15CPU 7 disabled)        = 15
+++ exited with 0 +++
```


```bash
# bpftrace -e 'kprobe:cpu_device_down { @[kstack()] = count(); } '
Attaching 1 probe...
^C

@[
    cpu_device_down+12
    cpu_subsys_offline+32
    device_offline+292
    online_store+140
    dev_attr_store+52
    sysfs_kf_write+100
    kernfs_fop_write_iter+436
    vfs_write+772
    ksys_write+144
    system_call_exception+320
    system_call_vectored_common+348
]: 1
```

#### hotplug static tracepoint

<details> <summary> perf record -e cpuhp:* -a -- chcpu -d 7 </summary>

```bash
# perf script
 kworker/0:1-mm_  493768 [000] 616455.544774:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 235 (cpuhp_kick_ap_work)
         cpuhp/7      51 [007] 616455.544802:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 234 (sched_cpu_deactivate)
         cpuhp/7      51 [007] 616455.604765:        cpuhp:cpuhp_exit:  cpu: 0007  state: 233 step: 234 ret: 0
         cpuhp/7      51 [007] 616455.604768:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 204 (free_slot_cache)
         cpuhp/7      51 [007] 616455.604776:        cpuhp:cpuhp_exit:  cpu: 0007  state: 203 step: 204 ret: 0
         cpuhp/7      51 [007] 616455.604779:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 203 (osnoise_cpu_die)
         cpuhp/7      51 [007] 616455.604783:        cpuhp:cpuhp_exit:  cpu: 0007  state: 202 step: 203 ret: 0
         cpuhp/7      51 [007] 616455.604786:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 202 (hwlat_cpu_die)
         cpuhp/7      51 [007] 616455.604789:        cpuhp:cpuhp_exit:  cpu: 0007  state: 201 step: 202 ret: 0
         cpuhp/7      51 [007] 616455.604792:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 196 (unregister_cpu_online)
         cpuhp/7      51 [007] 616455.604929:        cpuhp:cpuhp_exit:  cpu: 0007  state: 195 step: 196 ret: 0
         cpuhp/7      51 [007] 616455.604933:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 194 (stop_watchdog_on_cpu)
         cpuhp/7      51 [007] 616455.604937:        cpuhp:cpuhp_exit:  cpu: 0007  state: 193 step: 194 ret: 0
         cpuhp/7      51 [007] 616455.604941:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 193 (vmstat_cpu_down_prep)
         cpuhp/7      51 [007] 616455.604946:        cpuhp:cpuhp_exit:  cpu: 0007  state: 192 step: 193 ret: 0
         cpuhp/7      51 [007] 616455.604949:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 191 (text_area_cpu_down_mm)
         cpuhp/7      51 [007] 616455.604967:        cpuhp:cpuhp_exit:  cpu: 0007  state: 190 step: 191 ret: 0
         cpuhp/7      51 [007] 616455.604970:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 189 (rcutree_offline_cpu)
         cpuhp/7      51 [007] 616455.604976:        cpuhp:cpuhp_exit:  cpu: 0007  state: 188 step: 189 ret: 0
         cpuhp/7      51 [007] 616455.604980:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 187 (workqueue_offline_cpu)
         cpuhp/7      51 [007] 616455.605094:        cpuhp:cpuhp_exit:  cpu: 0007  state: 186 step: 187 ret: 0
         cpuhp/7      51 [007] 616455.605097:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 186 (lockup_detector_offline_cpu)
         cpuhp/7      51 [007] 616455.605102:        cpuhp:cpuhp_exit:  cpu: 0007  state: 185 step: 186 ret: 0
         cpuhp/7      51 [007] 616455.605105:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 185 (tmigr_cpu_offline)
         cpuhp/7      51 [007] 616455.605138:        cpuhp:cpuhp_exit:  cpu: 0007  state: 184 step: 185 ret: 0
         cpuhp/7      51 [007] 616455.605141:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 183 (ppc_hv_gpci_cpu_offline)
         cpuhp/7      51 [007] 616455.605144:        cpuhp:cpuhp_exit:  cpu: 0007  state: 182 step: 183 ret: 0
         cpuhp/7      51 [007] 616455.605148:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 182 (ppc_hv_24x7_cpu_offline)
         cpuhp/7      51 [007] 616455.605151:        cpuhp:cpuhp_exit:  cpu: 0007  state: 181 step: 182 ret: 0
         cpuhp/7      51 [007] 616455.605154:       cpuhp:cpuhp_enter: cpu: 0007 target: 143 step: 153 (perf_event_exit_cpu)
 kworker/0:1-mm_  493768 [000] 616455.605203:        cpuhp:cpuhp_exit:  cpu: 0007  state: 143 step: 235 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605207:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step: 143 (takedown_cpu)
 kworker/0:1-mm_  493768 [000] 616455.605609:        cpuhp:cpuhp_exit:  cpu: 0007  state:  88 step: 143 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605612:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  87 (finish_cpu)
 kworker/0:1-mm_  493768 [000] 616455.605615:        cpuhp:cpuhp_exit:  cpu: 0007  state:  86 step:  87 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605618:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  63 (timers_dead_cpu)
 kworker/0:1-mm_  493768 [000] 616455.605623:        cpuhp:cpuhp_exit:  cpu: 0007  state:  62 step:  63 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605626:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  59 (zs_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605630:        cpuhp:cpuhp_exit:  cpu: 0007  state:  58 step:  59 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605633:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  55 (topology_remove_dev)
 kworker/0:1-mm_  493768 [000] 616455.605669:        cpuhp:cpuhp_exit:  cpu: 0007  state:  54 step:  55 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605672:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  47 (rcutree_dead_cpu)
 kworker/0:1-mm_  493768 [000] 616455.605675:        cpuhp:cpuhp_exit:  cpu: 0007  state:  46 step:  47 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605678:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  44 (smpcfd_dead_cpu)
 kworker/0:1-mm_  493768 [000] 616455.605683:        cpuhp:cpuhp_exit:  cpu: 0007  state:  43 step:  44 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605686:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  40 (ppc_numa_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605689:        cpuhp:cpuhp_exit:  cpu: 0007  state:  39 step:  40 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605692:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  32 (dev_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605701:        cpuhp:cpuhp_exit:  cpu: 0007  state:  31 step:  32 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605703:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  31 (page_alloc_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605732:        cpuhp:cpuhp_exit:  cpu: 0007  state:  30 step:  31 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605735:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  30 (radix_tree_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605742:        cpuhp:cpuhp_exit:  cpu: 0007  state:  29 step:  30 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605744:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  29 (percpu_counter_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605890:        cpuhp:cpuhp_exit:  cpu: 0007  state:  28 step:  29 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605893:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  28 (memcg_hotplug_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605898:        cpuhp:cpuhp_exit:  cpu: 0007  state:  27 step:  28 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605901:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  27 (console_cpu_notify)
 kworker/0:1-mm_  493768 [000] 616455.605910:        cpuhp:cpuhp_exit:  cpu: 0007  state:  26 step:  27 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605912:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  26 (buffer_exit_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605916:        cpuhp:cpuhp_exit:  cpu: 0007  state:  25 step:  26 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605919: cpuhp:cpuhp_multi_enter: cpu: 0007 target:   0 step:  25 (blk_mq_hctx_notify_dead)
 kworker/0:1-mm_  493768 [000] 616455.605923:        cpuhp:cpuhp_exit:  cpu: 0007  state:  24 step:  25 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605925: cpuhp:cpuhp_multi_enter: cpu: 0007 target:   0 step:  22 (bio_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605929:        cpuhp:cpuhp_exit:  cpu: 0007  state:  21 step:  22 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605932: cpuhp:cpuhp_multi_enter: cpu: 0007 target:   0 step:  22 (bio_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605935:        cpuhp:cpuhp_exit:  cpu: 0007  state:  21 step:  22 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605938:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  21 (blk_softirq_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605941:        cpuhp:cpuhp_exit:  cpu: 0007  state:  20 step:  21 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605943:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  20 (irq_poll_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.605948:        cpuhp:cpuhp_exit:  cpu: 0007  state:  19 step:  20 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.605951:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  17 (pseries_cpuidle_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.644851:        cpuhp:cpuhp_exit:  cpu: 0007  state:  16 step:  17 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.644854:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  15 (takeover_tasklets)
 kworker/0:1-mm_  493768 [000] 616455.644862:        cpuhp:cpuhp_exit:  cpu: 0007  state:  14 step:  15 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.644864:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  14 (vmstat_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.644869:        cpuhp:cpuhp_exit:  cpu: 0007  state:  13 step:  14 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.644872:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  13 (page_writeback_cpu_online)
 kworker/0:1-mm_  493768 [000] 616455.644876:        cpuhp:cpuhp_exit:  cpu: 0007  state:  12 step:  13 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.644879:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:  11 (slub_cpu_dead)
 kworker/0:1-mm_  493768 [000] 616455.645522:        cpuhp:cpuhp_exit:  cpu: 0007  state:  10 step:  11 ret: 0
 kworker/0:1-mm_  493768 [000] 616455.645525:       cpuhp:cpuhp_enter: cpu: 0007 target:   0 step:   2 (perf_event_exit_cpu)
 kworker/0:1-mm_  493768 [000] 616455.645529:        cpuhp:cpuhp_exit:  cpu: 0007  state:   1 step:   2 ret: 0
```
</details>


<details><summary>states</summary>

```
# cat /sys/devices/system/cpu/hotplug/states
  0: offline
  1: threads:prepare
  2: perf:prepare
  5: perf/powerpc:prepare
 11: slub:dead
 13: mm/writeback:dead
 14: mm/vmstat:dead
 15: softirq:dead
 17: cpuidle/pseries:DEAD
 20: irq_poll:dead
 21: block/softirq:dead
 22: block/bio:dead
 25: block/mq:dead
 26: fs/buffer:dead
 27: printk:dead
 28: mm/memctrl:dead
 29: lib/percpu_cnt:dead
 30: lib/radix:dead
 31: mm/page_alloc:pcp
 32: net/dev:dead
 36: padata:dead
 38: random:prepare
 39: workqueue:prepare
 40: powerpc/numa:prepare
 41: hrtimers:prepare
 43: smpcfd:prepare
 44: relay:prepare
 46: RCU/tree:prepare
 54: base/topology:prepare
 57: trace/RB:prepare
 58: mm/zsmalloc:prepare
 62: timers:prepare
 63: tmigr:prepare
 65: crash/cpuhp
 87: cpu:bringup
 88: idle:dead
 89: ap:offline
 91: sched:starting
 92: RCU/tree:dying
134: clockevents/dummy_timer:starting
140: smpcfd:dying
141: hrtimers:dying
142: tick:dying
145: ap:online
146: cpu:teardown
150: sched:waitempty
151: smpboot/threads:online
152: irq/affinity:online
153: block/mq:online
156: perf:online
184: perf/powerpc/hv_24x7:online
185: perf/powerpc/hv_gcpi:online
187: tmigr:online
188: lockup_detector:online
189: workqueue:online
190: random:online
191: RCU/tree:online
193: powerpc/text_poke_mm:online
194: mm/writeback:online
195: mm/vmstat:online
196: powerpc/watchdog:online
197: padata:online
198: powerpc/topology:online
199: mm/compaction:online
200: io-wq/online
201: lib/percpu_cnt:online
202: cpuidle/pseries:online
203: printk:online
204: trace/hwlat:online
205: trace/osnoise:online
206: swap_slots_cache
236: sched:active
237: online
```
</details>


```
# bpftrace -e '
kprobe:cpu_down_maps_locked,
kprobe:__cpu_down_maps_locked
{
printf("%s %s %d\n", func, comm, cpu);
}'

kretprobe:cpu_down_maps_locked,
kretprobe:__cpu_down_maps_locked
{
printf("%s %s %d = %d\n", func, comm, cpu, retval);
}'
Attaching 4 probes...
cpu_down_maps_locked bash 74
__cpu_down_maps_locked kworker/0:1 0
work_for_cpu_fn kworker/0:1 0 = 0
target_store bash 74 = 0
```
Above, trace was collected by running `echo 207 > /sys/devices/system/cpu/cpu7/hotplug/target`

















































k

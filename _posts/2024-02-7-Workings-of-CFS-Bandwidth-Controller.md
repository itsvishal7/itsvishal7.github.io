# Creation of new thread group

Creating a new task group in the Linux kernel scheduler involves several steps, primarily focused on allocating and initializing the necessary data structures to support fair scheduling. Here's an overview of how a new task group is created, particularly focusing on the `alloc_fair_sched_group` and `sched_create_group` functions.

## Step 1: Allocation of Task Group Structure

The memory for the task group structure itself is allocated in the `sched_create_group` function, which takes a pointer to the parent task group as its argument. This step is crucial for setting up a new task group under a specific parent, establishing a hierarchical relationship between them.

```c
struct task_group *sched_create_group(struct task_group *parent);
```

## Step 2: Allocating Data Structures in `alloc_fair_sched_group`

The `alloc_fair_sched_group` function is responsible for allocating the necessary data structures needed for the task group `tg`. This function performs the following operations:

1. **Allocation of Arrays for CFS and SE:** It allocates two arrays of pointers for storing pointers to `cfs_rq` and `se` belonging to the task group `tg`.

```c
tg->cfs_rq = kcalloc(nr_cpu_ids, sizeof(cfs_rq), GFP_KERNEL);
if (!tg->cfs_rq)
    goto err;
tg->se = kcalloc(nr_cpu_ids, sizeof(se), GFP_KERNEL);
if (!tg->se)
    goto err;
```

2. **Setting Default Shares:** The function sets `tg->shares` with a default value, typically `NICE_0_LOAD`, which influences the scheduling fairness.

```c
tg->shares = NICE_0_LOAD;
```

## Step 3: [Initializing CFS Bandwidth Structure](#init_cfs_bandwidth-function)

It initializes the `cfs_bandwidth` structure related to this task group by making a call to `init_cfs_bandwidth(tg_cfs_bandwidth(tg), tg_cfs_bandwidth(parent))`. This step is essential for managing bandwidth allocation for the controlled fair scheduling (CFS) of the task group. \
[Notes on `init_cfs_bandwidth`](#init_cfs_bandwidth-function)

## Step 4: Allocating and Initializing Per-CPU Structures

For each possible CPU in the system, the task group setup process involves allocating memory for `cfs_rq` (Completely Fair Scheduler runqueue) and `se` (scheduling entity) structures, and initializing them accordingly. This is a critical step in ensuring that the scheduler has the necessary data structures to manage scheduling decisions for the task group on a per-CPU basis.

The operations performed for each CPU are as follows:

1. **Memory Allocation for `cfs_rq` and `se`:** For each possible CPU, memory is allocated for the `cfs_rq` and `se` structures. This allocation is done on the node corresponding to the CPU to optimize memory access patterns.

```c
for_each_possible_cpu(i) {
    cfs_rq = kzalloc_node(sizeof(struct cfs_rq), GFP_KERNEL, cpu_to_node(i));
    if (!cfs_rq)
        goto err;

    se = kzalloc_node(sizeof(struct sched_entity), GFP_KERNEL, cpu_to_node(i));
    if (!se)
        goto err_free_rq;
    ...
}
```

2. **Initialization of `cfs_rq`:** After successful allocation, the `cfs_rq` structure for that CPU is initialized. This step sets up the runqueue for managing tasks within the completely fair scheduler.

```c
init_cfs_rq(cfs_rq);
```

3. **Establishing Relationship Between `cfs_rq`, `se`, and `tg`:** The relationship between the created `cfs_rq` and `se` structures with the task group (`tg`) is established. This involves setting up pointers and references that link these structures together, ensuring that scheduling decisions can be made with full awareness of the task group's structure and hierarchy. \
[Notes on `init_tg_cfs_entry`](#init_tg_cfs_entry-function)

```c
init_tg_cfs_entry(tg, cfs_rq, se, i, parent->se[i]);
```

4. **Initializing the Scheduling Entity's Runnable Average:** Lastly, the runnable average for the scheduling entity (`se`) is initialized. This step is crucial for the scheduler's load balancing and task distribution mechanisms, as it provides a basis for understanding the task's historical CPU usage and predicting its future CPU needs.

```c
init_entity_runnable_average(se);
```

## Error Handling

During the allocation and initialization process, error handling is crucial to ensure that the system remains in a consistent state in case of memory allocation failures. The use of `goto` statements for error handling helps in cleanly rolling back partial changes if an error occurs during the setup of the per-CPU structures.


---

## `init_cfs_bandwidth` Function

The `init_cfs_bandwidth` function initializes `struct cfs_bandwidth` structure of a task group. Here's a breakdown of its operations:

<details>
<summary> code </summary>

```c
void init_cfs_bandwidth(struct cfs_bandwidth *cfs_b, struct cfs_bandwidth *parent)
{
    raw_spin_lock_init(&cfs_b->lock);
    cfs_b->runtime = 0;
    cfs_b->quota = RUNTIME_INF;
    cfs_b->period = ns_to_ktime(default_cfs_period());
    cfs_b->burst = 0;
    cfs_b->hierarchical_quota = parent ? parent->hierarchical_quota : RUNTIME_INF;

    INIT_LIST_HEAD(&cfs_b->throttled_cfs_rq);
    hrtimer_init(&cfs_b->period_timer, CLOCK_MONOTONIC, HRTIMER_MODE_ABS_PINNED);
    cfs_b->period_timer.function = sched_cfs_period_timer;

    /* Add a random offset so that timers interleave */
    hrtimer_set_expires(&cfs_b->period_timer,
                        get_random_u32_below(cfs_b->period));
    hrtimer_init(&cfs_b->slack_timer, CLOCK_MONOTONIC, HRTIMER_MODE_REL);
    cfs_b->slack_timer.function = sched_cfs_slack_timer;
    cfs_b->slack_started = false;
}
```
</details>

### Key Operations

- **Initialize Lock:** The function starts by initializing the spin lock `&cfs_b->lock` whose job is to ensure thread-safe access to the bandwidth control structure.

- **Set Runtime and Quota:** It sets the runtime to 0 and the quota to `RUNTIME_INF`, indicating no limit on the execution time until adjusted. This is the default state, ensuring the task group can execute without bandwidth restrictions initially.

- **Configure Period and Burst:** The period is set using `ns_to_ktime(default_cfs_period())`, converting the default period from nanoseconds to kernel time format. The burst capability, which could allow temporary exceedance of the quota, is initialized to 0.

- **Hierarchical Quota:** If a parent's `struct cfs_bandwidth`  exists, `cfs_b` inherits its hierarchical quota; otherwise, it's set to `RUNTIME_INF`, implying no hierarchical limits.

- **Throttled CFS Runqueue List:** Initializes a list head for `cfs_b->throttled_cfs_rq`, which will track CFS runqueues that have been throttled due to bandwidth limitations.

- **Period Timer Initialization:** A high-resolution timer (`period_timer`) is initialized to manage the periodic reset of runtime and quota. It uses a monotonic clock, pinned mode, and is set to trigger the `sched_cfs_period_timer` callback function. A random offset is added to the timer's expiration to interleave it with other timers, reducing simultaneous expirations.

- **Slack Timer Setup:** Another timer, `slack_timer`, is initialized for handling slack time, allowing for some flexibility in quota enforcement. It's set to trigger the `sched_cfs_slack_timer` callback and is initially inactive (`slack_started = false`).

---

## `init_tg_cfs_entry` Function

The `init_tg_cfs_entry` function is a crucial part of setting up task groups within the Completely Fair Scheduler (CFS) in the Linux kernel. It initializes the connection between a task group (`tg`), its scheduling entities (`se`), and the CFS runqueue (`cfs_rq`) for a specific CPU. Here's a detailed explanation of its operations:

<details>
<summary> code </summary>

```c
void init_tg_cfs_entry(struct task_group *tg, struct cfs_rq *cfs_rq,
                       struct sched_entity *se, int cpu,
                       struct sched_entity *parent)
{
    struct rq *rq = cpu_rq(cpu);

    cfs_rq->tg = tg;
    cfs_rq->rq = rq;
    init_cfs_rq_runtime(cfs_rq);

    tg->cfs_rq[cpu] = cfs_rq;
    tg->se[cpu] = se;

    /* se could be NULL for root_task_group */
    if (!se)
        return;

    if (!parent) {
        se->cfs_rq = &rq->cfs;
        se->depth = 0;
    } else {
        se->cfs_rq = parent->my_q;
        se->depth = parent->depth + 1;
    }

    se->my_q = cfs_rq;
    /* guarantee group entities always have weight */
    update_load_set(&se->load, NICE_0_LOAD);
    se->parent = parent;
}
```
</details>

### Key Operations

- **Retrieve CPU's Runqueue (`rq`):** Obtains the runqueue for the specified CPU to associate with the CFS runqueue.

- **Association with Task Group and Runqueue:** Sets `cfs_rq->tg` to the provided task group (`tg`) and associates the CFS runqueue (`cfs_rq`) with the CPU's runqueue (`rq`).

- **Initialize CFS Runqueue Runtime:** Calls `init_cfs_rq_runtime` to initialize runtime properties of `cfs_rq`, including setting `runtime_enabled` to 0 and initializing lists for throttled runqueues and control group scheduling domains.

- **Assign CFS Runqueue and Scheduling Entity to Task Group:** Associates the CFS runqueue and scheduling entity with the task group for the specific CPU.

- **Handle Root Task Group Case:** For the root task group, there may not be a scheduling entity (`se`), in which case the function returns early.

- **Set Scheduling Entity Properties:** Configures the scheduling entity's properties based on whether it has a parent:
  - If there is no parent, the scheduling entity is associated directly with the CPU's main CFS runqueue (`rq->cfs`) and its depth is set to 0.
  - If there is a parent, the scheduling entity is linked to the parent's queue (`parent->my_q`) and its depth is incremented by 1 from the parent's depth.

- **Update Scheduling Entity's Load:** Ensures that the scheduling entity has a default weight by setting its load to `NICE_0_LOAD`, indicating the standard weight for fair scheduling.

- **Parent Association:** If applicable, assigns the parent scheduling entity to `se->parent`.


---
---



# CFS Bandwidth Controller Unthrottling Logic

The unthrottling process in the CFS bandwidth controller follows a specific code flow to ensure that tasks are scheduled fairly according to their runtime needs.

## Code Flow

1. **SCHED_CFS_PERIOD_TIMER**:
    - This is likely the entry point in the timer interrupt that handles period timing for the CFS.
2. **DO_SCHED_CFS_PERIOD_TIMER**:
    - A function that is called by the period timer to handle tasks at the end of a period.
3. **DISTRIBUTE_CFS_RUNTIME**:
    - This function distributes run time to tasks. This could involve adjusting the runtime quotas for tasks based on their consumption and the period time.

4. The unthrottling logic branches based on the current CPU:
    - If `current_cpu == cfs_rq's cpu`:
       1. **UNTHROTTLE_CFS_RQ**:
           - This is the final unthrottling function that is called regardless of the CPU check. It is the common exit point for the unthrottling logic.

    - If `current_cpu != cfs_rq's cpu`:
        1. **UNTHROTTLE_CFS_RQ_ASYNC** executed by current CPU:
            - This function asynchronously unthrottles a run queue that is not on the current CPU.
        2. **_UNTHROTTLE_CFS_RQ_ASYNC** executed by current CPU:
            - This appears to be the internal function that actually performs the asynchronous unthrottling.
        3. **_CFSB_CSD_UNTHROTTLE** executed by cfs_rq's CPU:
            - This function initiates the unthrottling on the cfs_rq's CPU.
        4. **UNTHROTTLE_CFS_RQ** executed by cfs_rq's CPU
            - Unthrottles this cfs_rq for the given `tg`


## Note
- There is a note indicating that the function and subsequent calls will be made from the CPU of the `cfs_rq`. This implies that the unthrottling process respects the CPU affinity of the run queue it is managing.

---
---

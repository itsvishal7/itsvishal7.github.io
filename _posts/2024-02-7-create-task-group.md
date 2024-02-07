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

3. **Initializing CFS Bandwidth Structure:** It initializes the `cfs_bandwidth` structure related to this task group by making a call to `init_cfs_bandwidth(tg_cfs_bandwidth(tg), tg_cfs_bandwidth(parent))`. This step is essential for managing bandwidth allocation for the controlled fair scheduling (CFS) of the task group.

## Step 3: Allocating and Initializing Per-CPU Structures

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
}
```

2. **Initialization of `cfs_rq`:** After successful allocation, the `cfs_rq` structure for that CPU is initialized. This step sets up the runqueue for managing tasks within the completely fair scheduler.

```c
init_cfs_rq(cfs_rq);
```

3. **Establishing Relationship Between `cfs_rq`, `se`, and `tg`:** The relationship between the created `cfs_rq` and `se` structures with the task group (`tg`) is established. This involves setting up pointers and references that link these structures together, ensuring that scheduling decisions can be made with full awareness of the task group's structure and hierarchy.

```c
init_tg_cfs_entry(tg, cfs_rq, se, i, parent->se[i]);
```

4. **Initializing the Scheduling Entity's Runnable Average:** Lastly, the runnable average for the scheduling entity (`se`) is initialized. This step is crucial for the scheduler's load balancing and task distribution mechanisms, as it provides a basis for understanding the task's historical CPU usage and predicting its future CPU needs.

```c
init_entity_runnable_average(se);
```

## Error Handling

During the allocation and initialization process, error handling is crucial to ensure that the system remains in a consistent state in case of memory allocation failures. The use of `goto` statements for error handling helps in cleanly rolling back partial changes if an error occurs during the setup of the per-CPU structures.
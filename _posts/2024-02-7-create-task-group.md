# Creating a New Task Group

Creating a new task group in the Linux kernel scheduler involves several steps, primarily focused on allocating and initializing the necessary data structures to support fair scheduling. Here's an overview of how a new task group is created, particularly focusing on the `alloc_fair_sched_group` and `sched_create_group` functions.

### Step 1: Allocation of Task Group Structure

The memory for the task group structure itself is allocated in the `sched_create_group` function, which takes a pointer to the parent task group as its argument. This step is crucial for setting up a new task group under a specific parent, establishing a hierarchical relationship between them.

```c
struct task_group *sched_create_group(struct task_group *parent);
```

### Step 2: Allocating Data Structures in `alloc_fair_sched_group`

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


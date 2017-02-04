
### PARALLEL_DEGREE_POLICY

```
MANUAL
Disables automatic degree of parallelism, statement queuing, and in-memory parallel execution. This reverts the behavior of parallel execution to what it was prior to Oracle Database 11g Release 2 (11.2). This is the default.

LIMITED
Enables automatic degree of parallelism for some statements but statement queuing and in-memory Parallel Execution are disabled. Automatic degree of parallelism is only applied to those statements that access tables or indexes decorated explicitly with the PARALLEL clause. Tables and indexes that have a degree of parallelism specified will use that degree of parallelism.

AUTO
Enables automatic degree of parallelism, statement queuing, and in-memory parallel execution.
```

### PARALLEL_MAX_SERVERS

```
PARALLEL_MAX_SERVERS specifies the maximum number of parallel execution processes and parallel recovery processes for an instance. As demand increases, Oracle increases the number of processes from the number created at instance startup up to this value.

If you set this parameter too low, some queries may not have a parallel execution process available to them during query processing. If you set it too high, memory resource shortages may occur during peak periods, which can degrade performance.
```

### PARALLEL_MIN_PERCENT

```
PARALLEL_MIN_PERCENT lets you specify the minimum percentage of the requested number of parallel execution processes required for parallel execution. This parameter controls the behavior for parallel operations when parallel statement queuing is not enabled (when PARALLEL_DEGREE_POLICY is set to manual or limited). It ensures that an operation always gets a minimum percentage of parallel execution servers or errors out. Setting this parameter ensures that parallel operations will not execute unless adequate resources are available. The default value of 0 means that no minimum percentage of processes has been set.

这个参数指定并行执行时，申请并行服务进程的最小值，它是一个百分比，比如我们设定这个值为50. 当一个SQL需要申请20个并行进程时，如果当前并行服务进程不足，按照这个参数的要求，这个SQL比如申请到20*50%=10个并行服务进程，如果不能够申请到这个数量的并行服务，SQL 将报出一个ORA-12827的错误。
```

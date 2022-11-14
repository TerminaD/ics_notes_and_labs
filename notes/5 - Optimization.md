# Principals of Writing Efficient Programs
1. Select appropriate algorithms and data structures
2. Understanding the capabilities and limitations of compiler optimization
3. Explore parallelism


# Capabilities and Limitations of Compiler Optimization
## Memory Aliasing
![/img/截屏2022-10-27 下午2.18.46.png | 300]]

Though `add2` is more efficient, the compiler can't just use `add2` as an optimized version of `add1` in case that `xp == yp`: **Memory Aliasing**

## Function Calls
![/img/截屏2022-10-27 下午2.21.35.png | 300]]
![/img/截屏2022-10-27 下午2.22.08.png | 300]]

In this case, the compiler can't substitute `func1` for `func2` 

 
# Expressing Program Performance
e.g. Calculating the prefix sum of a vector
![/img/截屏2022-10-27 下午2.25.38.png | 500]]
Clock cycle to number of elements

- **Linear Factor**
- **CPE**: Clock cycles per element


# Eliminating Loop Inefficiencies
e.g. Traversing a vector
![/img/截屏2022-10-27 下午2.29.23.png | 500]]

## Code Motion
Removing `vec_length()` from the loop
![/img/截屏2022-10-27 下午2.34.59.png | 300]]

Performance:
![/img/截屏2022-10-27 下午2.36.31.png | 500]]

The compiler is unable to carry out this code motion optimization in fear of side effects

## Reducing Procedure Calls
Reducing calls to `get_vec_element()`
![/img/截屏2022-10-27 下午2.39.27.png | 300]]

Performance:
![/img/截屏2022-10-27 下午2.39.42.png | 500]]

- Performance improvements are negligible
- Other bottlenecks might exist in the loop

## Reducing Memory References
Asm Code:
![/img/截屏2022-10-27 下午2.41.31.png | 300]]

- Extensive memory read and write to change accumulated value
- Introduce local variable `acc`, which can be stored in a register
![/img/截屏2022-10-27 下午2.43.17.png | 300]]

Performance:
![/img/截屏2022-10-27 下午2.44.32.png | 500]]

- Note that `-O2` optimization flag yield nearly identical results


# Understanding Modern Processors
## Instruction Set Parallelism
1. `addq %rax, %rax`
	- one add instruction
2. `addq %rax, 8(%rdx)`
	1. Read: read data from memory to processor
	2. Add: add that data to data stored in a register
	3. Write: Write the result back to memory

![/img/截屏2022-10-27 下午3.02.52.png]]

With this decoding procedure, the workload can be spread across multiple dedicated hardware units:
- Load: handles read
- Store: handles write
This allows different parts of instructions to be worked on simultaneously

**Compare w/ pipelining:**
- Work on multiple operations in the same clock cycle
- Not in order

## Branch Prediction & Opportunistic Execution
![/img/截屏2022-10-27 下午3.08.17.png]]

- **Branch unit** in charge of verifying prediction
	- If prediction is wrong, all results are discarded and execution goes back to branching point
- **Retirement Unit** in charge of managing register files and instructions
	- All instructions are placed in a queue
		- When an instruction has finished execution and all branching points leading to it are confirmed to be correct, the instruction is popped from the queue, or **retired**. Registers are ok to be updated.
		- If one of the branching points are false, then the instruction is cleared and all results are discarded

## IRL Example: Intel Core i7 Haswell
7 functional units
![/img/截屏2022-10-27 下午3.09.06.png | 500]]

### Functional unit performance
![/img/截屏2022-10-27 下午3.15.02.png | 500]]

**Latency**: number of clock cycles from start to end
**Issue**: number of clock cycles between two ops
**Capacity**: number of functional units capable of this op

### Fundamental Bounds
Upper limit
![/img/截屏2022-10-27 下午3.20.24.png | 500]]

- Note that integer addition throughput bound is 0.5 due to memory functional unit bottlenecks

**Prefix sum array revisited:**
![/img/截屏2022-10-27 下午3.21.55.png]]


# Data-Flow Graphs
Show how data dependency dictates execution order and form critical performance paths

![/img/截屏2022-10-27 下午3.24.32.png | 500]]

## Categorizing Registers
1. Read-only
2. Write-only
3. Local: modified and used only within a loop, different iterations are irrelevant
	- e.g. Conditional code registers
4. Loop: both used as source and destination, different iterations are relevant
	- e.g. `%rdx`, `%xmm0`
	- **The key to performance**

## Simplifying Data-Flow Graphs
Retain only loop registers and key operations
![/img/截屏2022-10-27 下午3.28.38.png | 300]]

Loop it!
![/img/截屏2022-10-27 下午3.29.04.png | 300]]

- We can identify 2 data dependency chains
- Say that float multiplication latency is 5 clock cycles and int addition is 1. Then the left chain is the **critical path**, bottlenecking performance
- Only provide the lower limit for cycle counts 

## Other Restricting Factors
- The number of available functional units
- The number of data transfer between functional units


# Loop Unrolling
## The Why
1. Reduce loop operations
2. Reduce number of ops in the critical path

## e.g. 2\*1 Unrolling
![/img/截屏2022-10-27 下午3.34.51.png | 500]]

Performance:
![/img/截屏2022-10-27 下午3.37.14.png | 500]]
- Reduced overhead for int add ops
- Capped by latency bounds

### Why is that?
![/img/截屏2022-10-27 下午3.38.29.png | 300]]


![/img/截屏2022-10-27 下午3.39.43.png | 500]]
![/img/截屏2022-10-27 下午3.39.55.png | 300]]
![/img/截屏2022-10-27 下午3.40.16.png | 300]]

- Key path still has n multiplication ops

## Enhancing Parallelism
### 2\*2 Unrolling
![/img/截屏2022-10-27 下午3.41.28.png | 500]]

Performance:
![/img/截屏2022-10-27 下午3.42.24.png | 500]]
- No longer bound by latency bounds

**Deep dive:**
![/img/截屏2022-10-27 下午3.43.02.png | 500]]

![/img/截屏2022-10-27 下午3.43.23.png | 500]]
- Each critical path only contains n/2 ops

**The limits:**
![/img/截屏2022-10-27 下午3.44.47.png | 500]]
- Approach throughput bound

### Reallocation Transformation
![/img/截屏2022-10-27 下午3.46.25.png | 500]]

![/img/截屏2022-10-27 下午3.46.45.png | 500]]

![/img/截屏2022-10-27 下午3.48.28.png | 500]]
![/img/截屏2022-10-27 下午3.48.47.png | 500]]


# Understanding Memory Performance
Keep unrolling to 20\*20 hurts performance: why?

## Register Spilling
![/img/截屏2022-10-27 下午3.51.09.png | 300]]
- Too much registers are used, forcibly placing some data onto the runway stack
- Most of the time, throughput bounds are reached before register spilling occurs

## Load Performance
![/img/截屏2022-10-27 下午3.53.56.png | 500]]
Everything is held up by loading the contents of `%rdi`

## Store Performance
![/img/截屏2022-10-27 下午3.56.19.png | 500]]
In example B, the read operation is dependent on the previous store operation: **write/read dependency**

![/img/截屏2022-10-27 下午4.00.05.png | 500]]


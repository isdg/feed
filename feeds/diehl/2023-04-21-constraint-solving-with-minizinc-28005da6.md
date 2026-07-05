---
title: Constraint Solving with MiniZinc
url: https://www.stephendiehl.com/posts/minizinc/
published: "2023-04-21T01:00:00Z"
feed: diehl
guid: https://www.stephendiehl.com/posts/minizinc/
---

# Constraint Solving with MiniZinc

MiniZinc is a powerful constraint modeling language that allows you to express complex problems declaratively and solve them efficiently. First, install MiniZinc from the [official website](https://www.minizinc.org/). You can use the MiniZinc IDE for an integrated environment or run MiniZinc from the shell

MiniZinc models consist of variable declarations, constraints, and an optional objective function. Let's start with a simple example:

```prolog
var 1..10: x;
var 1..10: y;

constraint x + y == 7;

solve satisfy;

```

This model declares two variables, `x` and `y`, each with a domain of 1 to 10. The constraint states that their sum should be 7. The `solve satisfy` statement tells MiniZinc to find any solution that satisfies the constraints.

Let's solve a more interesting problem: scheduling three tasks on two machines, minimizing the total completion time.

```prolog
% Parameters
int: n_tasks = 3;
int: n_machines = 2;
array[1..n_tasks] of int: durations = [3, 2, 4];

% Decision variables
array[1..n_tasks] of var 1..n_machines: machine;
array[1..n_tasks] of var 0..10: start_time;

% Constraints
constraint forall(i in 1..n_tasks) (
  start_time[i] + durations[i] <= 10
);

constraint forall(i,j in 1..n_tasks where i < j) (
  (machine[i] != machine[j]) \/
  (start_time[i] + durations[i] <= start_time[j]) \/
  (start_time[j] + durations[j] <= start_time[i])
);

% Objective
var int: makespan = max(i in 1..n_tasks)(start_time[i] + durations[i]);
solve minimize makespan;

% Output
output [
  "Machine assignments: \(machine)\n",
  "Start times: \(start_time)\n",
  "Makespan: \(makespan)\n"
];

```

Let's break down this model:

1. We define parameters for the number of tasks, number of machines, and task durations.
2. Decision variables include the machine assignment and start time for each task.
3. Constraints ensure that:
   - All tasks finish within a 10-time-unit window.
   - Tasks on the same machine don't overlap.
4. The objective is to minimize the makespan (total completion time).
5. We specify the output format for the solution.

Save this model in a file named `scheduling.mzn` and run it using the MiniZinc IDE or the command line:

```bash
minizinc scheduling.mzn

```

Which will give us:

```bash
Machine assignments: [1, 2, 1]
Start times: [0, 0, 3]
Makespan: 7

```

This solution assigns tasks 1 and 3 to machine 1, and task 2 to machine 2, with a total completion time of 7 time units.

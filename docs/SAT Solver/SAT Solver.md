# SAT Solver Encoding

The `sat_solver.py` module contains the core logic for encoding the scheduling problem as a SAT instance and invoking the solver. It employs the [PySAT](https://pysathq.github.io/) library with the [Minisat solver](http://minisat.se/MiniSat.html). 

## Key Components

### `function(num_courses, num_slots, courses, clash, m, clash_slots)`

- **Inputs:**
  - `num_courses` (int): Number of courses.
  - `num_slots` (int): Number of *unique* slots.
  - `courses` (list of lists): `courses[i]` is a list of slot indices that course *i* can occupy.
  - `clash` (list of lists): `clash[i]` is a list of course indices that cannot share a slot with course *i*.
  - `m` (int): Slot capacity parameter (maximum number of courses allowed in a slot).
  - `clash_slots` (list of pairs): Each pair `(s, t)` indicates that courses in slot `s` and slot `t` have additional clash restrictions.

- **Operation:**
  1. Creates a fresh `Minisat22` solver instance.
  2. Constructs a 2D array `arr` of integer variables of size `[num_courses][num_slots]`.  
     Each `arr[i][j]` represents the proposition “course *i* is assigned to slot *j*.”
  3. **Course Assignment Constraints:**
     - Adds a clause ensuring at least one of `arr[i][j]` is true for `j` in `courses[i]` (so each course is assigned to one of its allowed slots).
     - Adds pairwise clauses forbidding two slots for the same course simultaneously (for all pairs of slots `j, k` in `courses[i]`, add `(-arr[i][j] OR -arr[i][k])`).
  4. **Course Clash Constraints:**
     - For each pair of courses `(i, k)` in `clash[i]`, adds clauses `(-arr[i][j] OR -arr[k][j])` for all slots `j`, preventing them from sharing the same slot.
  5. **Slot Capacity Constraints:**
     - Builds a list `slot[j]` of courses that could be in slot `j`.
     - For each slot `j`, if there are ≥ `m` courses, generates all combinations of those courses of size `m` and adds a clause forbidding all of them being together in slot `j` (i.e., at most `m-1` courses in slot `j`).
  6. **Additional Clash-Slot Constraints:**
     - For each pair `(s, t)` in `clash_slots`, iterates over courses `j` in slot `s` and `k` in slot `t`.  
       If course `j` conflicts with course `k`, adds `(-arr[j][s] OR -arr[k][t])` to enforce that conflict.
  7. Calls `g.solve()` to run the SAT solver.
     - If satisfiable, collects all positive assignments in the model and converts them to slot indices (`(var-1) % num_slots` for each positive `var`).
  8. Returns `(v, ans)` where:
     - `v` is a boolean indicating satisfiability.
     - `ans` is the list of assigned slot indices for each course.

### `solver(num_courses, num_slots, courses, clashes, clash_slots)`

Defined in `pages/Head.py`, this function wraps `function(...)` in a loop:

1. It increments the slot capacity parameter `m` starting from 0.
2. Calls `function(...)` with the current `m`.
3. Repeats until a satisfiable assignment (`v == True`) is found.
4. Returns the solution `ans` for the minimal `m`.

This strategy automatically determines the minimum slot capacity required for a valid schedule.

---

With this encoding, the scheduler ensures:

- **Every course** is assigned to exactly one of its preferred slots.  
- **No two conflicting courses** share the same time slot.  
- **Slot capacities** are respected, automatically increasing capacity until a solution is found.  


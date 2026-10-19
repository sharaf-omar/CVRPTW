## 2. Formal problem statement

> Problem version: Level A (Advanced), the Capacitated Vehicle Routing Problem with Time Windows (CVRPTW), with peak-hour traffic.
> Every route must respect vehicle capacity and every customer's time window. Late service is forbidden. Early arrival is allowed, and the vehicle waits. Travel times depend on the time of day: they are longer during the peak period.

### 2.1 Problem definition

The CVRPTW is a combinatorial optimization problem in which a fleet of identical vehicles delivers goods from one depot to a set of customers. Each customer has a known demand and a time window during which service must begin. Each vehicle has a limited capacity, and every customer must be served exactly once.

The goal is to find a set of routes of minimum total cost that respects vehicle capacity, the single visit to each customer, and the time windows. Every route starts and ends at the depot. A vehicle that arrives before a window opens waits until it opens. A vehicle may not start service after the window closes.

**Traffic.** Road conditions change during the day. We use the simplest model that captures this: the planning horizon is cut into time slots (off-peak, peak, off-peak), and every road segment has an expected speed in each slot. The time to drive from one customer to another therefore depends on the slot in which the vehicle leaves. A trip that takes 20 minutes off-peak may take 30 minutes at the peak. This changes which routes are feasible: a customer reachable before its window closes in free-flow traffic may not be reachable at the peak, and a vehicle may do better by waiting for the peak to end.

The problem is motivated by urban delivery logistics. In the context of the CesiCDP project, it can represent a delivery company serving customers spread over an urban road network. The solution must decide three things together: which customers each vehicle serves, in what order, and at what time each delivery starts.

**Scope and assumptions.**

* **(A1)** There is a single depot, and the vehicles are identical (same capacity $Q$).
* **(A2)** Demands, service durations, time windows and expected traffic are known and deterministic.
* **(A3)** Every customer has strictly positive demand, $1\le q_i\le Q$. A customer with $q_i>Q$ makes the instance infeasible.
* **(A4)** Every customer is served in a single visit by a single vehicle. Deliveries are not split.
* **(A5)** Time windows are hard. Waiting is allowed, before service and after service, and costs nothing.
* **(A6)** Vehicles leave the depot at time $0$, in the first slot.
* **(A7)** At most $m$ vehicles are available, and each performs at most one route during the horizon.
* **(A8)** The travel time of a trip is fixed by the slot in which the vehicle departs. Speed changes during a trip are ignored (a standard simplification, see 2.9).

### 2.2 Mathematical notation

The problem is defined on a directed graph $G=(V,A)$. The depot appears twice: node $0$ is where every route starts, and node $n+1$ is a copy of the depot where every route ends. This lets the depot take part in the time constraints like any other node.

$$N=\{1,\dots,n\},\qquad V=\{0\}\cup N\cup\{n+1\},$$

$$A=\{(0,j):j\in N\}\ \cup\ \{(i,j):i,j\in N,\ i\ne j\}\ \cup\ \{(i,n+1):i\in N\}.$$

There is no arc into $0$, none out of $n+1$, and no arc $(0,n+1)$. The horizon $[0,H]$ is cut into $P$ slots with boundaries $0=\theta_0<\theta_1<\dots<\theta_P=H$. Slot $p$ is the interval from $\theta_{p-1}$ to $\theta_p$, and $\mathcal{P}=\{1,\dots,P\}$.

| Parameter | Domain | Definition |
|---|---|---|
| $n$ | $\mathbb{N}$ | Number of customers |
| $m$ | $\mathbb{N}$, $m\ge1$ | Maximum number of vehicles |
| $Q$ | $\mathbb{N}$ | Capacity of each vehicle |
| $q_i$ | $1\le q_i\le Q$ for $i\in N$; $q_0=q_{n+1}=0$ | Delivery demand of customer $i$ |
| $a_i$ | $\mathbb{R}_{\ge0}$ | Earliest permitted service start at $i$ |
| $b_i$ | $\mathbb{R}_{\ge0}$, $a_i\le b_i$ | Latest permitted service start at $i$ |
| $d_i$ | $\mathbb{R}_{\ge0}$; $d_0=d_{n+1}=0$ | Service duration at $i$ |
| $H$ | $\mathbb{R}_{>0}$ | End of the planning horizon; the depot window is $[0,H]$ |
| $P$, $\theta_p$ | $\mathbb{N}$, $\theta_0=0<\dots<\theta_P=H$ | Number of time slots and their boundaries |
| $t_{ij}$ | $\mathbb{R}_{\ge0}$ | Free-flow (traffic-free) travel time from $i$ to $j$ |
| $\rho_p$ | $\mathbb{R}$, $\rho_p\ge1$ | Congestion factor of slot $p$ ($\rho_p=1$ means free flow) |
| $t_{ij}^p$ | $\mathbb{R}_{\ge0}$ | Travel time from $i$ to $j$ when leaving in slot $p$; in the peak model $t_{ij}^p=\rho_p\,t_{ij}$ |
| $c_{ij}$ | $\mathbb{R}_{\ge0}$ | Cost of arc $(i,j)$, equal to the free-flow time $t_{ij}$ |
| $M_{ij}^p$ | $\mathbb{R}_{\ge0}$ | Big-M constant, defined in 2.6 |

For the depot nodes, $[a_0,b_0]=[a_{n+1},b_{n+1}]=[0,H]$, and rows and columns $0$ and $n+1$ are identical. All time quantities use the same unit, for example minutes from the start of the horizon. For benchmark instances, $t_{ij}$ is the Euclidean distance between the coordinates of $i$ and $j$, rounded as the benchmark prescribes.

### 2.3 Traffic model (simple peak)

**From road segments to customer pairs.** Customers are not joined by single road segments, so traffic on each segment cannot enter the routing model directly. We use two layers.

1. *Road layer (preprocessing).* Each road segment $e$ has an expected speed $v_e^p$ in each slot $p$. For every pair of locations and every slot, a shortest path on the road network with the speeds of that slot gives the travel time $t_{ij}^p$.
2. *Routing layer (the ILP and the heuristic).* Only the values $t_{ij}^p$ are used. All road detail is absorbed into them.

**Simple peak profile.** With three slots, one morning off-peak period, one peak period and one evening off-peak period:

| Slot $p$ | Interval | Factor $\rho_p$ |
|---|---|---|
| 1 (off-peak) | $[0,\theta_1]$ | $1$ |
| 2 (peak) | $[\theta_1,\theta_2]$ | $\rho\ge1$ (for example $1.5$) |
| 3 (off-peak) | $[\theta_2,H]$ | $1$ |

Then $t_{ij}^p=\rho_p\,t_{ij}$. For example, with $t_{ij}=20$, $\theta_1=120$, $\theta_2=240$ and $\rho=1.5$: leaving at time $100$ takes $20$, leaving at time $130$ takes $30$, and leaving at time $240$ takes $20$ again. Setting $\rho=1$ or $P=1$ gives the ordinary static CVRPTW, which we use as a consistency test and for benchmark validation. If segment-level speeds are available, the same model holds with the exact values $t_{ij}^p$ from the road layer.

**Waiting.** The vehicle may idle after finishing service, for example to let the peak pass. This is the reason a vehicle can sometimes do better by leaving later, and it is what keeps the model consistent: leaving later never forces a later arrival than leaving at the same instant would give, because the vehicle could always have left at the same instant.

### 2.4 Decision variables

| Variable | Domain | Meaning |
|---|---|---|
| $x_{ij}$ | $\{0,1\}$, $(i,j)\in A$ | 1 if some vehicle drives directly from $i$ to $j$ |
| $z_{ij}^p$ | $\{0,1\}$, $(i,j)\in A$, $p\in\mathcal{P}$ | 1 if arc $(i,j)$ is used and the vehicle leaves $i$ in slot $p$ |
| $s_i$ | $[a_i,b_i]$, $i\in V$ | Time at which service starts at $i$ ($s_0$: departure from the depot, $s_{n+1}$: return time) |
| $\delta_i$ | $[0,H]$, $i\in\{0\}\cup N$ | Time at which the vehicle leaves $i$ |
| $u_i$ | $[q_i,Q]$, $i\in N$ | Cumulative demand delivered on the route up to and including $i$ |

No variable carries a vehicle index. Each customer is visited once, so one $s_i$, one $\delta_i$ and one $u_i$ per customer are enough. Routes are recovered from $x$ by starting at node $0$ and following selected arcs to node $n+1$. The variables $z$ say which traffic slot applies to each used arc. Since $x_{ij}=\sum_p z_{ij}^p$, the variables $x$ are redundant, and we keep them for readability.

### 2.5 Objective function

$$
\boxed{\ \min Z=\sum_{(i,j)\in A} c_{ij}\,x_{ij}\ }\qquad c_{ij}=t_{ij}
$$

The objective is the total free-flow travel time (equal to distance on the benchmarks). Waiting time is not counted. The number of vehicles is bounded by $m$ but not penalized.

Traffic acts through the time constraints. Peak congestion does not change the cost of an arc, but it can make a route infeasible or force a vehicle to wait or to take a longer sequence. This choice keeps the model comparable with the static benchmarks, and it keeps one useful property: since the cost does not depend on when vehicles leave, an earlier schedule is never worse (Proposition 1).

*Variant (experienced travel time).* To minimize the time actually spent driving, replace the objective by
$$\min\ \sum_{(i,j)\in A}\sum_{p\in\mathcal{P}} t_{ij}^p\,z_{ij}^p.$$
It is still linear. The price is that start and departure times become real decisions: waiting out the peak can reduce the cost, so the earliest schedule is no longer always optimal, and the heuristic has to choose departure slots as well. The team should decide which objective to use.

*Variant (fleet size).* To minimize vehicles first, add $f\sum_{j\in N}x_{0j}$ with $f$ larger than the sum of all $c_{ij}$.

### 2.6 Complete ILP

$$\min\ \sum_{(i,j)\in A} c_{ij}\,x_{ij} \tag{0}$$

subject to

$$\sum_{j:(i,j)\in A} x_{ij}=1 \qquad \forall\, i\in N \tag{1}$$

$$\sum_{i:(i,j)\in A} x_{ij}=1 \qquad \forall\, j\in N \tag{2}$$

$$\sum_{j\in N} x_{0j}\ \le\ m \tag{3}$$

$$\sum_{j\in N} x_{0j}\ =\ \sum_{i\in N} x_{i,n+1} \tag{4}$$

$$u_i-u_j+Q\,x_{ij}\ \le\ Q-q_j \qquad \forall\, i,j\in N,\ i\ne j \tag{5}$$

$$q_i\ \le\ u_i\ \le\ Q \qquad \forall\, i\in N \tag{6}$$

$$\sum_{p\in\mathcal{P}} z_{ij}^p = x_{ij} \qquad \forall\, (i,j)\in A \tag{7}$$

$$\delta_i\ \ge\ s_i+d_i\quad \forall\, i\in N,\qquad \delta_0=0 \tag{8}$$

$$\delta_i\ \ge\ \theta_{p-1}\,z_{ij}^p \qquad \forall\, (i,j)\in A,\ p\in\mathcal{P} \tag{9}$$

$$\delta_i\ \le\ \theta_p+(H-\theta_p)\,(1-z_{ij}^p) \qquad \forall\, (i,j)\in A,\ p\in\mathcal{P} \tag{10}$$

$$\delta_i+t_{ij}^p-s_j\ \le\ M_{ij}^p\,(1-z_{ij}^p) \qquad \forall\, (i,j)\in A,\ p\in\mathcal{P} \tag{11}$$

$$a_i\ \le\ s_i\ \le\ b_i \quad \forall\, i\in V,\qquad s_0=0 \tag{12}$$

$$x_{ij},\,z_{ij}^p\in\{0,1\},\qquad 0\le\delta_i\le H,\qquad u_i\in\mathbb{R},\ s_i\in\mathbb{R} \tag{13}$$

with

$$M_{ij}^p=\max\{\,0,\ \hat H_i+t_{ij}^p-a_j\,\},\qquad \hat H_0=0,\quad \hat H_i=H\ \ (i\in N). \tag{14}$$

The bound $s_{n+1}\le H$ in (12) forces every vehicle to be back before the horizon ends. With $P=1$ and $\theta_1=H$, constraints (7) to (11) reduce to the static condition $s_i+d_i+t_{ij}\le s_j$ when $x_{ij}=1$ (up to the idle-time variable $\delta_i$, which then has no effect).

**Name.** The variables $s_i$, $\delta_i$ and $u_i$ are continuous, so strictly this is a mixed-integer linear program (MILP). When all data are integers, an earliest schedule has integer times, and these variables can be declared integer without changing the optimum, which gives a pure ILP.

**Size.** $(n^2+n)(P+1)$ binary variables, $3n+2$ continuous variables, and $O(n^2P)$ constraints. None depends on $m$. With the three-slot peak model the number of binaries is about four times that of the static model.

**Slot boundaries.** The slots overlap at their boundaries, so a departure exactly at $\theta_p$ may be assigned to either neighbouring slot. Leaving an arbitrarily small instant earlier gives the same time as the cheaper slot, so the optimal value is unchanged. With integer time units, replace $\theta_p$ by $\theta_p-1$ in (10) for $p<P$ to make the slots disjoint.

### 2.7 Meaning of the constraints

| # | Constraint | Meaning |
|---|---|---|
| (0) | $\min\sum c_{ij}x_{ij}$ | Minimize the total free-flow travel time of all vehicles. |
| (1) | out-degree of each customer $=1$ | Every customer is left exactly once. |
| (2) | in-degree of each customer $=1$ | Every customer is entered exactly once. With (1), each customer is visited exactly once. |
| (3) | $\sum_j x_{0j}\le m$ | At most $m$ vehicles leave the depot. |
| (4) | departures $=$ arrivals | As many routes end as start. Implied by (1) and (2); kept to make the route structure explicit. |
| (5) | $u_i-u_j+Qx_{ij}\le Q-q_j$ | If $j$ directly follows $i$, then $u_j\ge u_i+q_j$: the load accumulates along the route. When $x_{ij}=0$ it always holds. It also forbids cycles that avoid the depot. |
| (6) | $q_i\le u_i\le Q$ | The load after serving $i$ is at least $i$'s own demand and never exceeds the capacity. |
| (7) | $\sum_p z_{ij}^p=x_{ij}$ | A used arc has exactly one departure slot, and an unused arc has none. |
| (8) | $\delta_i\ge s_i+d_i$, $\delta_0=0$ | A vehicle cannot leave a customer before service is finished. It may idle longer. Vehicles leave the depot at time $0$. |
| (9), (10) | $\theta_{p-1}\le\delta_i\le\theta_p$ if $z_{ij}^p=1$ | The departure time lies inside the slot chosen for the arc. When $z_{ij}^p=0$ both inequalities are always satisfied. |
| (11) | $s_j\ge\delta_i+t_{ij}^p$ if $z_{ij}^p=1$ | Service at $j$ cannot start before the vehicle has left $i$ and driven for the travel time of the chosen slot. Being an inequality, it allows waiting before service. |
| (12) | $a_i\le s_i\le b_i$, $s_0=0$ | Service starts inside the window. Late is forbidden. Early means waiting until $a_i$. Every vehicle is back by $H$. |
| (13) | domains | Arc and slot variables are binary. Load and time variables are continuous. |
| (14) | $M_{ij}^p$ | The smallest constant that keeps (11) valid when $z_{ij}^p=0$ (Proposition 3). |

### 2.8 Correctness and properties

**Evaluation rule (earliest schedule).** Let a route $0\to i_1\to\dots\to i_r\to n+1$ be given. Set $s_0=0$ and, for each arc $(i,j)$ of the route, let $r_i=s_i+d_i$ be the time the vehicle is ready to leave $i$. The earliest arrival at $j$ is

$$\text{arr}_{ij}(r_i)=\min_{p:\ \max(r_i,\theta_{p-1})\le\theta_p}\Big\{\max(r_i,\theta_{p-1})+t_{ij}^p\Big\},\qquad s_j=\max\{a_j,\ \text{arr}_{ij}(r_i)\}.$$

The route is feasible if all $s_j\le b_j$, $s_{n+1}\le H$, and the load is at most $Q$. The rule tries each slot, leaving as soon as possible within it, and keeps the earliest arrival. It takes $O(P)$ time per arc, and it is the route-evaluation function for the heuristic.

**Proposition 1 (equivalence and earliest schedule).** A solution $(x,z,u,s,\delta)$ satisfies (1) to (14) if and only if the arcs with $x_{ij}=1$ form at most $m$ routes that partition $N$, each capacity-feasible, for which a schedule exists that respects the windows, the horizon, and the slot-dependent travel times. A route has such a schedule if and only if the evaluation rule above produces a feasible one. The rule gives the componentwise earliest start times.

*Proof.* ($\Rightarrow$) By (1) and (2), every customer has one selected incoming and one selected outgoing arc, so the selected arcs form paths from $0$ to $n+1$ plus possibly cycles in $N$. By Proposition 2 there are no cycles. The number of paths is $\sum_jx_{0j}\le m$. Along a path, (5) and (6) bound the load by $Q$. By (7) each used arc has one slot $p$, by (9) and (10) its departure time $\delta_i$ lies in slot $p$, by (8) $\delta_i\ge s_i+d_i$, and by (11) $s_j\ge\delta_i+t_{ij}^p$. With (12) this is a valid schedule.

($\Leftarrow$) Given a route that is feasible under the evaluation rule, let $p$ be a slot attaining the minimum for each arc, and set $\delta_i=\max(r_i,\theta_{p-1})$, $z_{ij}^p=1$, $s_j$ as in the rule, and $u_i$ the cumulative load. All constraints hold on used arcs by construction. On unused arcs, (9) and (10) hold because $z=0$ and $0\le\delta_i\le H$, (11) holds by Proposition 3, and (5) reduces to $u_i-u_j\le Q-q_j$, which follows from $u_i\le Q$ and $u_j\ge q_j$.

Earliest schedule: the function $r\mapsto\text{arr}_{ij}(r)$ is nondecreasing, because every term in the minimum is nondecreasing in $r$ and the set of admissible slots can only shrink as $r$ grows. So an earlier ready time never gives a later arrival, and the rule yields the smallest possible $s_j$ at every customer. Since the objective (0) does not depend on times, and waiting is free, any other schedule of the same route is feasible only if the earliest one is. $\square$

**Proposition 2 (no subtours).** Constraints (5) and (6) forbid any cycle $C\subseteq N$ with $x_{ij}=1$ on all its arcs.

*Proof.* Summing (5) over the arcs of $C$ gives $\sum_{(i,j)\in C}(u_i-u_j)+Q|C|\le Q|C|-\sum_{j\in C}q_j$. The first sum is $0$, because each vertex of the cycle appears once as $i$ and once as $j$. This leaves $0\le-\sum_{j\in C}q_j<0$ by (A3), a contradiction. $\square$

The time constraints exclude cycles too when $d_i+t_{ij}^p>0$ on every arc, by the same summation. They do not when travel and service times can both be zero, so correctness relies on (5) and (A3).

**Proposition 3 (the value of $M_{ij}^p$).** If $z_{ij}^p=0$, constraint (11) holds for every $s,\delta$ satisfying (12) and (13) as long as $M_{ij}^p\ge\hat H_i+t_{ij}^p-a_j$, and no smaller constant has this property.

*Proof.* The left side of (11) is largest when $\delta_i$ is largest and $s_j$ smallest. By (13) and (8), $\delta_i\le\hat H_i$, and by (12) $s_j\ge a_j$, so the left side is at most $\hat H_i+t_{ij}^p-a_j$, and this value can be reached. $\square$

The constant is computed for each arc and slot. A single loose global $M$ would make the LP relaxation much weaker and can cause numerical trouble. Because vehicles may idle, $\delta_i$ can be as large as $H$, which makes $M_{ij}^p$ larger than in a static model. This is the main reason the traffic model is harder for the solver.

**Preprocessing (arc elimination).** An arc can be removed without changing the optimum when no feasible route can use it. Because $\rho_p\ge1$, the best case for travel time is the free-flow value $t_{ij}$ (and $\min_p t_{ij}^p$ in general).

* $(i,j)$ with $i,j\in N$ and $q_i+q_j>Q$ (capacity);
* $(i,j)$ with $a_i+d_i+\min_pt_{ij}^p>b_j$ (even the best departure arrives too late);
* $(0,j)$ with $\min_pt_{0j}^p>b_j$, and $(i,n+1)$ with $a_i+d_i+\min_pt_{i0}^p>H$.

A customer left with no incoming or no outgoing arc proves the instance infeasible. The same test, applied with the peak time $\rho\,t_{ij}$ in place of the free-flow time, shows which arcs traffic makes unusable in the worst case.

**Infeasibility.** The instance is infeasible if some $q_i>Q$, if some customer cannot be reached or left in time, or if $m$ is smaller than the number of routes any feasible solution needs.

### 2.9 Additional constraints and modelling choices

| Choice | Decision | Reason |
|---|---|---|
| Time windows | Hard, one per customer | Required by Level A. |
| Waiting | Allowed before and after service, not counted in the objective | Required by Level A; keeps the model consistent under peak traffic (2.3). |
| Traffic | Piecewise constant by departure slot; simple peak with factor $\rho$; general $t_{ij}^p$ allowed | Linear model; the road layer can supply segment-level values. |
| Speed change during a trip | Ignored | A trip started just before the peak ends is not slowed down or sped up midway. The step-speed model of Ichoua et al. [9] removes this, at the cost of a piecewise-linear arrival time. We list it as a limitation. |
| Objective | Free-flow cost $c_{ij}=t_{ij}$ | Keeps the earliest-start property and comparability with benchmarks. The experienced-time variant is in 2.5. |
| Service duration $d_i$ | Included, $\ge0$ | Present in the standard benchmarks. Set $d_i=0$ to ignore it. |
| Planning horizon | Window $[0,H]$ on the depot | Bounds route duration and defines the slots. |
| Fleet | Homogeneous, at most $m$ vehicles | Stated in the project guidelines. |
| Depot | Single | Guidelines and benchmarks. |
| Delivery | One visit per customer, no split delivery | (A4). |
| Number of vehicles | Bounded by $m$, optionally minimized with $f$ | To be fixed with the team. |

Not modelled: multiple depots, heterogeneous fleets, multiple windows per customer, pickup and delivery, driver breaks, stochastic demand or travel times, and live (unforeseen) traffic changes.

**Why the two-index formulation.** The guidelines specify identical vehicles, so a vehicle index $k$ adds no information. The three-index model $x_{ijk}$ would need about $n^2m$ binaries (times $P$ with traffic) and would contain up to $m!$ equivalent copies of each solution, which slows branch and bound. The two-index model has a size independent of $m$ and one start time, one departure time and one load per customer. The set partitioning formulation (one binary per feasible route, solved by column generation) has a much stronger LP bound and underlies the best exact methods. With traffic, route feasibility is checked by the evaluation rule of 2.8 when a route is generated. We cite it for bounds and do not implement it.

### 2.10 Expected output

The output is the set of routes (ordered lists of customers), the start time $s_i$ and departure time $\delta_i$ at every customer, the departure slot of every used arc, the number of vehicles used, and the total cost $Z$.

A solution is feasible if and only if:

* every customer is served exactly once;
* every route starts and ends at the depot, and at most $m$ routes are used;
* no route carries more than $Q$;
* every service starts in $[a_i,b_i]$, every vehicle is back by $H$, and every trip takes the time of the slot in which the vehicle leaves.

### 2.11 Applications of the exact model

1. **Direct solving of small instances.** A MILP solver returns a provably optimal solution. The size it can handle depends strongly on how tight the windows are, because tight windows remove many arcs. Traffic multiplies the number of binaries by about $P+1$, so the limit is lower than for the static model.
2. **Benchmark for the heuristic.** On every instance the solver can close, the optimal cost gives an exact optimality gap $100\,(C_{\text{heur}}-C^*)/C^*$.
3. **Lower bound.** Relaxing the binaries to $[0,1]$ gives the LP bound $Z_{LP}\le C^*$. It is cheap but weak because of the big-M constraints. When the solver stops at a time limit, its dual bound is also a lower bound.
4. **Upper bound.** Stopped early, the solver's best solution is feasible and gives an upper bound. A heuristic solution can be supplied as a warm start.
5. **Validation of the heuristic.** Constraints (1) to (12) are an executable definition of feasibility. Any solution of the metaheuristic must satisfy them, and its route-evaluation function (2.8) must agree with them.
6. **Test oracle for neighbourhood operators.** On small instances, a move declared feasible by the heuristic can be checked against the model.
7. **Measuring the effect of traffic and of time windows.** Solving the same instance with $\rho=1$ and with $\rho>1$ gives the exact cost of congestion. Solving with and without the window constraints gives the exact cost of the windows.
8. **Matheuristic component (optional).** Re-optimizing a few routes with the ILP while the rest stay fixed is a possible repair step for a large-neighbourhood search.

### 2.12 Validation plan ("complete and corrected")

The model will be implemented in Python (PuLP with CBC or HiGHS) and checked as follows.

1. **Brute force.** On random instances with 5 to 7 customers, compare the ILP optimum with an independent enumeration of all orderings and splits into at most $m$ routes, using the evaluation rule of 2.8 as a separate feasibility check.
2. **Static consistency.** With $P=1$ or $\rho=1$, the optimum must equal that of the static CVRPTW model (the version without constraints (7) to (11)).
3. **Traffic effect.** Raising $\rho$ must never lower the optimal cost. On a hand-made example, a customer reachable only before the peak must force the expected order, and a customer whose window lies inside the peak must show the longer trip.
4. **Waiting out the peak.** Build a case where leaving after the peak ends is the only feasible option, and check that the model idles the vehicle.
5. **Variants.** Results must be identical with and without arc preprocessing, and with integer times.
6. **Edge cases.** Demand above $Q$, $m$ too small, a window that closes before the customer can be reached, a horizon too short for the return trip: all must be reported infeasible. A zero-width window must give service at exactly $a_i$.
7. **LP bound.** The relaxation value must never exceed the optimum.
8. **Cross-check.** If possible, solve a small benchmark instance ($\rho=1$) and compare with the best-known value.

### 2.13 Interface with the other sections

| Section | What it receives | Point to agree on |
|---|---|---|
| 3. Data representation | The symbols of 2.2, the instance fields $n,m,Q,q,a,b,d,H,\theta_p,t_{ij},\rho_p$ (or the full array $t_{ij}^p$), and the depot copy $n+1$ | Storage of the traffic data (factor per slot, or one matrix per slot); how the peak profile is generated for benchmark instances; distance rounding |
| 4. Theoretical study | The decision version: is there a set of at most $m$ feasible routes of total cost at most $B$? | The definition of a feasible route must match 2.10 exactly. The static problem is the special case $\rho=1$, so hardness carries over. |
| 6. Solution method | The route-evaluation rule of 2.8 and the objective | Is $m$ a hard bound or minimized first? Which objective (2.5)? Are infeasible intermediate solutions allowed with a penalty? |
| Experiments | The ILP as a reference for small instances; $\rho$ and the peak position as experimental factors | Gap formula, peak parameters, and the instances used |

### 2.14 References

1. Dantzig, G. B. and Ramser, J. H. (1959). The truck dispatching problem. *Management Science*, 6(1), 80-91.
2. Miller, C. E., Tucker, A. W. and Zemlin, R. A. (1960). Integer programming formulation of traveling salesman problems. *Journal of the ACM*, 7(4), 326-329.
3. Desrochers, M. and Laporte, G. (1991). Improvements and extensions to the Miller-Tucker-Zemlin subtour elimination constraints. *Operations Research Letters*, 10(1), 27-36.
4. Solomon, M. M. (1987). Algorithms for the vehicle routing and scheduling problems with time window constraints. *Operations Research*, 35(2), 254-265.
5. Desrochers, M., Desrosiers, J. and Solomon, M. (1992). A new optimization algorithm for the vehicle routing problem with time windows. *Operations Research*, 40(2), 342-354.
6. Savelsbergh, M. W. P. (1985). Local search in routing problems with time windows. *Annals of Operations Research*, 4, 285-305.
7. Kallehauge, B. (2008). Formulations and exact algorithms for the vehicle routing problem with time windows. *Computers & Operations Research*, 35(7), 2307-2330.
8. Toth, P. and Vigo, D. (eds.) (2014). *Vehicle Routing: Problems, Methods, and Applications*, 2nd ed. SIAM.
9. Ichoua, S., Gendreau, M. and Potvin, J.-Y. (2003). Vehicle dispatching with time-dependent travel times. *European Journal of Operational Research*, 144(2), 379-396.
10. Malandraki, C. and Daskin, M. S. (1992). Time dependent vehicle routing problems: formulations, properties and heuristic algorithms. *Transportation Science*, 26(3), 185-200.

References 1 and 4 define the problem and its benchmarks. References 2 and 3 are the source of the load constraints (5) and (6). Reference 5 gives the set partitioning formulation, and 7 and 8 survey the exact methods. References 9 and 10 cover time-dependent travel times. These two were written from memory: check volume and page numbers before the final report, cite every reference by number in the text, and add the last-access date for any URL.

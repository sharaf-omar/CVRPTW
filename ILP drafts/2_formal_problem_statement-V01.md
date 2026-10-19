## 2. Formal problem statement

> Problem version: Level A (Advanced), the Capacitated Vehicle Routing Problem with Time Windows (CVRPTW), with peak-hour traffic.
> Every route respects the vehicle capacity and every customer's time window. Late service is forbidden. A vehicle that arrives early waits. Travel times are longer during a peak period.

### 2.1 Problem definition

A fleet of identical vehicles delivers goods from one depot to a set of customers. Each customer has a known demand and a time window in which service must start. Every customer is served exactly once, every route starts and ends at the depot, and no vehicle carries more than its capacity. A vehicle that arrives before a window opens waits until it opens. The goal is to find the set of routes with the smallest total cost. This is the vehicle routing problem of Dantzig and Ramser [1] with the time windows studied by Solomon [4]; surveys of its formulations and exact methods are in [7, 8, 9].

Road conditions change during the day, so we add the simplest model of this. The horizon is cut into time slots (off-peak, peak, off-peak) and the travel time between two locations depends on the slot in which the vehicle leaves. A trip of 20 minutes off-peak may take 30 minutes at the peak. This changes which routes are feasible: a customer reachable in time at free flow may be unreachable at the peak, and a vehicle may do better by waiting for the peak to end.

The setting is urban delivery. A solution decides three things together: which customers each vehicle serves, in what order, and when each service starts.

Assumptions:

* (A1) One depot. The vehicles are identical, with capacity $Q$.
* (A2) Demands, service durations, time windows and travel times are known and deterministic.
* (A3) Every demand satisfies $1\le q_i\le Q$. A customer with $q_i>Q$ makes the instance infeasible.
* (A4) Each customer is served in one visit by one vehicle. Deliveries are not split.
* (A5) Time windows are hard. Waiting, before and after service, is allowed and costs nothing.
* (A6) At most $m$ vehicles are used, each on one route. Routes start at time $0$ and end by the horizon $H$. Starting at $0$ loses nothing under the objective of 2.5, because an earlier departure never causes a later arrival (Section 2.8).
* (A7) The travel time of a trip is fixed by the slot in which the vehicle leaves. Speed changes during a trip are ignored.

### 2.2 Notation

The problem lives on a directed graph $G=(V,A)$. The depot appears twice: node $0$ starts every route and node $n+1$ is a copy where every route ends. This lets the depot take part in the time constraints like any other node.

$$N=\{1,\dots,n\},\qquad V=\{0\}\cup N\cup\{n+1\},$$

$$A=\{(0,j):j\in N\}\ \cup\ \{(i,j):i,j\in N,\ i\ne j\}\ \cup\ \{(i,n+1):i\in N\}.$$

There is no arc into $0$, none out of $n+1$, and no arc $(0,n+1)$, so every route visits at least one customer. The horizon $[0,H]$ is cut at $0=\theta_0<\theta_1<\dots<\theta_P=H$. Slot $p$ is the closed interval $[\theta_{p-1},\theta_p]$, and $\mathcal{P}=\{1,\dots,P\}$.

| Parameter | Domain | Definition |
|---|---|---|
| $n$, $m$, $Q$ | $\mathbb{N}$ | Number of customers, maximum number of vehicles, vehicle capacity |
| $q_i$ | $1\le q_i\le Q$ ($i\in N$); $q_0=q_{n+1}=0$ | Demand of customer $i$ |
| $a_i,\ b_i$ | $0\le a_i\le b_i$ | Earliest and latest permitted start of service at $i$ |
| $d_i$ | $\ge0$; $d_0=d_{n+1}=0$ | Service duration at $i$ |
| $H$ | $>0$ | End of the horizon. The depot window is $[0,H]$, so $[a_0,b_0]=[a_{n+1},b_{n+1}]=[0,H]$ |
| $P$, $\theta_p$ | $\theta_0=0<\dots<\theta_P=H$ | Number of slots and their boundaries |
| $t_{ij}$ | $\ge0$ | Free-flow travel time from $i$ to $j$ |
| $\rho_p$ | $\ge1$ | Congestion factor of slot $p$ ($1$ means free flow) |
| $t_{ij}^p$ | $\ge0$ | Travel time from $i$ to $j$ when leaving in slot $p$; in the peak model $t_{ij}^p=\rho_p t_{ij}$ |
| $c_{ij}$ | $\ge0$ | Cost of arc $(i,j)$, equal to $t_{ij}$ |
| $M_{ij}^p$ | $\ge0$ | Big-M constant of arc $(i,j)$ and slot $p$, defined in (13) |

All times share one unit, for example minutes from the start of the horizon. On benchmark instances $t_{ij}$ is the Euclidean distance, rounded as the benchmark prescribes.

### 2.3 Traffic model

Customers are not joined by single road segments, so traffic is handled in two layers. In a preprocessing layer, every road segment has an expected speed in each slot, and a shortest path with the speeds of that slot gives $t_{ij}^p$ for each pair of locations. The routing layer (the ILP and the heuristic) only sees the values $t_{ij}^p$. This is the standard way to build time-dependent travel times [12, 13], with a piecewise-constant travel-time function in the line of Malandraki and Daskin [11].

The default profile has three slots:

| Slot $p$ | Interval | Factor $\rho_p$ |
|---|---|---|
| 1 (off-peak) | $[0,\theta_1]$ | $1$ |
| 2 (peak) | $[\theta_1,\theta_2]$ | $\rho\ge1$, default $1.5$ |
| 3 (off-peak) | $[\theta_2,H]$ | $1$ |

Take $t_{ij}=20$, $\theta_1=120$, $\theta_2=240$ and $\rho=1.5$. Leaving at time $100$ takes $20$, leaving at $130$ takes $30$, and leaving at $240$ takes $20$ again. Slots are closed intervals, so a departure exactly at a boundary may use either neighbouring slot, and the faster one is taken. The ILP and the evaluation rule of 2.8 follow the same convention.

Waiting after service matters. With $t_{ij}=40$ and a vehicle ready at time $230$, leaving at once arrives at $230+60=290$. Waiting until $240$ arrives at $240+40=280$. A model that forced vehicles to leave as soon as they finish would make such a vehicle arrive later than one that could wait. Allowing waiting also removes the usual FIFO difficulty of step-wise travel times, where a later departure could arrive earlier (see 2.8). Ichoua et al. [10] avoid it by changing speed inside a trip, which we do not model.

Two special cases are used for validation. Setting $\rho=1$ (or $P=1$) gives the static CVRPTW. Setting also $a_i=0$, $b_i=H$ with $H$ large and $d_i=0$ gives the plain CVRP, which is what the vrplib instances require.

### 2.4 Decision variables

| Variable | Domain | Meaning |
|---|---|---|
| $x_{ij}$ | $\{0,1\}$, $(i,j)\in A$ | 1 if a vehicle drives directly from $i$ to $j$ |
| $z_{ij}^p$ | $\{0,1\}$, $(i,j)\in A$, $p\in\mathcal{P}$ | 1 if arc $(i,j)$ is used and the vehicle leaves $i$ in slot $p$ |
| $s_i$ | $[a_i,b_i]$, $i\in V$; $s_0=0$ | Start of service at $i$ ($s_{n+1}$ is the return time) |
| $\delta_i$ | $[0,H]$, $i\in\{0\}\cup N$; $\delta_0=0$ | Time at which the vehicle leaves $i$ |
| $u_i$ | $[q_i,Q]$, $i\in N$ | Cumulative demand delivered on the route up to and including $i$ |

No variable has a vehicle index. Each customer is visited once, so one $s_i$, one $\delta_i$ and one $u_i$ per customer are enough. Routes are read off $x$ by starting at node $0$ and following selected arcs to node $n+1$. Since $x_{ij}=\sum_pz_{ij}^p$, the variables $x$ are redundant. We keep them because they make the degree and load constraints easier to read.

### 2.5 Objective function

$$\min\ Z=\sum_{(i,j)\in A}c_{ij}\,x_{ij},\qquad c_{ij}=t_{ij}.$$

The objective is the total free-flow travel time, which equals distance on the benchmarks. Waiting is not counted, and the number of vehicles is bounded by $m$ but not penalised. Traffic therefore acts only through the time constraints: it can make a route infeasible, force a wait, or force a different order, but it does not change the cost of an arc. We chose this because it keeps the results comparable with the static benchmarks, and because it makes an earlier schedule never worse than a later one.

Two variants are available.

* Experienced travel time: minimise $\sum_{(i,j)\in A}\sum_{p}t_{ij}^p z_{ij}^p$. The model stays linear, but waiting out the peak can now lower the cost. The depot departure then also becomes a decision, with one departure variable per route (for example one per arc $(0,j)$) in place of the single $\delta_0=0$.
* Fleet size first: add $f\sum_{j\in N}x_{0j}$ to $Z$, with $f>\sum_{(i,j)\in A}c_{ij}$.

### 2.6 Complete ILP

$$\min\ \sum_{(i,j)\in A} c_{ij}\,x_{ij} \tag{0}$$

subject to

$$\sum_{j:(i,j)\in A} x_{ij}=1 \qquad \forall\, i\in N \tag{1}$$

$$\sum_{i:(i,j)\in A} x_{ij}=1 \qquad \forall\, j\in N \tag{2}$$

$$\sum_{j\in N} x_{0j}\ \le\ m \tag{3}$$

$$u_i-u_j+Q\,x_{ij}\ \le\ Q-q_j \qquad \forall\, i,j\in N,\ i\ne j \tag{4}$$

$$q_i\ \le\ u_i\ \le\ Q \qquad \forall\, i\in N \tag{5}$$

$$\sum_{p\in\mathcal{P}} z_{ij}^p = x_{ij} \qquad \forall\, (i,j)\in A \tag{6}$$

$$\delta_i\ \ge\ s_i+d_i\ \ \forall\, i\in N,\qquad \delta_0=0 \tag{7}$$

$$\delta_i\ \ge\ \theta_{p-1}\,z_{ij}^p \qquad \forall\, (i,j)\in A,\ p\in\mathcal{P} \tag{8}$$

$$\delta_i\ \le\ \theta_p+(H-\theta_p)\,(1-z_{ij}^p) \qquad \forall\, (i,j)\in A,\ p\in\mathcal{P} \tag{9}$$

$$\delta_i+t_{ij}^p-s_j\ \le\ M_{ij}^p\,(1-z_{ij}^p) \qquad \forall\, (i,j)\in A,\ p\in\mathcal{P} \tag{10}$$

$$a_i\ \le\ s_i\ \le\ b_i \quad \forall\, i\in V,\qquad s_0=0 \tag{11}$$

$$x_{ij},\,z_{ij}^p\in\{0,1\},\qquad 0\le\delta_i\le H \tag{12}$$

$$M_{ij}^p=\max\{\,0,\ \hat H_i+t_{ij}^p-a_j\,\},\qquad \hat H_0=0,\quad \hat H_i=H\ \ (i\in N). \tag{13}$$

| # | Meaning |
|---|---|
| (0) | Minimise the total free-flow travel time of all vehicles. |
| (1), (2) | Every customer is left once and entered once, so it is visited exactly once. |
| (3) | At most $m$ vehicles leave the depot. |
| (4) | If $j$ directly follows $i$, then $u_j\ge u_i+q_j$: the load accumulates along the route. When $x_{ij}=0$ the constraint always holds. It follows the load-based form of Miller, Tucker and Zemlin [2] and Desrochers and Laporte [3]. |
| (5) | The load after serving $i$ is at least $q_i$ and at most $Q$. |
| (6) | A used arc has exactly one departure slot, and an unused arc has none. |
| (7) | A vehicle cannot leave a customer before finishing service, and it may idle longer. Vehicles leave the depot at time $0$. |
| (8), (9) | If $z_{ij}^p=1$, the departure time $\delta_i$ lies in $[\theta_{p-1},\theta_p]$. If $z_{ij}^p=0$ both hold for any $\delta_i\in[0,H]$. |
| (10) | If $z_{ij}^p=1$, then $s_j\ge\delta_i+t_{ij}^p$: service at $j$ starts after the drive of the chosen slot. Being an inequality, it allows waiting before service. |
| (11) | Service starts inside the window. Early means waiting until $a_i$. With $i=n+1$ it forces the return by $H$. |
| (12) | Domains. The variables $s$ and $u$ are continuous and bounded by (5) and (11). |
| (13) | The smallest constant that keeps (10) valid when $z_{ij}^p=0$ (Proposition 2). |

The model has $(n^2+n)(P+1)$ binary variables, about $3n$ continuous variables, and $O(n^2P)$ constraints. None of these counts depends on $m$. With the three-slot profile there are about four times as many binaries as in the static model. Strictly the model is a mixed-integer program because $s$, $\delta$ and $u$ are continuous. When all data are integers, an earliest schedule has integer times, so these variables can be declared integer without changing the optimum, which gives a pure ILP.

### 2.7 Evaluation rule

Given a route $0\to i_1\to\dots\to i_r\to n+1$, set $s_0=0$. For each arc $(i,j)$ let $r_i=s_i+d_i$ be the time the vehicle is ready to leave $i$. Then

$$\text{arr}_{ij}(r_i)=\min_{p:\ \max(r_i,\theta_{p-1})\le\theta_p}\Big\{\max(r_i,\theta_{p-1})+t_{ij}^p\Big\},\qquad s_j=\max\{a_j,\ \text{arr}_{ij}(r_i)\}.$$

The route is feasible if every $s_j\le b_j$, $s_{n+1}\le H$, and the load is at most $Q$. The rule tries each slot that has not ended, leaves as early as that slot allows, and keeps the earliest arrival. It costs $O(P)$ per arc. The heuristic of Section 6 uses it to evaluate routes, and the ILP is its exact counterpart.

### 2.8 Correctness

**Proposition 1 (no subtours).** Constraints (4) and (5) forbid any cycle $C\subseteq N$ with $x_{ij}=1$ on all its arcs.

*Proof.* Summing (4) over the arcs of $C$ gives $\sum_{(i,j)\in C}(u_i-u_j)+Q|C|\le Q|C|-\sum_{j\in C}q_j$. The first sum is $0$, because each node of the cycle appears once as $i$ and once as $j$. This leaves $0\le-\sum_{j\in C}q_j<0$ by (A3), a contradiction. $\square$

**Proposition 2 (value of $M_{ij}^p$).** If $z_{ij}^p=0$, constraint (10) holds for all $s,\delta$ satisfying (11) and (12) exactly when $M_{ij}^p\ge\hat H_i+t_{ij}^p-a_j$.

*Proof.* The left side of (10) is largest when $\delta_i$ is largest and $s_j$ smallest. By (12) and (7), $\delta_i\le\hat H_i$, and by (11) $s_j\ge a_j$. So the left side is at most $\hat H_i+t_{ij}^p-a_j$, and this value is reached. $\square$

The constant is computed for each arc and slot. A single global $M$ would weaken the LP relaxation and can cause numerical trouble. Because vehicles may idle, $\delta_i$ can be as large as $H$, which makes $M_{ij}^p$ larger than in the static model. This is the main reason the traffic model is harder for a solver.

**Proposition 3 (equivalence with the evaluation rule).** A vector $(x,z,u,s,\delta)$ satisfies (1) to (13) if and only if the arcs with $x_{ij}=1$ form at most $m$ routes that partition $N$ and that are all feasible under the rule of 2.7. The rule gives the earliest possible start times.

*Proof.* ($\Rightarrow$) By (1) and (2) the selected arcs form paths from $0$ to $n+1$ plus possibly cycles in $N$, and Proposition 1 removes the cycles. By (3) there are at most $m$ paths. Along a path, (4) and (5) bound the load by $Q$. For a used arc, (6) gives one slot $p$, (8) and (9) put $\delta_i$ in that slot, (7) gives $\delta_i\ge s_i+d_i$, and (10) gives $s_j\ge\delta_i+t_{ij}^p$. With (11) this is a schedule that respects the windows and the horizon. The rule yields start times no later than any such schedule (see the monotonicity argument below), so the route is feasible under it.

($\Leftarrow$) For a route feasible under the rule, take for each arc the slot $p$ that attains the minimum, and set $\delta_i=\max(r_i,\theta_{p-1})$, $z_{ij}^p=1$, $s_j$ as in the rule, and $u_i$ the cumulative load. All constraints hold on used arcs by construction. On unused arcs, (8) and (9) hold because $z=0$, (10) holds by Proposition 2, and (4) reduces to $u_i-u_j\le Q-q_j$, which follows from $u_i\le Q$ and $u_j\ge q_j$.

Monotonicity: $r\mapsto\text{arr}_{ij}(r)$ is nondecreasing. Each term of the minimum is nondecreasing in $r$, and the set of admissible slots can only shrink as $r$ grows, which can only raise the minimum. So an earlier ready time never gives a later arrival, the rule gives the smallest $s_j$ at every customer, and any schedule of the same route is feasible only if the earliest one is. $\square$

**Preprocessing.** An arc can be removed without changing the optimum if no feasible route can use it. Because $\rho_p\ge1$, the best travel time is $\min_pt_{ij}^p$. We remove $(i,j)$ with $i,j\in N$ if $q_i+q_j>Q$ or $a_i+d_i+\min_pt_{ij}^p>b_j$, remove $(0,j)$ if $\min_pt_{0j}^p>b_j$, and remove $(i,n+1)$ if $a_i+d_i+\min_pt_{i,n+1}^p>H$. A customer left with no incoming or no outgoing arc proves the instance infeasible. Tight windows remove many arcs, which is why they make the exact model easier.

**Infeasibility.** An instance is infeasible if some $q_i>Q$, if some customer cannot be reached or left in time, or if $m$ is smaller than the number of routes any feasible solution needs. With $m$ fixed, finding a feasible solution is already hard in general: for $m=1$ it is the travelling salesman problem with time windows, whose feasibility version is NP-complete [6].

### 2.9 Additional constraints and modelling choices

| Choice | Decision | Reason |
|---|---|---|
| Time windows | Hard, one per customer | Required by Level A. |
| Waiting | Allowed before and after service, free | Required by Level A; keeps the peak model consistent (2.3). |
| Traffic | Piecewise constant by departure slot; simple peak with factor $\rho$; general $t_{ij}^p$ allowed | Linear model; the road layer can supply segment-level values. |
| Speed change inside a trip | Ignored | The step-speed model of Ichoua et al. [10] removes this at the cost of piecewise-linear arrival times. Listed as a limitation. |
| Objective | Free-flow cost | Comparable with benchmarks; keeps the earliest-schedule property. |
| Service duration $d_i$ | Included, $\ge0$ | Present in the standard benchmarks; set $d_i=0$ to ignore it. |
| Fleet | Identical vehicles, at most $m$ | Project guidelines. $m$ is a hard bound. |
| Depot | Single | Guidelines and benchmarks. |

Not modelled: several depots, mixed fleets, several windows per customer, pickups and deliveries, driver breaks, random demand or travel times, and traffic changes that are not known in advance.

We use a two-index formulation. The vehicles are identical, so a vehicle index $k$ adds nothing. A three-index model $x_{ijk}$ would need about $n^2m$ binaries (times $P$ with traffic) and would contain up to $m!$ equivalent copies of each solution, which slows branch and bound. The load-based constraints (4) are compact but give a weak LP bound. The set partitioning formulation, with one binary per feasible route and solved by column generation, has a much stronger bound and underlies the best exact methods [5, 8]. With traffic, route feasibility would be checked by the rule of 2.7 when a column is generated. We cite it for bounds and do not implement it.

### 2.10 Expected output

The output is the set of routes (ordered lists of customers), the start time $s_i$ and departure time $\delta_i$ at each customer, the departure slot of each used arc, the number of vehicles used, and the total cost $Z$. A solution is feasible if and only if:

* every customer is served exactly once;
* every route starts and ends at the depot, and at most $m$ routes are used;
* no route carries more than $Q$;
* every service starts in $[a_i,b_i]$, every vehicle is back by $H$, and every trip takes the time of the slot in which the vehicle leaves.

### 2.11 Uses of the exact model

1. Direct solving of small instances, where a MILP solver returns a provably optimal solution. The reachable size depends on how tight the windows are, and it is smaller than for the static model because traffic multiplies the binaries by about $P+1$.
2. Reference for the heuristic. On every instance the solver closes, the optimal cost $C^*$ gives the exact gap $100\,(C_{\text{heur}}-C^*)/C^*$.
3. Lower bound. The LP relaxation gives $Z_{LP}\le C^*$. It is cheap and weak because of the big-M constraints. When the solver stops at a time limit, its dual bound is also a lower bound.
4. Upper bound. A solver stopped early returns a feasible solution, and a heuristic solution can be passed as a warm start.
5. Test oracle. Constraints (1) to (13) are an executable definition of feasibility. The route evaluation and the moves of the heuristic can be checked against them on small instances.
6. Cost of congestion and of windows. Solving one instance with $\rho=1$ and with $\rho>1$ gives the exact cost of congestion, and solving with and without the window constraints gives the exact cost of the windows.
7. Matheuristic step (optional). Re-optimising a few routes with the ILP while the others stay fixed can serve as a repair step in a large neighbourhood search.

### 2.12 Validation of the model

The model will be implemented in Python with PuLP (CBC or HiGHS) and checked as follows.

1. On random instances with 5 to 7 customers, the ILP optimum must equal an independent enumeration of all orderings and splits into at most $m$ routes, using the rule of 2.7 as a separate feasibility test.
2. With $\rho=1$ the optimum must equal that of the static CVRPTW model (without constraints (6) to (10) and with $\delta$ removed), and raising $\rho$ must never lower the optimum.
3. A hand-made case where leaving after the peak is the only feasible option must show the vehicle idling.
4. Demand above $Q$, $m$ too small, a window that closes before the customer can be reached, and a horizon too short for the return trip must all be reported infeasible. A zero-width window must give service at exactly $a_i$.
5. The LP value must never exceed the optimum, and results must be identical with and without arc preprocessing.
6. A small benchmark instance with $\rho=1$ is solved and compared with its best-known value.

### 2.13 References

1. Dantzig, G. B. and Ramser, J. H. (1959). The truck dispatching problem. *Management Science*, 6(1), 80-91.
2. Miller, C. E., Tucker, A. W. and Zemlin, R. A. (1960). Integer programming formulation of traveling salesman problems. *Journal of the ACM*, 7(4), 326-329.
3. Desrochers, M. and Laporte, G. (1991). Improvements and extensions to the Miller-Tucker-Zemlin subtour elimination constraints. *Operations Research Letters*, 10(1), 27-36.
4. Solomon, M. M. (1987). Algorithms for the vehicle routing and scheduling problems with time window constraints. *Operations Research*, 35(2), 254-265.
5. Desrochers, M., Desrosiers, J. and Solomon, M. (1992). A new optimization algorithm for the vehicle routing problem with time windows. *Operations Research*, 40(2), 342-354.
6. Savelsbergh, M. W. P. (1985). Local search in routing problems with time windows. *Annals of Operations Research*, 4, 285-305.
7. Cordeau, J.-F., Desaulniers, G., Desrosiers, J., Solomon, M. M. and Soumis, F. (2002). VRP with time windows. In Toth, P. and Vigo, D. (eds), *The Vehicle Routing Problem*, SIAM, 157-193. https://doi.org/10.1137/1.9780898718515.ch7 (accessed 18 October 2026).
8. Kallehauge, B., Larsen, J., Madsen, O. B. G. and Solomon, M. M. (2005). Vehicle routing problem with time windows. In Desaulniers, G., Desrosiers, J. and Solomon, M. M. (eds), *Column Generation*, Springer, 67-98. https://doi.org/10.1007/0-387-25486-2_3 (accessed 18 October 2026).
9. Toth, P. and Vigo, D. (eds) (2014). *Vehicle Routing: Problems, Methods, and Applications*, 2nd ed. SIAM.
10. Ichoua, S., Gendreau, M. and Potvin, J.-Y. (2003). Vehicle dispatching with time-dependent travel times. *European Journal of Operational Research*, 144(2), 379-396.
11. Malandraki, C. and Daskin, M. S. (1992). Time dependent vehicle routing problems: formulations, properties and heuristic algorithms. *Transportation Science*, 26(3), 185-200.
12. Kok, A. L., Hans, E. W. and Schutten, J. M. J. (2012). Vehicle routing under time-dependent travel times: the impact of congestion avoidance. *Computers & Operations Research*, 39(5), 910-918. https://doi.org/10.1016/j.cor.2011.05.027 (accessed 18 October 2026).
13. Gendreau, M., Ghiani, G. and Guerriero, E. (2015). Time-dependent routing problems: a review. *Computers & Operations Research*, 64, 189-197. https://doi.org/10.1016/j.cor.2015.06.001 (accessed 18 October 2026).

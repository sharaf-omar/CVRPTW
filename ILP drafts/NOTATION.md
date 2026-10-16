# Shared Notation: CVRPTW (Level A) with peak traffic, two-index formulation

Every section of the project uses these symbols. Please do not rename them locally. If something must change, change it here first.

## Sets and graph

| Symbol | Meaning |
|---|---|
| $N=\{1,\dots,n\}$ | Customers |
| $0$ | Start depot (every route begins here) |
| $n+1$ | End depot, a copy of the depot (every route ends here) |
| $V=\{0\}\cup N\cup\{n+1\}$ | All nodes |
| $A$ | Arcs: $(0,j)$, $(i,j)$ with $i\ne j$ in $N$, and $(i,n+1)$. No arc into $0$, out of $n+1$, or $(0,n+1)$ |
| $\mathcal{P}=\{1,\dots,P\}$ | Traffic time slots; slot $p$ is the interval $[\theta_{p-1},\theta_p]$ |

## Input parameters

| Symbol | Domain | Meaning |
|---|---|---|
| $n$ | $\mathbb{N}$ | Number of customers |
| $m$ | $\mathbb{N}$ | Maximum number of vehicles |
| $Q$ | $\mathbb{N}$ | Vehicle capacity (identical vehicles) |
| $q_i$ | $1\le q_i\le Q$; $q_0=q_{n+1}=0$ | Demand of customer $i$ |
| $a_i$, $b_i$ | $a_i\le b_i$ | Earliest and latest permitted **start of service** at $i$ |
| $d_i$ | $\ge0$; $d_0=d_{n+1}=0$ | Service duration at $i$ |
| $H$ | $>0$ | End of the planning horizon; depot window is $[0,H]$ |
| $P$, $\theta_p$ | $\theta_0=0<\dots<\theta_P=H$ | Number of slots and slot boundaries |
| $t_{ij}$ | $\ge0$ | Free-flow travel time from $i$ to $j$ |
| $\rho_p$ | $\ge1$ | Congestion factor of slot $p$ (1 = free flow) |
| $t_{ij}^p$ | $\ge0$ | Travel time from $i$ to $j$ when leaving in slot $p$; simple peak: $t_{ij}^p=\rho_p t_{ij}$ |
| $c_{ij}$ | $\ge0$ | Cost of arc $(i,j)$, $c_{ij}=t_{ij}$ (free-flow) |
| $M_{ij}^p$ | $\ge0$ | Big-M of arc $(i,j)$ and slot $p$: $\max\{0,\ \hat H_i+t_{ij}^p-a_j\}$, with $\hat H_0=0$ and $\hat H_i=H$ for customers |

## Simple peak profile (default)

$P=3$. Slot 1 and slot 3 are off-peak ($\rho_1=\rho_3=1$). Slot 2 is the peak ($\rho_2=\rho\ge1$, default $1.5$) on $[\theta_1,\theta_2]$. Setting $\rho=1$ (or $P=1$) recovers the static CVRPTW.

## Decision variables

| Symbol | Domain | Meaning |
|---|---|---|
| $x_{ij}$ | $\{0,1\}$ | 1 if some vehicle drives directly from $i$ to $j$ |
| $z_{ij}^p$ | $\{0,1\}$ | 1 if arc $(i,j)$ is used and the vehicle leaves $i$ in slot $p$ |
| $s_i$ | $[a_i,b_i]$, $s_0=0$ | Start-of-service time at $i$ ($s_{n+1}$: return time) |
| $\delta_i$ | $[0,H]$, $\delta_0=0$ | Departure time from $i$ (idling after service is allowed) |
| $u_i$ | $[q_i,Q]$ | Cumulative demand delivered on the route up to and including $i$ |

## Derived quantities

| Symbol | Meaning |
|---|---|
| $Z$ | Total cost, $\sum_{(i,j)\in A}c_{ij}x_{ij}$ |
| route | Path $0\to i_1\to\dots\to i_r\to n+1$ with $r\ge1$ |
| $r_i$ | Ready time at $i$: $s_i+d_i$ |
| $\text{arr}_{ij}(r)$ | Earliest arrival at $j$ when ready to leave $i$ at time $r$ |
| $C^*$, $C_{\text{heur}}$ | Optimal cost, cost found by the heuristic |
| gap | $100\,(C_{\text{heur}}-C^*)/C^*$ |

## Route-evaluation rule (shared by ILP checks and the heuristic)

For an arc $(i,j)$ of a route, with $r_i=s_i+d_i$:

$$\text{arr}_{ij}(r_i)=\min_{p:\ \max(r_i,\theta_{p-1})\le\theta_p}\{\max(r_i,\theta_{p-1})+t_{ij}^p\},\qquad s_j=\max\{a_j,\ \text{arr}_{ij}(r_i)\}.$$

Feasible if every $s_j\le b_j$, $s_{n+1}\le H$, and the load is at most $Q$. Start at $s_0=0$.

## Conventions

* Waiting is allowed (before and after service) and free. Lateness is forbidden. Windows are hard.
* Vehicles leave at time 0 and must return by $H$.
* Travel time is fixed by the slot of departure; speed changes during a trip are ignored.
* Single depot, identical vehicles, one visit per customer, no split delivery.
* The heuristic's solution is a list of routes. Start times are recomputed from the routes by the evaluation rule.

## Open decisions (to settle with the team)

1. Objective: free-flow cost (default) or experienced travel time $\sum t_{ij}^pz_{ij}^p$?
2. Is $m$ a hard bound, or is the number of vehicles minimized first (fixed cost $f$)?
3. Peak parameters: $\theta_1,\theta_2,\rho$, and how they are generated for benchmark instances.
4. Distance rounding, and instance storage format (shared with Section 3).

## Translation from the first version of the code and notebook

| Earlier symbol | Now |
|---|---|
| $K$ (number of vehicles) | $m$ |
| $e_i$, $l_i$ | $a_i$, $b_i$ |
| $s_i$ (service duration) | $d_i$ |
| $T_i$ (start time) | $s_i$ |
| $y_i$ (cumulative load) | $u_i$ |
| $l_0$ (horizon end) | $H$ |
| $x_{ijk}$, $s_{ik}$, $u_{ik}$ (three-index draft) | $x_{ij}$, $s_i$, $u_i$ |

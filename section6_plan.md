# Section 6 "Solution method": proposed plan

*Draft for team discussion, 21 October 2026. To be agreed on before the section is written.*

## 1. Proposed method: hybrid ALNS with local search

The proposed method is **ALNS** (Adaptive Large Neighbourhood Search, Ropke and Pisinger 2006), combined with local search. Each iteration works in four steps:

1. **Destroy:** remove q customers from the current solution.
2. **Repair:** reinsert those customers at the best feasible positions.
3. **Improve:** run a short local search on the result.
4. **Accept or reject:** a **simulated annealing** criterion decides whether the new solution is kept.

The destroy and repair operators are selected adaptively, according to how well each one has performed so far.

The starting solution comes from the **Clarke-Wright savings** algorithm.

### Why this method

- **Time windows and traffic:** repair inserts customers one at a time. Each insertion is checked with the route-evaluation rule of Section 2.7. Peak-hour traffic (slot-dependent travel times) changes only this evaluation, not the algorithm itself. This keeps the method consistent with the model of Section 2.
- **Solution quality:** in the literature, ALNS usually reaches a gap of about 1-3% to the best known solutions on the Solomon and Homberger instances. This leaves a clear margin below the 7-10% threshold.
- **Scalability:** only q customers are removed per iteration, so an iteration costs about O(q·n) instead of O(n²). Granular neighbourhoods (only the 20-40 nearest neighbours of each customer are tested) keep the method usable in Python up to a few thousand customers.
- **Fit with the instructions:** the method covers every item requested:
  - definition of solutions;
  - neighbourhood structure (LNS plus local search);
  - operators (insertion, exchange, ...);
  - guidance mechanisms (adaptive weights and simulated annealing).

### Alternatives considered (one line each in the section)

- **Hybrid Genetic Search (HGS, Vidal 2013):** better quality, but the crossover and penalty management are heavy to implement and to explain.
- **Tabu search:** less efficient on large instances.
- **Ant colony optimisation or a simple genetic algorithm:** larger gaps to the best known solutions.
- **PyVRP library:** used only as a comparison baseline, not as the solver.

## 2. Outline of Section 6

- **6.1 Choice of method and justification**
  - A short paragraph.
  - A comparison table of four rows: ALNS, HGS, tabu search, ACO.
- **6.2 Solution representation and evaluation**
  - A solution is a list of routes.
  - Service start times are recomputed with the rule of Section 2.7.
  - Cost = free-flow travel cost + a penalty for each unserved customer.
- **6.3 Initial solution: Clarke-Wright savings, adapted to time windows**
  - Two routes are merged only if the merged route is feasible: its load fits the capacity, and the rule of Section 2.7 confirms the time windows, the traffic slots and the return before H.
  - Feasibility is checked at every merge, not only at the end. A solution found infeasible at the end could not be repaired.
  - If more than m routes remain, the smallest routes are removed. Their customers are then reinserted by the ALNS repair operators.
- **6.4 Neighbourhoods and operators**
  - **Destroy:** random removal, worst-cost removal, Shaw (related) removal, route removal.
  - **Repair:** greedy insertion, regret-2 and regret-3 insertion.
  - **Local search:** relocate, swap, 2-opt*, or-opt, restricted to the k nearest neighbours.
- **6.5 Guidance mechanisms**
  - **Adaptive operator weights:** roulette-wheel selection with scores σ1, σ2 and σ3, updated after each segment of iterations.
  - **Acceptance:** simulated annealing, defined by an initial temperature and a cooling rate.
  - **Diversification:** noise added to the insertion costs during repair.
- **6.6 Algorithm and stopping criterion**
  - The overall scheme is Clarke-Wright, then the ALNS loop. Each loop iteration runs: destroy → repair → local search → acceptance → weight update.
  - Local search runs **inside** the loop, not only once at the end. It recovers the last few percent of the gap.
  - To save time, local search can be applied only to promising candidates (for example within 1% of the best solution). A full pass is run on the final solution.
  - The method stops at a time limit proportional to n, or after N iterations without improvement.
  - The section ends with pseudocode of 10-15 lines.

**Target length and references:**
- The section has the same length as Section 1 and contains no code.
- It cites only one new reference, Ropke and Pisinger 2006, to be read before it is cited.

## 3. Decisions for the team

1. **Method:** hybrid ALNS (proposed) or HGS?
2. **Feasibility during the search.** Two options:
   - *Proposed, simpler:* time windows and capacity stay hard, and only unserved customers are penalised.
   - *Stronger but more complex:* infeasible solutions are allowed and penalised.
3. **Number of vehicles: is m a hard bound, or is the fleet size minimised first?**
   - Many published best known solutions for the Solomon and Homberger instances minimise the number of vehicles first, then distance.
   - The best known solutions provided through vrplib must be checked. Otherwise the gaps may not be comparable.
   - This is linked to open decision 2 in NOTATION.md.
4. **Objective:** free-flow travel cost (default) or experienced travel time? Comparison with best known solutions requires ρ = 1 and the free-flow objective.
5. **Starting parameter values:**
   - destroy size q (for example 10-40% of n, with an upper cap);
   - initial temperature and cooling rate;
   - number of nearest neighbours k.
   
   They will be tuned in the final deliverable.
6. **Fast evaluation with traffic:** the classic O(1) feasibility checks based on forward time slack (Savelsbergh) do not apply directly when travel times depend on the time slot. The proposed workaround is to recompute only the end of the modified route, and to stop as soon as the start times no longer change.
7. **Writing:** who writes Section 6?

# SupplyMind - Phase-Wise Implementation Prompts

## How To Use This File

This file contains **self-contained prompts** for Claude Code Opus 4.6. Each phase prompt includes full context so two team members can work in parallel:

- **Person A** works on: Phase 2A (Engine) → Phase 3A (Baseline + Docker)
- **Person B** works on: Phase 2B (Tasks + Server) → Phase 3B (Tests + Docs)
- **Both together** for: Phase 1 (Foundation - already done) → Phase 4 (Integration)

Copy-paste the relevant phase prompt into a new Claude Code session. Each prompt is self-contained with all necessary context.

---

## PHASE 1: FOUNDATION (COMPLETED)

Phase 1 creates: CLAUDE.md, models.py, pyproject.toml, openenv.yaml, .gitignore, directory structure.

**Already completed.** See existing files in the repository.

---

## PHASE 2A PROMPT: ENGINE CORE (Person A)

```
You are building the simulation engine for SupplyMind, an OpenEnv-compliant supply chain risk management environment. The project structure and models are already created. You need to implement the engine that powers the simulation.

IMPORTANT: Read CLAUDE.md and models.py first to understand the project rules and data models.

### YOUR TASK
Implement all files in server/engine/ and create all JSON data files in server/data/

### FILES TO CREATE

#### 1. server/engine/graph.py - Supply Chain Graph

Build a SupplyChainGraph class using NetworkX DiGraph. This is the core domain model.

**Node types (5):**
- SUPPLIER: {id, name, tier (1/2/3), lat, lng, country, lead_time_days, annual_spend, single_source: bool, components: list[str], backup_supplier_ids: list[str], is_operational: bool, risk_score: float}
- WAREHOUSE: {id, name, lat, lng, country, inventory_days_cover, capacity_units, current_inventory_units, daily_consumption_rate}
- PORT: {id, name, lat, lng, country, port_type (sea/air/rail), avg_dwell_time_hours, congestion_score: float, is_operational: bool}
- FACTORY: {id, name, lat, lng, country, production_capacity_daily, utilization_pct, is_operational: bool}
- CUSTOMER: {id, name, lat, lng, country, revenue_contribution: float, sla_days: int}

**Edge types (4):**
- SUPPLIES: {component, quantity, lead_time_days, transport_mode, cost_per_unit, dependency_weight: float}
- SHIPS_VIA: {transit_time_days, carrier, is_active: bool}
- STORES_AT: {component, quantity, reorder_point}
- DELIVERS_TO: {lead_time_days, sla_days}

**Required methods:**
- load_from_json(filepath) - Load graph from JSON file
- propagate_disruption(node_id, severity, duration_days) → dict of {node_id: {delay_days, severity, revenue_at_risk, time_to_impact}}. Uses BFS from blueprint Section 2.4. Severity decays through tiers. Inventory buffers absorb delay.
- inventory_cover(node_id, disrupted_supplier_ids) → float days. From blueprint Section 2.4.
- apply_action(action: SupplyMindAction) → ActionResult. Validates action, modifies graph state, returns cost and effect.
  - activate_backup_supplier: Sets backup as active supplier, adds 15-30% cost premium
  - reroute_shipment: Changes SHIPS_VIA edges, may add transit time
  - increase_safety_stock: Increases warehouse inventory, costs units * cost_per_unit * 0.25/365 * days
  - expedite_order: Changes transport_mode, multiplies cost (air=8x, rail=3x, express_sea=2x base)
  - hedge_commodity: Stores hedge, costs 3% of notional as premium
  - issue_supplier_alert: Free, marks supplier as alerted (gives info bonus in reward)
  - do_nothing: No-op
- get_total_revenue_at_risk() → float. Sum of revenue_contribution for all disrupted downstream customers.
- get_health_score() → float 0-100. Composite of: operational nodes %, average inventory cover, average risk score.
- get_sla_compliance() → float 0-1. Fraction of customers whose delivery within SLA.
- get_node_statuses() → list[SupplierStatus]. Build observation data from current graph state.
- find_backup_suppliers(node_id) → list[str]. Return backup_supplier_ids for a node.

#### 2. server/engine/financial.py - Financial Impact Model

Build a FinancialEngine class that tracks all financial state.

**Track:**
- budget_total, budget_remaining
- cumulative_cost_incurred (from actions)
- cumulative_revenue_lost (from disrupted supply paths)
- cumulative_penalty_fees (from SLA violations)
- commodity_prices: dict[str, float] (multipliers, 1.0 = baseline)

**Action cost table:**
| Action | Cost Formula |
|--------|-------------|
| activate_backup_supplier | one-time $50K qualification + ongoing 20% premium on affected spend |
| reroute_shipment | $10K per port change + potential delay cost |
| increase_safety_stock | units * unit_cost * (0.25/365) * additional_days (carrying cost 25%/yr) |
| expedite_order | base_shipping_cost * multiplier (air=8x, rail=3x, express_sea=2x) |
| hedge_commodity | 3% of hedge_amount_usd (option premium) |
| issue_supplier_alert | $0 (free information action) |
| do_nothing | $0 |

**Revenue loss calculation:**
- For each disrupted node, calculate downstream revenue at risk
- Daily revenue loss = annual_revenue_at_risk / 365 * severity
- SLA penalty = $10K per customer per day beyond SLA

**Methods:**
- process_action_cost(action, graph) → float cost
- calculate_daily_revenue_loss(graph) → float
- calculate_sla_penalties(graph) → float
- apply_commodity_price_change(commodity, multiplier)
- get_snapshot() → FinancialSnapshot (from models.py)

#### 3. server/engine/disruptions.py - Disruption Lifecycle Engine

Build a DisruptionEngine class managing disruption lifecycles.

**Disruption lifecycle phases:**
- WARNING: Signal detected, no impact yet. time_to_impact_hours > 0, severity reflects predicted intensity
- ACTIVE: Disruption is happening. Nodes affected, severity at peak
- RECOVERY: Severity decreasing, nodes partially restored
- RESOLVED: Disruption over, nodes fully operational

**Pre-scripted scenario format (JSON):**
```json
{
  "disruptions": [
    {
      "signal_id": "SIG-001",
      "disruption_type": "cyclone",
      "trigger_day": 2,
      "warning_severity": 0.4,
      "warning_confidence": 0.6,
      "peak_severity": 0.9,
      "impact_day": 5,
      "recovery_start_day": 12,
      "resolved_day": 18,
      "affected_region": "Taiwan",
      "affected_node_ids": ["SUP001", "PORT001"],
      "estimated_duration_days": 13,
      "description": "Category 3 typhoon approaching Taiwan manufacturing corridor"
    }
  ]
}
```

**Methods:**
- load_scenarios(filepath) - Load from JSON
- advance_day(current_day) → list[DisruptionSignal]. Returns new/updated signals for this day.
- get_active_signals() → list[DisruptionSignal]
- get_new_signals() → list[DisruptionSignal] (appeared this step only)
- apply_to_graph(graph) - Update graph node operational status based on active disruptions

**Lifecycle rules:**
- Day < trigger_day: No signal
- trigger_day <= Day < impact_day: WARNING phase, severity = warning_severity, confidence increases as impact_day approaches
- impact_day <= Day < recovery_start_day: ACTIVE phase, severity = peak_severity
- recovery_start_day <= Day < resolved_day: RECOVERY phase, severity linearly decreases to 0
- Day >= resolved_day: RESOLVED, signal removed

#### 4. server/engine/rewards.py - Dense Reward Function

Build a RewardCalculator class with 7 components.

```python
def compute_step_reward(self, prev_state, current_state, action, action_result) -> float:
    """Compute reward for current step. Returns float in [-1.0, 1.0]."""

    reward = 0.0

    # 1. REVENUE PRESERVATION (35%) - continuous
    # If revenue-at-risk decreased (agent's action helped), positive reward
    delta_risk = prev_state.revenue_at_risk - current_state.revenue_at_risk
    max_risk = self.initial_total_revenue if self.initial_total_revenue > 0 else 1.0
    revenue_signal = delta_risk / max_risk
    reward += 0.35 * max(-1.0, min(1.0, revenue_signal * 10))  # Scale up for sensitivity

    # 2. PROACTIVE BONUS (15%) - sparse
    # Acting during WARNING phase (before disruption hits) gets bonus
    if action.action_type != "do_nothing" and action_result.success:
        if any(s.lifecycle_phase == "warning" for s in current_state.active_signals
               if action.target_node_id in s.affected_node_ids):
            reward += 0.15

    # 3. COST PENALTY (10%) - continuous
    if action_result.cost > 0:
        cost_ratio = action_result.cost / current_state.budget_total
        reward -= 0.10 * min(1.0, cost_ratio * 5)  # Scale for sensitivity

    # 4. STOCKOUT PENALTY (25%) - event-driven
    # Any customer with inventory_days_cover <= 0 triggers severe penalty
    stockout_nodes = [n for n in current_state.node_statuses
                      if n.node_type == "customer" and n.inventory_days_cover <= 0]
    if stockout_nodes:
        stockout_fraction = len(stockout_nodes) / max(1, current_state.total_customers)
        reward -= 0.25 * stockout_fraction

    # 5. UNNECESSARY ACTION PENALTY (5%) - sparse
    # Action on a node with no active disruption and no warning
    if action.action_type not in ("do_nothing", "issue_supplier_alert"):
        target_affected = any(
            action.target_node_id in s.affected_node_ids
            for s in current_state.active_signals
        )
        if not target_affected and action.target_node_id is not None:
            reward -= 0.05

    # 6. HEALTH MAINTENANCE (5%) - continuous
    health_delta = current_state.health_score - prev_state.health_score
    reward += 0.05 * max(-1.0, min(1.0, health_delta / 20.0))

    # 7. SLA COMPLIANCE (5%) - continuous
    reward += 0.05 * current_state.sla_compliance

    return max(-1.0, min(1.0, reward))
```

#### 5. server/engine/monte_carlo.py - Monte Carlo Simulation

Build a MonteCarloEngine class for probabilistic impact estimation.

**Purpose:** Run N=500 simulations with randomized parameters to estimate loss distribution. Results shown in observation to help agent make informed decisions.

```python
def run_simulation(self, graph, active_disruptions, n_simulations=500):
    """
    Returns {
        "p50_loss": float,
        "p95_loss": float,
        "p99_loss": float,
        "avg_nodes_affected": float,
        "max_delay_days": float
    }
    """
    # For each simulation:
    # 1. Randomize severity using Beta(alpha, beta) where mean = current_severity
    # 2. Randomize duration using lognormal(log(expected), 0.3)
    # 3. Run graph.propagate_disruption() with randomized params
    # 4. Sum revenue_at_risk across all affected nodes
    # Collect results, compute percentiles with numpy
```

#### 6. server/engine/simulation.py - Core Step Loop

Build a SimulationEngine class that ties everything together.

**Constructor:** Takes graph, disruption_schedule, budget, max_steps
**Methods:**
- get_initial_observation() → SupplyMindObservation
- step(action: SupplyMindAction) → SupplyMindObservation

**Step loop:**
1. Validate action (check target_node_id exists, backup exists, budget sufficient)
2. Apply action to graph via graph.apply_action()
3. Process action cost via financial.process_action_cost()
4. Advance simulation day (current_day += 1)
5. Activate new disruptions via disruptions.advance_day()
6. Apply disruptions to graph via disruptions.apply_to_graph()
7. Update node statuses (recalculate operational status, inventory, risk)
8. Deplete inventory for disrupted supply paths
9. Calculate daily revenue loss and SLA penalties
10. Run mini Monte Carlo (N=200 for speed) on current state
11. Compute step reward via rewards.compute_step_reward()
12. Check termination (current_day >= max_steps or all disruptions resolved and day > min_episode_days)
13. Build situation_summary text (natural language for LLM agents)
14. Build and return SupplyMindObservation

**situation_summary generation:**
Write a clear, concise text summary like:
"Day 5/30 | Budget: $4.2M/$5.0M | Health: 72/100
ACTIVE DISRUPTIONS: Typhoon Gaemi hitting Taiwan (severity 0.9). TSMC Fab 14 OFFLINE. Port of Kaohsiung CLOSED.
AT RISK: $12.4M revenue. Warehouse inventory covers 15 more days.
AVAILABLE ACTIONS: Samsung (KR) available as backup supplier. Air freight expedite available ($800K).
LAST ACTION: Issued supplier alert to TSMC — response pending."

#### 7. JSON Data Files

Create 3 supply chain graphs and 3 disruption scenarios:

**server/data/graphs/easy_graph.json** - 12 nodes:
- Suppliers: TSMC (Taiwan, tier 1, single-source, $500M spend), Samsung (Korea, tier 1, backup)
- Ports: Kaohsiung (sea), Busan (sea), Long Beach (sea)
- Factory: Foxconn (Vietnam)
- Warehouses: Ontario CA, Dallas TX
- Customers: Apple US ($200M revenue), Dell US ($150M), HP US ($100M)
- Use REAL coordinates for all nodes

**server/data/graphs/medium_graph.json** - 25 nodes:
- Add: Thailand suppliers (Tier 2), Chinese rare earth supplier, European components
- Add: Rotterdam port, Shanghai port
- Add: Mexico factory
- More complex routing with multiple paths

**server/data/graphs/hard_graph.json** - 40 nodes:
- Full global automotive: Japan (Toyota parts), Germany (Bosch), India (Tata Steel)
- 6 countries, 3 tiers, 8 suppliers, 4 ports, 3 warehouses, 2 factories, 5 customers
- Multiple single-source dependencies

**server/data/disruptions/easy_scenarios.json:**
Single typhoon: warning Day 2, impact Day 5, recovery Day 12, resolved Day 18

**server/data/disruptions/medium_scenarios.json:**
Three concurrent: port strike (Day 7-25), Thailand flood (Day 9-30), sanctions (Day 18 onward)

**server/data/disruptions/hard_scenarios.json:**
Cascade: military exercises (Day 2), shipping restricted (Day 5), blockade (Day 8), TSMC offline (Day 10), Samsung delays (Day 12), commodity spike (Day 15), cyber attack (Day 20), partial reopening (Day 30)

**server/data/commodities/price_data.json:**
```json
{
  "semiconductors": {"baseline_price_per_unit": 45.00, "currency": "USD"},
  "rare_earths": {"baseline_price_per_kg": 280.00, "currency": "USD"},
  "shipping_container_40ft": {"baseline_price": 2500.00, "currency": "USD"},
  "crude_oil_barrel": {"baseline_price": 75.00, "currency": "USD"}
}
```

### IMPORTANT CONSTRAINTS
- All disruption scenarios are PRE-SCRIPTED (deterministic) - no randomness in scenario progression
- Monte Carlo randomness is OK (it's for estimation, not grading)
- Graph uses REAL coordinates for authenticity
- Financial parameters must be realistic (check against blueprint Sections 2.4, 2.6, 4.5)
- Import models from models.py (SupplyMindAction, SupplyMindObservation, etc.)
- Keep imports at top of files, use type hints everywhere
- Write docstrings on all public methods

### TESTING YOUR CODE
After implementing, verify:
1. Graph loads from JSON without error
2. BFS propagation returns expected affected nodes
3. Actions modify graph state correctly
4. Financial calculations are reasonable
5. Reward function returns values in [-1.0, 1.0]
6. Monte Carlo returns percentile estimates
7. Full step loop produces valid SupplyMindObservation
```

---

## PHASE 2B PROMPT: TASKS, GRADERS & SERVER (Person B)

```
You are building the tasks, graders, and server for SupplyMind, an OpenEnv-compliant supply chain risk management environment. The project structure and models are already created. Person A is building the engine in server/engine/ — you should NOT modify those files.

IMPORTANT: Read CLAUDE.md and models.py first to understand the project rules and data models.

### YOUR TASK
Implement: server/tasks/, server/graders/, server/supply_environment.py, server/app.py, client.py

### FILES TO CREATE

#### 1. server/tasks/registry.py - Task Registry

```python
from dataclasses import dataclass

@dataclass
class TaskDefinition:
    task_id: str
    name: str
    difficulty: str  # easy, medium, hard
    description: str
    episode_length: int  # max steps
    budget: float  # USD
    graph_file: str  # path to graph JSON
    disruption_file: str  # path to disruption scenario JSON
    min_episode_days: int  # minimum days before episode can end early

class TaskRegistry:
    _tasks: dict[str, TaskDefinition] = {}

    @classmethod
    def register(cls, task: TaskDefinition):
        cls._tasks[task.task_id] = task

    @classmethod
    def get(cls, task_id: str) -> TaskDefinition:
        if task_id not in cls._tasks:
            raise ValueError(f"Unknown task: {task_id}. Available: {list(cls._tasks.keys())}")
        return cls._tasks[task_id]

    @classmethod
    def list_tasks(cls) -> list[TaskDefinition]:
        return list(cls._tasks.values())

    @classmethod
    def register_all(cls):
        """Register all built-in tasks."""
        from server.tasks.task_easy import register_easy_task
        from server.tasks.task_medium import register_medium_task
        from server.tasks.task_hard import register_hard_task
        register_easy_task()
        register_medium_task()
        register_hard_task()
```

#### 2. server/tasks/task_easy.py - Typhoon Response

```python
def register_easy_task():
    TaskRegistry.register(TaskDefinition(
        task_id="easy_typhoon_response",
        name="Typhoon Response",
        difficulty="easy",
        description=(
            "Manage a semiconductor supply chain through a single typhoon "
            "disruption affecting Taiwan. You have 12 supply chain nodes across "
            "2 tiers. A Category 3 typhoon gives you 72-hour warning before "
            "hitting TSMC (your single-source chip supplier). Activate backup "
            "supplier Samsung and expedite critical orders to prevent stockout."
        ),
        episode_length=30,
        budget=5_000_000,
        graph_file="server/data/graphs/easy_graph.json",
        disruption_file="server/data/disruptions/easy_scenarios.json",
        min_episode_days=20,
    ))
```

#### 3. server/tasks/task_medium.py - Multi-Front Crisis

Register with:
- task_id="medium_multi_front"
- name="Multi-Front Crisis"
- 45 steps, $8M budget, 25 nodes
- Description: Triage 3 concurrent disruptions (port strike, flooding, sanctions)
- min_episode_days=35

#### 4. server/tasks/task_hard.py - Cascading Crisis

Register with:
- task_id="hard_cascading_crisis"
- name="Cascading Crisis"
- 60 steps, $10M budget, 40 nodes
- Description: Navigate cascading geopolitical crisis (Taiwan Strait cascade)
- min_episode_days=45

#### 5. server/graders/grader.py - Programmatic Graders

Build graders that score 0.0-1.0 with multi-component weighted scoring.

CRITICAL REQUIREMENTS:
- Graders MUST be deterministic
- Graders MUST return DIFFERENT scores for different strategies
- Do-nothing agent should score ~0.1-0.2 (NOT zero — some baseline revenue is preserved)
- Optimal agent should score ~0.85-0.95 (NOT perfect 1.0 — that's unrealistic)

```python
class EpisodeGrader:
    def __init__(self, task_id: str):
        self.task_id = task_id
        self.breakdown = {}

    def grade(self, episode_history: list, engine) -> float:
        """Grade a completed episode. Returns score 0.0-1.0."""
        if self.task_id == "easy_typhoon_response":
            return self._grade_easy(episode_history, engine)
        elif self.task_id == "medium_multi_front":
            return self._grade_medium(episode_history, engine)
        elif self.task_id == "hard_cascading_crisis":
            return self._grade_hard(episode_history, engine)
        raise ValueError(f"Unknown task: {self.task_id}")

    def _grade_easy(self, history, engine) -> float:
        # Component 1: Revenue Preserved (40%)
        max_possible_loss = engine.calculate_max_possible_loss()  # what happens with do-nothing
        actual_loss = engine.financial.cumulative_revenue_lost
        revenue_preserved = 1.0 - (actual_loss / max_possible_loss) if max_possible_loss > 0 else 1.0
        revenue_score = max(0.0, min(1.0, revenue_preserved))

        # Component 2: Timeliness (25%)
        # Did agent act BEFORE impact day (day 5)?
        first_meaningful_action_day = None
        for action, obs in history:
            if action.action_type not in ("do_nothing", "issue_supplier_alert"):
                first_meaningful_action_day = obs.current_day
                break
        if first_meaningful_action_day is None:
            timeliness_score = 0.0
        elif first_meaningful_action_day <= 3:  # Acted during warning
            timeliness_score = 1.0
        elif first_meaningful_action_day <= 5:  # Acted at impact
            timeliness_score = 0.6
        elif first_meaningful_action_day <= 8:  # Acted during active
            timeliness_score = 0.3
        else:
            timeliness_score = 0.1

        # Component 3: Cost Efficiency (20%)
        total_cost = engine.financial.cumulative_cost_incurred
        budget = engine.financial.budget_total
        cost_ratio = total_cost / budget if budget > 0 else 0
        # Ideal: spend 10-30% of budget. Penalize both under and overspending
        if cost_ratio < 0.05:  # Barely spent anything (probably did nothing)
            cost_score = 0.2
        elif cost_ratio < 0.30:  # Sweet spot
            cost_score = 1.0
        elif cost_ratio < 0.60:  # Spent a lot but managed
            cost_score = 0.6
        else:  # Overspent
            cost_score = 0.3

        # Component 4: Stockout Prevention (15%)
        any_stockout = engine.any_customer_experienced_stockout()
        stockout_score = 0.0 if any_stockout else 1.0

        # Calculate weighted score
        self.breakdown = {
            "revenue_preserved": {"score": revenue_score, "weight": 0.40},
            "timeliness": {"score": timeliness_score, "weight": 0.25},
            "cost_efficiency": {"score": cost_score, "weight": 0.20},
            "stockout_prevention": {"score": stockout_score, "weight": 0.15},
        }

        final = sum(v["score"] * v["weight"] for v in self.breakdown.values())
        return round(max(0.0, min(1.0, final)), 4)

    def _grade_medium(self, history, engine) -> float:
        # 5 components: financial_impact 30%, triage_quality 25%, budget_utilization 20%,
        # sla_compliance 15%, proactive_score 10%

        # Financial impact (30%)
        max_loss = engine.calculate_max_possible_loss()
        actual_loss = engine.financial.cumulative_revenue_lost + engine.financial.cumulative_penalty_fees
        financial_score = 1.0 - (actual_loss / max_loss) if max_loss > 0 else 1.0

        # Triage quality (25%) - Were highest-impact disruptions addressed first?
        # Check action order vs disruption severity order
        action_targets = []
        for action, obs in history:
            if action.action_type not in ("do_nothing", "issue_supplier_alert") and action.target_node_id:
                action_targets.append(action.target_node_id)
        # Score based on whether agent addressed port strike and flood before sanctions
        triage_score = self._evaluate_triage_order(action_targets, engine)

        # Budget utilization (20%)
        spent = engine.financial.cumulative_cost_incurred
        budget = engine.financial.budget_total
        utilization = spent / budget if budget > 0 else 0
        budget_score = 1.0 if 0.2 <= utilization <= 0.6 else max(0.2, 1.0 - abs(utilization - 0.4))

        # SLA compliance (15%)
        sla_score = engine.graph.get_sla_compliance()

        # Proactive score (10%)
        proactive_actions = sum(1 for a, o in history
                               if a.action_type not in ("do_nothing",) and o.current_day < 7)
        proactive_score = min(1.0, proactive_actions / 3)

        self.breakdown = {
            "financial_impact": {"score": max(0, min(1, financial_score)), "weight": 0.30},
            "triage_quality": {"score": max(0, min(1, triage_score)), "weight": 0.25},
            "budget_utilization": {"score": max(0, min(1, budget_score)), "weight": 0.20},
            "sla_compliance": {"score": max(0, min(1, sla_score)), "weight": 0.15},
            "proactive_score": {"score": max(0, min(1, proactive_score)), "weight": 0.10},
        }

        final = sum(v["score"] * v["weight"] for v in self.breakdown.values())
        return round(max(0.0, min(1.0, final)), 4)

    def _grade_hard(self, history, engine) -> float:
        # 6 components: loss_minimized 25%, cascade_containment 20%,
        # information_efficiency 15%, budget_roi 15%, resilience 15%, customer_impact 10%

        # Loss minimized (25%)
        max_loss = engine.calculate_max_possible_loss()
        actual_total = (engine.financial.cumulative_revenue_lost +
                       engine.financial.cumulative_cost_incurred +
                       engine.financial.cumulative_penalty_fees)
        loss_score = 1.0 - (actual_total / max_loss) if max_loss > 0 else 1.0

        # Cascade containment (20%)
        max_cascade = engine.calculate_max_cascade_nodes()
        actual_cascade = engine.count_nodes_that_went_offline()
        cascade_score = 1.0 - (actual_cascade / max_cascade) if max_cascade > 0 else 1.0

        # Information efficiency (15%)
        alerts_before_actions = sum(1 for a, o in history if a.action_type == "issue_supplier_alert")
        total_actions = sum(1 for a, o in history if a.action_type != "do_nothing")
        info_score = min(1.0, alerts_before_actions / max(1, total_actions) * 3)

        # Budget ROI (15%)
        spent = engine.financial.cumulative_cost_incurred
        saved = max_loss - actual_total if max_loss > actual_total else 0
        roi = saved / spent if spent > 0 else 0
        roi_score = min(1.0, roi / 10)  # ROI of 10x = perfect score

        # Resilience (15%)
        final_health = engine.graph.get_health_score()
        resilience_score = final_health / 100.0

        # Customer impact (10%)
        customer_score = engine.graph.get_sla_compliance()

        self.breakdown = {
            "loss_minimized": {"score": max(0, min(1, loss_score)), "weight": 0.25},
            "cascade_containment": {"score": max(0, min(1, cascade_score)), "weight": 0.20},
            "information_efficiency": {"score": max(0, min(1, info_score)), "weight": 0.15},
            "budget_roi": {"score": max(0, min(1, roi_score)), "weight": 0.15},
            "resilience": {"score": max(0, min(1, resilience_score)), "weight": 0.15},
            "customer_impact": {"score": max(0, min(1, customer_score)), "weight": 0.10},
        }

        final = sum(v["score"] * v["weight"] for v in self.breakdown.values())
        return round(max(0.0, min(1.0, final)), 4)

    def get_breakdown(self) -> dict:
        return self.breakdown
```

#### 6. server/supply_environment.py - Environment Class

```python
from uuid import uuid4
from models import SupplyMindAction, SupplyMindObservation, SupplyMindState
from server.engine.simulation import SimulationEngine
from server.engine.graph import SupplyChainGraph
from server.engine.disruptions import DisruptionEngine
from server.engine.financial import FinancialEngine
from server.engine.rewards import RewardCalculator
from server.engine.monte_carlo import MonteCarloEngine
from server.tasks.registry import TaskRegistry
from server.graders.grader import EpisodeGrader

class SupplyMindEnvironment:
    def __init__(self):
        TaskRegistry.register_all()
        self.engine = None
        self.current_task = None
        self._state = SupplyMindState()
        self._episode_history = []

    def reset(self, task_id: str = "easy_typhoon_response") -> SupplyMindObservation:
        task = TaskRegistry.get(task_id)
        self.engine = SimulationEngine(
            graph_file=task.graph_file,
            disruption_file=task.disruption_file,
            budget=task.budget,
            max_steps=task.episode_length,
            min_episode_days=task.min_episode_days,
        )
        self.current_task = task
        self._state = SupplyMindState(
            episode_id=str(uuid4()),
            task_id=task_id,
            task_name=task.name,
            task_difficulty=task.difficulty,
            total_steps=task.episode_length,
        )
        self._episode_history = []
        return self.engine.get_initial_observation()

    def step(self, action: SupplyMindAction) -> SupplyMindObservation:
        obs = self.engine.step(action)
        self._state.step_count += 1
        self._state.cumulative_reward += obs.reward
        self._state.is_done = obs.done
        self._episode_history.append((action, obs))
        return obs

    @property
    def state(self) -> SupplyMindState:
        return self._state

    def grade(self) -> dict:
        grader = EpisodeGrader(self._state.task_id)
        score = grader.grade(self._episode_history, self.engine)
        return {
            "task_id": self._state.task_id,
            "task_name": self._state.task_name,
            "difficulty": self._state.task_difficulty,
            "score": score,
            "steps_taken": self._state.step_count,
            "cumulative_reward": round(self._state.cumulative_reward, 4),
            "breakdown": grader.get_breakdown(),
        }
```

#### 7. server/app.py - FastAPI Application

```python
from fastapi import FastAPI, HTTPException
from models import SupplyMindAction, SupplyMindObservation, SupplyMindState
from server.supply_environment import SupplyMindEnvironment

app = FastAPI(
    title="SupplyMind",
    description="Supply chain risk management OpenEnv environment",
    version="1.0.0",
)

env = SupplyMindEnvironment()

@app.get("/health")
async def health():
    return {"status": "ok", "environment": "supplymind", "version": "1.0.0"}

@app.post("/reset")
async def reset(task_id: str = "easy_typhoon_response"):
    try:
        obs = env.reset(task_id=task_id)
        return obs.model_dump()
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.post("/step")
async def step(action: SupplyMindAction):
    if env.engine is None:
        raise HTTPException(status_code=400, detail="Call /reset first")
    if env.state.is_done:
        raise HTTPException(status_code=400, detail="Episode is done. Call /reset.")
    obs = env.step(action)
    return obs.model_dump()

@app.get("/state")
async def get_state():
    return env.state.model_dump()

@app.get("/tasks")
async def list_tasks():
    from server.tasks.registry import TaskRegistry
    tasks = TaskRegistry.list_tasks()
    return {
        "tasks": [
            {
                "task_id": t.task_id,
                "name": t.name,
                "difficulty": t.difficulty,
                "description": t.description,
                "episode_length": t.episode_length,
                "budget": t.budget,
            }
            for t in tasks
        ],
        "action_schema": SupplyMindAction.model_json_schema(),
    }

@app.post("/grader")
async def grade():
    if env.engine is None:
        raise HTTPException(status_code=400, detail="No episode to grade. Run an episode first.")
    return env.grade()

@app.post("/baseline")
async def run_baseline():
    try:
        from baseline import run_all_baselines
        results = run_all_baselines(env)
        return results
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Baseline failed: {str(e)}")
```

#### 8. client.py - Environment Client

```python
import httpx
from models import SupplyMindAction, SupplyMindObservation, SupplyMindState

class SupplyMindClient:
    def __init__(self, base_url: str = "http://localhost:8000"):
        self.base_url = base_url.rstrip("/")
        self.client = httpx.Client(timeout=60.0)

    def reset(self, task_id: str = "easy_typhoon_response") -> SupplyMindObservation:
        resp = self.client.post(f"{self.base_url}/reset", params={"task_id": task_id})
        resp.raise_for_status()
        return SupplyMindObservation(**resp.json())

    def step(self, action: SupplyMindAction) -> SupplyMindObservation:
        resp = self.client.post(f"{self.base_url}/step", json=action.model_dump())
        resp.raise_for_status()
        return SupplyMindObservation(**resp.json())

    def state(self) -> SupplyMindState:
        resp = self.client.get(f"{self.base_url}/state")
        resp.raise_for_status()
        return SupplyMindState(**resp.json())

    def tasks(self) -> dict:
        resp = self.client.get(f"{self.base_url}/tasks")
        resp.raise_for_status()
        return resp.json()

    def grade(self) -> dict:
        resp = self.client.post(f"{self.base_url}/grader")
        resp.raise_for_status()
        return resp.json()

    def close(self):
        self.client.close()
```

### IMPORTANT
- Do NOT modify server/engine/ files — Person A is building those
- Import from models.py for all Pydantic models
- The grader MUST return different scores for different strategies — test this
- app.py should be a thin layer — all logic lives in SupplyMindEnvironment
- Handle errors gracefully with proper HTTP status codes
```

---

## PHASE 3A PROMPT: BASELINE & DOCKER (Person A)

```
You are adding the baseline inference script and Docker containerization to SupplyMind. The engine (server/engine/) is already built by you in Phase 2A. The server (server/app.py) is built by Person B.

IMPORTANT: Read CLAUDE.md first. The baseline script is a MANDATORY competition requirement.

### YOUR TASK
Create: baseline.py, server/Dockerfile, server/requirements.txt

#### 1. baseline.py - OpenAI API Baseline Inference

REQUIREMENTS (from competition rules):
- Uses OpenAI API client (NOT Anthropic/Claude)
- Reads OPENAI_API_KEY from environment variable
- Produces reproducible baseline scores on ALL 3 tasks
- Temperature=0.1 for reproducibility

```python
import os
import json
from openai import OpenAI
from models import SupplyMindAction, SupplyMindObservation

client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

SYSTEM_PROMPT = """You are an expert supply chain risk manager. You manage a global supply chain and must respond to disruptions to minimize financial impact.

OBSERVATION FORMAT:
You receive a JSON observation with:
- current_day / days_remaining: Episode progress
- active_signals: List of disruption signals with severity, type, affected nodes
- new_signals: Signals that appeared this step
- node_statuses: Status of all supply chain nodes (operational, risk, inventory)
- financials: Budget remaining, revenue at risk, costs, Monte Carlo projections
- situation_summary: Natural language summary

ACTION FORMAT:
Respond with a JSON object:
{
    "action_type": "one of: do_nothing, activate_backup_supplier, reroute_shipment, increase_safety_stock, expedite_order, hedge_commodity, issue_supplier_alert",
    "target_node_id": "node ID (if applicable)",
    "backup_supplier_id": "backup ID (for activate_backup_supplier)",
    "reroute_via": ["port_ids"] (for reroute_shipment),
    "additional_stock_days": 30 (for increase_safety_stock, 1-90),
    "expedite_mode": "air|rail|express_sea" (for expedite_order),
    "commodity": "commodity_name" (for hedge_commodity),
    "hedge_amount_usd": 100000 (for hedge_commodity)
}

STRATEGY:
1. On warning signals (lifecycle_phase="warning"): Act proactively! Activate backups and expedite BEFORE the disruption hits.
2. Prioritize by revenue_at_risk: Address highest-impact disruptions first.
3. Budget awareness: Don't overspend. Prefer cheaper actions (issue_supplier_alert is free).
4. Prevent stockouts: If inventory_days_cover is low, increase_safety_stock immediately.
5. If no active disruptions and no warnings: do_nothing.
"""

def select_action(observation: SupplyMindObservation) -> SupplyMindAction:
    """Use GPT-4o to select an action."""
    obs_dict = observation.model_dump()
    # Remove large nested structures to fit context
    obs_summary = {
        "current_day": obs_dict["current_day"],
        "days_remaining": obs_dict["days_remaining"],
        "situation_summary": obs_dict["situation_summary"],
        "active_signals": obs_dict["active_signals"],
        "new_signals": obs_dict["new_signals"],
        "financials": obs_dict["financials"],
        "node_statuses": [
            {k: v for k, v in n.items()
             if k in ("node_id", "name", "node_type", "is_operational",
                       "current_risk_score", "inventory_days_cover",
                       "has_backup", "backup_supplier_ids", "active_disruption_ids")}
            for n in obs_dict["node_statuses"]
        ],
    }

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": json.dumps(obs_summary, indent=2, default=str)},
        ],
        response_format={"type": "json_object"},
        temperature=0.1,
        max_tokens=500,
    )

    action_dict = json.loads(response.choices[0].message.content)
    # Filter to valid fields only
    valid_fields = set(SupplyMindAction.model_fields.keys())
    filtered = {k: v for k, v in action_dict.items() if k in valid_fields and v is not None}
    return SupplyMindAction(**filtered)

def run_all_baselines(env) -> dict:
    """Run baseline on all 3 tasks. Returns scores dict."""
    results = {}
    task_ids = ["easy_typhoon_response", "medium_multi_front", "hard_cascading_crisis"]

    for task_id in task_ids:
        print(f"\n{'='*50}")
        print(f"Running baseline: {task_id}")
        print(f"{'='*50}")

        obs = env.reset(task_id=task_id)
        step_count = 0

        while not obs.done:
            try:
                action = select_action(obs)
            except Exception as e:
                print(f"  Error selecting action: {e}. Using do_nothing.")
                action = SupplyMindAction(action_type="do_nothing")

            obs = env.step(action)
            step_count += 1
            print(f"  Day {obs.current_day}: {action.action_type}"
                  f" | Reward: {obs.reward:.3f}"
                  f" | Health: {obs.financials.supply_chain_health_score:.0f}")

        grade_result = env.grade()
        results[task_id] = grade_result
        print(f"\nFinal score: {grade_result['score']}")
        print(f"Breakdown: {json.dumps(grade_result['breakdown'], indent=2)}")

    return results

if __name__ == "__main__":
    from server.supply_environment import SupplyMindEnvironment
    env = SupplyMindEnvironment()
    results = run_all_baselines(env)
    print("\n" + "="*50)
    print("BASELINE RESULTS SUMMARY")
    print("="*50)
    for task_id, result in results.items():
        print(f"  {task_id}: {result['score']:.4f}")
```

#### 2. server/Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for layer caching
COPY server/requirements.txt ./server/requirements.txt
RUN pip install --no-cache-dir -r server/requirements.txt

# Copy application code
COPY models.py .
COPY client.py .
COPY baseline.py .
COPY openenv.yaml .
COPY server/ ./server/

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Run server
CMD ["uvicorn", "server.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### 3. server/requirements.txt

```
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
pydantic>=2.0.0
networkx>=3.2
numpy>=1.24.0
httpx>=0.25.0
openai>=1.0.0
```

### TESTING
1. docker build -t supplymind -f server/Dockerfile .
2. docker run -p 8000:8000 supplymind
3. curl http://localhost:8000/health
4. curl -X POST http://localhost:8000/reset?task_id=easy_typhoon_response
5. OPENAI_API_KEY=sk-... python baseline.py
```

---

## PHASE 3B PROMPT: TESTING & DOCUMENTATION (Person B)

```
You are writing tests and documentation for SupplyMind. All code is already built.

IMPORTANT: Read CLAUDE.md for all competition documentation requirements.

### YOUR TASK
Create: tests/*.py and README.md

#### 1. tests/test_models.py
- Test all Pydantic models serialize/deserialize correctly
- Test field validation (severity must be 0-1, action_type must be valid enum, etc.)
- Test edge cases (empty lists, None optional fields)

#### 2. tests/test_engine.py
- Test graph loads from all 3 JSON files
- Test BFS propagation returns expected affected nodes
- Test inventory_cover calculation
- Test financial calculations
- Test Monte Carlo returns reasonable percentiles
- Test reward function returns values in [-1.0, 1.0]

#### 3. tests/test_tasks.py
- Test all 3 tasks register correctly
- Test TaskRegistry.list_tasks() returns 3 tasks
- Test each task's graph and disruption files exist and load

#### 4. tests/test_graders.py (MOST CRITICAL)
- Test that do-nothing agent scores LOW (~0.1-0.2)
- Test that a smart hand-crafted strategy scores HIGH (~0.7-0.9)
- Test that DIFFERENT strategies produce DIFFERENT scores
- Test that scores are always in [0.0, 1.0]
- Test grader breakdown has expected components

#### 5. tests/test_server.py
- Integration test using FastAPI TestClient
- Test GET /health returns 200
- Test POST /reset returns valid observation JSON
- Test POST /step returns observation with reward
- Test GET /state returns valid state
- Test GET /tasks returns 3 tasks
- Test POST /grader returns score in [0, 1]
- Test POST /step after episode done returns 400

#### 6. README.md

Must include ALL of the following (competition requirement):

# SupplyMind

## Overview
Supply chain risk management OpenEnv environment. [Explain what it does, why it matters]

## Motivation
$184B disruption costs. Why this is a real-world task, not a toy.

## Action Space
Table of all 7 action types with parameters and costs.

## Observation Space
Description of all observation fields: signals, node statuses, financials, summary.

## Tasks

### Easy: Typhoon Response
[Description, nodes, steps, budget, what makes it easy]

### Medium: Multi-Front Crisis
[Description, what makes it harder]

### Hard: Cascading Crisis
[Description, what makes it genuinely challenging]

## Difficulty Progression
How difficulty increases across tasks.

## Setup & Installation

### Local
pip install -e ".[dev]"
uvicorn server.app:app

### Docker
docker build -t supplymind -f server/Dockerfile .
docker run -p 8000:8000 supplymind

### Environment Variables
OPENAI_API_KEY=sk-... (for baseline only)

## Usage

### Reset
POST /reset?task_id=easy_typhoon_response

### Step
POST /step with SupplyMindAction JSON body

### State
GET /state

### Tasks
GET /tasks

### Grader
POST /grader

### Baseline
POST /baseline (requires OPENAI_API_KEY)

## Baseline Scores
| Task | Difficulty | Score | Steps |
|------|-----------|-------|-------|
| Typhoon Response | Easy | ~0.72 | 30 |
| Multi-Front Crisis | Medium | ~0.55 | 45 |
| Cascading Crisis | Hard | ~0.38 | 60 |

## License
MIT
```

---

## PHASE 4 PROMPT: INTEGRATION & DEPLOYMENT (Both people)

```
SupplyMind is fully built. You need to integrate Person A and Person B's code, fix any issues, and deploy.

### INTEGRATION STEPS

1. Ensure server/engine/ imports work correctly from server/supply_environment.py
2. Run all tests: pytest tests/ -v
3. Fix any import errors or interface mismatches
4. Run a full episode manually to verify:
   - python -c "from server.supply_environment import SupplyMindEnvironment; env = SupplyMindEnvironment(); obs = env.reset(); print(obs.situation_summary)"

### DOCKER VERIFICATION
1. docker build -t supplymind -f server/Dockerfile .
2. docker run -p 8000:8000 supplymind
3. Test all endpoints:
   - curl http://localhost:8000/health
   - curl -X POST "http://localhost:8000/reset?task_id=easy_typhoon_response"
   - curl -X POST http://localhost:8000/step -H "Content-Type: application/json" -d '{"action_type": "do_nothing"}'
   - curl http://localhost:8000/state
   - curl http://localhost:8000/tasks
   - curl -X POST http://localhost:8000/grader

### BASELINE RUN
OPENAI_API_KEY=sk-... python baseline.py
Record the scores and update README.md

### HUGGING FACE SPACES DEPLOYMENT
1. Create HF Space (Docker type)
2. Push code to Space repo
3. Verify Space builds and deploys
4. Test: curl https://<username>-supplymind.hf.space/health
5. Test: curl -X POST https://<username>-supplymind.hf.space/reset?task_id=easy_typhoon_response

### PRE-SUBMISSION CHECKLIST
Run through EVERY item in CLAUDE.md pre-submission checklist.
Fix any failures before submitting.
```

---

## MERGE STRATEGY

When merging Person A and Person B's code:

1. **No conflicts expected**: Person A writes server/engine/ files. Person B writes server/tasks/, server/graders/, server/app.py, server/supply_environment.py. No overlapping files.

2. **Interface contract**: Both depend on models.py (created in Phase 1). As long as both follow the model definitions, code merges cleanly.

3. **Merge order**: Pull Person A's engine code first, then Person B's server code (since server imports from engine).

4. **Integration test**: After merge, run `pytest tests/ -v` to verify everything works together.

# MFQ E‑Commerce System Test Plan ‑ Test Strategy Note
## 1. Test Objectives
This test adopts the MFQ (Models‑Functional‑Quality) framework to conduct systematic testing for core e‑commerce transaction links. It focuses on ensuring data consistency of orders and inventory across systems (Mall‑WMS/SC2P), covering positive business flows, boundary exceptions, asynchronous message interactions, reliability and risk fallback capabilities. It proactively identifies online risks such as capital loss, overselling and status disorder, so as to guarantee controllable quality for version releases.

## 2. Priority Definition
- **P0 (Highest Priority, must be fully verified and passed)**
Core business workflows, inventory synchronization consistency, message idempotency & out‑of‑sequence handling, exception compensation strategies, transaction amount calculation, release entry/exit criteria.
These are high‑risk online domains directly related to capital loss, overselling and frozen orders. 100% of test cases shall be executed before release, and blocking defects are not allowed to remain.

- **P1 (Important Priority; full coverage with sufficient resources; key sampling under resource constraints)**
Partial order split & shipment, partial return, API parameter validation, performance metrics, observable logs & monitoring alerts, security authentication & permission, failure drill and data repair procedures.
These ensure system robustness. Non‑blocking defects may remain with risk review and documentation required.

- **P2 (Low Priority; risk assessment only, full test cases can be omitted)**
Minor UI interactions, extremely low‑frequency edge combined scenarios, non‑core background auxiliary functions. When time is limited, only risk analysis is required instead of complete test case design.

## 3. Key Trade‑offs
### 3.1 Data Consistency Trade‑off
Distributed lock plus database row‑level lock is applied for strong consistency during order placement pre‑occupation to prioritize overselling prevention. Final consistency is adopted for asynchronous callbacks between Mall and WMS/SC2P as well as return‑to‑warehouse scenarios.

> Rationale: Real‑time strong consistency cannot be achieved since WMS is an external warehouse system. Scheduled reconciliation Jobs and compensation tasks provide fallback remediation. Partial real‑time consistency is sacrificed for overall cross‑system availability and throughput performance.

### 3.2 Message Link Strategy
MQ messages serve as the primary communication channel with HTTP callbacks as fallback supplements. All business messages carry a globally unique serial number for idempotency. The order state machine intercepts out‑of‑sequence, delayed and invalid callback messages.

> Rationale: Asynchronous messages introduce risks including message loss, duplication and out‑of‑sequence delivery, increasing test complexity. However, they decouple the Mall from WMS failures and prevent external warehouse failures from crashing the main e‑commerce site.

### 3.3 High‑Concurrency Seckill Test Strategy
Testing prioritizes inventory locking and anti‑overselling logic. **Data accuracy takes precedence over ultimate concurrent throughput**. Pressure testing verifies predefined concurrency baselines; blind pursuit of unlimited TPS is avoided. Seckill timeout inventory release logic requires intensive validation to prevent under‑selling or overselling.

### 3.4 Boundaries of Automated Compensation
Automated compensation Jobs handle regular exceptions: payment timeout pre‑occupation release, frozen order inspection, minor inventory discrepancies. Large inventory discrepancies and suspicious abnormal orders are prohibited from automatic system remediation and must be escalated to manual review tickets.

> Rationale: Automated compensation reduces operation manpower, yet incorrect auto‑remediation may amplify capital loss risks. Manual approval gates are added for high‑risk data modifications.

### 3.5 Performance Metric Positioning
Documented performance metrics for concurrency, latency and throughput are **recommended baselines** and can be reviewed and adjusted against actual business traffic. Testing focuses primarily on business correctness. Non‑compliant performance without functional defects triggers dedicated performance optimization rather than direct release blocking, unless performance issues induce business errors.

## 4. Core Risk Domains & Test Focus
1. **Cross‑system Inventory Synchronization Risk**: Asynchronous interactions between Mall and WMS/SC2P may trigger message loss, duplication, out‑of‑sequence delivery and callback timeout, leading to inventory discrepancy, overselling and frozen orders. F‑layer inventory synchronization special tests, M‑layer consistency models, Q‑layer reconciliation & compensation and alerts are all designed for this risk.
2. **Order State Transition Risk**: Illegal state transitions, reverse return workflows, combined scenarios of order split / partial shipment may cause mismatches between status and inventory, constrained by the order state machine model.
3. **Transaction Amount Risk**: Discount allocation, partial refund, price snapshot and amount precision issues may result in capital loss. Intensive validation covers amount models and refund logic.
4. **External Dependency Failure Risk**: Unavailable WMS, MQ jitter, lost payment notifications. Verify circuit breaking, degradation, failure drill and data repair procedures.

## 5. Test Scope Trimming Principles
1. When test cycles are compressed, all P0 test cases shall be executed first; high‑risk subsets are sampled for P1 scenarios; P2 scope is directly trimmed.
2. When the real third‑party WMS/SC2P external system is unavailable, simulated Mock services are used for interaction scenario testing.
3. If environment resources are insufficient for failure drills and full‑scale heavy pressure testing, replace them with solution review plus small‑scale pressure testing. Online gray‑scale release and monitoring act as fallback safeguards.

## 6. Version Quality Control
Strictly follow the defined **test entry criteria and release exit criteria**. Formal system testing shall not start without satisfying entry criteria; release is forbidden if exit criteria are not fully met. All remaining defects must go through risk review with risks and mitigation measures documented.

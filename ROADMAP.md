## Roadmap

### Phase 1: Q1 2026 – Core Integration & MVP Development

Deploy a Claude-based agent in a Kubernetes cluster (e.g., via the open-source [kagent controller](https://www.cloudnativedeepdive.com/kagent-claude-k8s-your-private-agentic-troubleshooter/) or a custom operator). This provides a persistent "SRE bot" service with secure cluster API access via RBAC.

Install or develop core Claude Skills that equip the agent with Kubernetes domain knowledge. For example, a Kubernetes manifests skill helps Claude write and validate YAML configurations for Pods, Deployments, Services, etc., following best practices. A cluster operations skill might focus on running `kubectl` commands, checking pod status, and streaming logs. Leverage community-contributed skills like the [Kubernetes Manager](https://mcpmarket.com/tools/skills/kubernetes-manager) skill or reference [example skill structures](https://github.com/sairin1202/claude-skill-factory/blob/627c139ca3b488d83a94b09d49f9a3bf155fc7ce/skills/thebushidocollective-han-jutsu-jutsu-kubernetes-skills-kubernetes-manifests-skill-md.json#L6-L14).

Use the Claude agent in an interactive ChatOps mode (e.g., Slack or CLI integration). Start in a read-only advisory capacity: maintainers can ask "Show me the logs of Pod X" or "Generate a Deployment for image Y with 3 replicas", and Claude (via its skills) will respond with logs or ready-to-apply YAML. All actions are presented to the user for review rather than auto-executed.

**Initial Focus – Problem Detection & YAML Automation:**
- **Resource Inspection**: retrieving cluster state (pods, nodes, events) on request
- **Manifest Generation**: creating/editing Kubernetes manifests on demand, using built-in templates from skills (automatically adding resource limits, probes, and labels in Deployment specs)
- **Basic Troubleshooting Guidance**: analyzing common issues (CrashLoopBackOff, pod not scheduled, etc.) and suggesting fixes

**Outcome**: A functional "Kubernetes assistant" in the cluster that can offload manual YAML writing and surface cluster info quickly, establishing the foundation for more advanced, autonomous operations in subsequent phases.

---

### Phase 2: Q2 2026 – Integrating Advanced AI Infrastructure Features

With core functions in place, expand the agent’s skills to handle advanced Kubernetes features that are critical for AI/ML workloads. This phase is about **deep integration with scheduling, networking, and device management primitives**, and moving from “assistant” to **domain-aware AI Infra co-pilot**.

---

#### 1. Gang Scheduling & Kueue Integration

Develop a dedicated **Kueue Skill** that understands batch semantics, quotas, and gang scheduling constraints.

##### 1.1 Manage Queues and Workloads
- Create, update, and inspect:
  - `ClusterQueue`
  - `LocalQueue`
  - `Workload`
- Capabilities:
  - Allocate or rebalance resources for AI training/inference jobs.
  - Detect workloads stuck in `Pending` / `Admitted=false` and explain **why** (quota, flavor mismatch, DRA not satisfied).
  - Suggest actionable remediations (increase quota, change flavor, move namespace to another queue).

##### 1.2 Optimize Gang Scheduling
- Validate gang semantics for distributed AI jobs:
  - Ensure `minAvailable` matches world size (e.g., TP/DP ranks).
  - Check pod set homogeneity (requests, tolerations, topology).
- Explain scheduling outcomes in **human terms**:
  - “This job cannot start because only 3/4 GPUs are available in the same flavor.”
- Goal: bridge the gap between **scheduler internals** and **user mental model**, especially for GPU-heavy jobs.

##### 1.3 Kueue + DRA Coordination
As Kueue adopts **Dynamic Resource Allocation (DRA)**:
- Teach the agent to reason about:
  - `ResourceClaim`
  - `ResourceClaimTemplate`
  - `DeviceClass`
- Example capability:
  - For a job requiring 4 GPUs:
    - Verify each Pod references a `ResourceClaimTemplate`.
    - Confirm Kueue will only admit the workload once **all claims can be satisfied**.
- Detect mismatches early:
  - Job requests GPUs via `limits` but cluster expects DRA → flag configuration error.

---

#### 2. Gateway API & GAIE for LLM Inference

Incorporate skills around **Gateway API** and **GAIE (Gateway API Inference Extension)** to support production-grade LLM serving.

##### 2.1 Inference Gateway Management (GAIE)
- Assist with installation and configuration of GAIE implementations (e.g., `kgateway`).
- Generate GAIE-specific policies:
  - `InferenceExtension`
  - EPP (Extended Processing Policies)
- Advanced routing scenarios:
  - Session-affinity routing for KV-cache reuse.
  - Route-by-model-version or tenant.
- Example:
  > “Route requests with the same conversation ID to the same backend pod.”

##### 2.2 Standard Gateway API Operations
- Automate creation and validation of:
  - `Gateway`
  - `HTTPRoute`
- Natural-language driven workflows:
  - “Expose this model on `/api/v1/chat` with TLS and rate limiting.”
- Enforce best practices:
  - TLS, timeouts, retries, and clear separation between infra and app teams.

##### 2.3 Traffic Analysis and Scaling
- Consume Gateway metrics (latency, QPS, error rate).
- Capabilities:
  - Detect inference latency spikes.
  - Recommend scaling model replicas.
  - Suggest routing weight adjustments.
- This effectively becomes a **smart L7 + AI-aware load-balancing advisor**.

---

#### 3. Dynamic Resource Allocation (DRA) & Device Plugins

Enable first-class support for **device-centric scheduling**, beyond traditional `resources.limits`.

##### 3.1 ResourceClaim Orchestration
- Generate:
  - `ResourceClaim`
  - `ResourceClaimTemplate`
- Integrate directly into:
  - `Job`
  - `Pod`
  - `LeaderWorkerSet` / distributed workloads
- Example prompt:
  > “Schedule this job with 2 GPUs and 1 InfiniBand device.”

##### 3.2 Monitoring and Sharing
- Understand advanced DRA concepts:
  - Device sharing
  - Fractional GPUs
  - Allocation policies (e.g., `FirstAvailable`)
- Capabilities:
  - Query DRA allocation state.
  - Advise on optimal placement strategies.
  - Verify that **all Pods in a gang have acquired claims** before execution (Kueue + DRA synergy).

##### 3.3 Plugin Awareness
- Detect whether the cluster supports DRA end-to-end.
- If not:
  - Recommend installing appropriate drivers (e.g., NVIDIA DRA GPU driver).
  - Explain limitations of legacy device plugins.

---

#### 4. Security and Policy Automation

As autonomy increases, guardrails become mandatory.

- Manage and reason about:
  - RBAC for multi-tenant AI platforms.
  - NetworkPolicies for isolation between workloads.
- Skill-level enforcement:
  - Ensure generated Pod specs include:
    - `securityContext`
    - Approved `tolerations`
    - Namespace-scoped access only.
- Align with organizational and platform policies by default.

---

#### 5. Semi-Autonomous Execution Model

During Phase 2, selected skills may operate in **semi-autonomous mode**.

- Examples:
  - Auto-create Kueue `Workload` objects.
  - Update Gateway routing rules.
- Safety model:
  - Human-in-the-loop for destructive actions.
  - Dry-run + explain-before-apply by default.
- Testing:
  - Each sub-skill must be validated in isolation.
  - Simulate:
    - Distributed training jobs.
    - Partial resource availability.
    - DRA contention scenarios.

---

#### Phase 2 Outcome

By the end of **Q2 2026**, the Claude agent evolves into a **sophisticated AI Infrastructure Assistant** that can:

- Orchestrate complex AI workflows.
- Reason about gang scheduling, inference routing, and accelerator allocation.
- Interact with Kubernetes using **domain-native abstractions**, not just YAML.
- Operate via natural language or proactive automation.

This phase establishes the technical and conceptual foundation required for a **stable, production-ready system** in Phase 3.

---

### Phase 3: Q3 2026 – Stabilization, Architecture Refinement & Production Readiness

#### 1. Robust Architecture Patterns

##### 1.1 In-Cluster Agent Service
- Run the agent inside the cluster for low-latency API access.
- Requirements:
  - High availability (replicas + leader election if needed).
  - Internal API for UIs, CLIs, or ChatOps tools.

##### 1.2 MCP and Connectors
- Optionally adopt:
  - Model Context Protocol (MCP)
  - Agent Gateway pattern
- Benefits:
  - Clear separation between Claude runtime and cluster access.
  - Easier compliance for regulated or hybrid environments.
- Choose based on:
  - On-prem vs cloud constraints.
  - Security posture.

##### 1.3 Multi-Agent Design
- Split responsibilities:
  - Scheduling Agent (Kueue, DRA)
  - Networking Agent (Gateway API, GAIE)
  - Observability Agent (metrics, alerts)
- Coordinate via a central orchestrator.
- Improves:
  - Reliability
  - Context isolation
  - Debuggability

---

#### 2. Hardening and Safety

##### 2.1 Permission Scoping
- Principle of least privilege:
  - Read cluster-wide.
  - Write only to approved namespaces and resource types.
- High-risk actions require confirmation.

##### 2.2 Auditability
- Log:
  - Actions taken
  - Decisions made
  - Signals used (metrics, events)
- Example:
  > “Scaled Deployment X from 3 → 5 due to sustained P95 latency.”

##### 2.3 Validation & Testing
- Non-production clusters for:
  - Gang scheduling failures.
  - DRA scarcity scenarios.
  - Inference rollout edge cases.
- Validate that recommendations **actually resolve issues**.

---

#### 3. Performance & Cost Optimization

- Use summarization and caching:
  - Avoid re-fetching full cluster state.
- Tune autonomous polling frequency.
- Monitor:
  - Anthropic API usage (cloud)
  - Resource usage (on-prem models)
- Ensure cost aligns with operational value.

---

#### 4. Documentation & Training

- Produce:
  - Skill catalog.
  - Supported prompts and workflows.
  - Safety and operational guidelines.
- Document extension patterns:
  - How to add support for new CRDs.
- Position `kubernetes-skills` as the **community skill hub**.

---

#### 5. Use-Case Driven Refinement

- Incorporate real-world feedback:
  - Add specialized skills (e.g., GPU memory pressure diagnostics).
  - Improve handling of AI-specific edge cases:
    - Admitted-but-never-started workloads.
    - Inference service rolling upgrades.

---

#### Phase 3 Outcome

By **Q3 2026**, the Claude–Kubernetes integration reaches a **stable, production-ready release**:

- A reliable co-pilot for cluster maintainers.
- Supports proactive optimization and reactive troubleshooting.
- Enforces Kubernetes and AI Infra best practices by default.
- Advanced features—gang scheduling, inference routing, dynamic hardware allocation—are fully operational.

The result is an AI-native Kubernetes platform that **approaches self-management**, freeing experts to focus on higher-level system design and innovation.



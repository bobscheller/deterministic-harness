# Technical Brief: The Deterministic Harness Architecture
## Orchestrating Agentic Workflows at Global Scale

## Table of Contents
1. Executive Summary
2. The Deterministic Harness Architecture Overview
    1. The Network Layer (Foundation)
    2. The Model Output Layer (Intelligence)
    3. The Deterministic Harness (Execution)
    4. Observability: The Utility Monitor
3. Details for Building the Deterministic Harness
4. Failure Mode Matrix Integration
5. Threat Matrix Integration
6. Status and Validation
7. Conclusion



## Executive Summary
The "Deterministic Harness Architecture" is a multi-layered architectural pattern designed to solve the critical challenge of deploying powerful, autonomous AI agents in high-stakes or regulated environments. It ensures that while the AI can learn and adapt (agility), its behavior remains predictable, controllable, and legally compliant (safety). The pattern is presented here as validated on a single cyber-physical case (CNC thermal monitoring); the layering itself generalizes to other high-stakes domains such as autonomous vehicles, medical diagnosis, and financial trading, but the physical-grounding mechanisms it relies on — the Digital Twin and cross-modal sensor fusion — do not transfer directly. Each new domain must supply its own ground-truth substitute, which is part of the per-domain engineering rather than a property the architecture provides for free. The network layer is responsible for isolating the AI from the outside world (zero trust), the model output layer is responsible for the AI's intelligence (probabilistic), and the deterministic harness is responsible for the AI's behavior (deterministic). 

### Relationship to Prior Work

The core pattern of this architecture — a simple, trusted, deterministic supervisor that monitors and, when necessary, overrides a more capable but less trustworthy controller — is not new, and this document does not claim it to be. It descends from the **Simplex architecture** (Sha, 2001), from **run-time assurance (RTA)** as codified for safety-critical aviation in **ASTM F3269**, and from **shielding** in reinforcement learning (Alshiekh et al., 2018). The harness's output-validation mechanism is likewise an instance of **policy-as-code** (e.g., Open Policy Agent) and of the constrained-output approach embodied by tools such as NVIDIA NeMo Guardrails. Readers familiar with these will recognize the Deterministic Harness as their application to LLM- and SLM-driven agents.

What this architecture contributes is not the layering, but two things the classical formulations do not address. First, classical Simplex and RTA assume both a *formally verified* safety controller and a *computable* safe-recovery region — assumptions that hold for linear and physical control systems but break down when the high-performance controller is an LLM reasoning over an open-world domain, where neither the safe envelope nor the recovery action can be fully verified in advance. The harness is therefore positioned as a mechanism that *bounds* catastrophic failure rather than one that *guarantees* safety, and the Failure Mode and Threat Matrices exist precisely to make the residual coverage gaps explicit. Second, the **Utility Monitor** treats over-restriction — a supervisor so conservative that it destroys the system's commercial value — as a first-class, measured failure mode (Utility Decay). The classical safety-supervisor literature optimizes for safety alone and has no equivalent concept; instrumenting the safety-versus-value tradeoff, and the organizational drift it induces, is where this work aims to add something new.

### Definitions
Reality Gap -The measurable divergence between the state of a physical production environment and its Digital Twin simulation. When telemetry from the real system (vibration, throughput, power draw, etc.) diverges from the twin beyond a defined statistical threshold (e.g., 2-sigma), the Utility Monitor flags a Reality Gap and pauses the agentic loop for recalibration. Reality Gap monitoring is a primary function of the Utility Monitor layer.

Compliance Hot-Patching - The capability of the Deterministic Harness to absorb an immediate regulatory or legal change without requiring a simultaneous model retrain. When a new legal parameter is issued, it is injected directly into the harness as a new validation rule, acting as a real-time compliance filter that blocks non-compliant outputs. The background model is then queued for a formal clean-room retrain asynchronously. This decouples legal agility from model release cycles.

Golden Rule Set - The core, non-negotiable set of safety and business rules encoded in the Deterministic Harness that must never be violated under any operational condition. These rules define the hard boundaries of agent behavior. The Golden Rule Set is version-controlled, auditable, and serves as the "immutable safety floor" — the set of constraints that cannot be relaxed through normal operational tuning or organizational pressure.

Utility Decay - A measurable degradation in the commercial value produced by the AI system, caused by the Deterministic Harness becoming so restrictive that it rejects a majority of high-value agentic actions. Utility Decay is tracked by the Utility Monitor as the delta between the agent's proposed optimizations and the harness's allowed actions. When this delta exceeds a defined threshold, the monitor triggers a Re-Architecture Alert, signaling that the safety or legal constraints need review to restore the system's return on investment.

Sovereign Shard - A Trusted Execution Environment (TEE)-backed execution boundary that hosts an agent or cluster of related agents, along with their model weights, code, and operational data, within a single jurisdictional perimeter. Sovereign Shards are the primary mechanism of the Network Layer, ensuring that data, inference, and learning loops remain within the regulatory regime (e.g., GDPR, CCPA) applicable to the deployment region. Nothing leaves the shard without explicit policy approval.

Digital Twin - A high-fidelity software simulation of a physical system — such as a manufacturing floor, autonomous vehicle, or field deployment — that mirrors the real environment's state, behavior, and constraints in real time. In the Deterministic Harness Architecture, the Digital Twin serves two distinct functions: it is the validation sandbox in which harness rules and agentic behaviors are tested before deployment to the physical system, and it is the continuous parity reference against which the Utility Monitor reconciles live telemetry. When sensor data from the physical environment diverges from the twin's predicted state beyond a defined threshold, the monitor flags a Reality Gap and initiates recalibration. The Digital Twin is noted as a significant engineering undertaking in its own right — custom-built per deployment domain — and is a prerequisite for meaningful Reality Gap monitoring and safe harness rule development. It does not replace physical testing but provides a deterministic, repeatable environment for exercising edge cases that would be unsafe or impractical to reproduce on the production floor.

### References

NVIDIA OpenShell — https://github.com/NVIDIA/OpenShell — Open-source, policy-enforced sandbox runtime for autonomous agents. Enforces filesystem, network-egress, and process isolation through declarative YAML policy (Landlock, network namespaces, seccomp) rather than hardware TEE. Referenced as a concrete implementation of the Network Layer's zero-trust egress posture. Hardware-level confidentiality (encrypted memory/VRAM, attestation) is provided by NVIDIA's separate Confidential Computing / TEE stack, which can sit beneath OpenShell to realize the full TEE-backed Sovereign Shard. Note: as of this writing, OpenShell is early-stage (alpha) software; validate its maturity against your own production requirements.

NVIDIA NeMo Guardrails — https://developer.nvidia.com/nemo-guardrails — Open-source toolkit for constraining LLM/SLM outputs. Referenced as the primary tool for the Model Output Layer — enforcing appropriate responses and preventing disclosure of sensitive information at the inference boundary.

GDPR (General Data Protection Regulation) — EU 2016/679 — The European regulatory framework that defines data sovereignty requirements for the EU region. Cited as the primary driver for Sovereign Shard regional isolation and the jurisdictional boundary enforcement in the Network Layer.

CCPA (California Consumer Privacy Act) — Cal. Civ. Code §1798.100 et seq. — The US state-level data privacy regulation cited alongside GDPR as a defining constraint for Sovereign Shard partitioning in North American deployments.

NIST SP 800-38D — https://csrc.nist.gov/publications/detail/sp/800-38d/final — Recommendation for Block Cipher Modes of Operation: GCM and GMAC. NIST standard for authenticated encryption. Referenced in the security context of data protection — applicable to the encryption of data at rest and in transit within the sovereign boundary, as well as securing gradient exchange in Federated Knowledge Synthesis.

Claude Managed Agents / Gemini Enterprise Agent Platform - Frontier-provider managed agent environments noted as emerging alternatives for sovereign execution environments. Flagged with a caution that security postures for these offerings should be independently validated, as they were relatively new as of the document's publication date (mid-2026).

Simplex Architecture — L. Sha, "Using Simplicity to Control Complexity," IEEE Software, vol. 18, no. 4, pp. 20–28, 2001. https://doi.org/10.1109/MS.2001.936213 — The originating formulation of a trusted, high-assurance controller that supervises and can override a complex, high-performance controller; the direct antecedent of the harness-over-model pattern.

Run-Time Assurance (RTA) — ASTM F3269-21, "Standard Practice for Methods to Safely Bound Behavior of Aircraft Systems Containing Complex Functions Using Run-Time Assurance." https://www.astm.org/f3269-21.html — The aviation-industry codification of RTA for systems whose complex functions cannot be certified by conventional software-assurance methods (e.g., DO-178C); the safety-engineering standard the harness pattern descends from.

Shielding (Safe Reinforcement Learning) — M. Alshiekh, R. Bloem, R. Ehlers, B. Könighofer, S. Niekum, U. Topcu, "Safe Reinforcement Learning via Shielding," Proceedings of the Thirty-Second AAAI Conference on Artificial Intelligence (AAAI-18), pp. 2669–2678, 2018. https://ojs.aaai.org/index.php/AAAI/article/view/11797 — Introduces a reactive "shield" that monitors an agent's actions and corrects them only when the chosen action would violate a temporal-logic safety specification; the reinforcement-learning analog of the Deterministic Harness.

Policy-as-Code — Open Policy Agent (OPA), Cloud Native Computing Foundation. https://www.openpolicyagent.org/ — General-purpose engine for expressing and enforcing declarative rules over system actions; representative of the mechanism the harness uses to evaluate model output against the Golden Rule Set.

Control Barrier Functions — A. D. Ames, S. Coogan, M. Egerstedt, G. Notomista, K. Sreenath, P. Tabuada, "Control Barrier Functions: Theory and Applications," 2019 18th European Control Conference (ECC), pp. 3420–3431. https://arxiv.org/abs/1903.11199 — The control-theoretic formalism for keeping a system within a provably safe set; grounds the "bright-line invariant" concept in formal safe-set theory.

NeMo Guardrails (paper) — T. Rebedea, R. Dinu, M. N. Sreedhar, C. Parisien, J. Cohen, "NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails," Proceedings of EMNLP 2023 (System Demonstrations), pp. 431–445. https://arxiv.org/abs/2310.10501 — The peer-reviewed write-up behind the NeMo Guardrails tool cited above; the constrained-output instantiation applied at the Model Output Layer.

Companion Artifacts Referenced in this Document
  - [Failure Matrix](failure-matrix.md)
  - [Threat Matrix](threat-matrix.md)
  - [Network to Model Interface](network-to-model-interface.md)
  - [Model to Harness Interface](model-to-harness-interface.md)

### Out of Scope
The following items are out of scope for this architecture, but necessary for a comprehensive implementation:
- Training-Time Safety: No section addressing security concerns of the training data
- Model Alignment: No information regarding underlying capabilities of the model
- Supply Chain Security beyond the network layer: Recognized in the threat matrix, but not comprehensive security
- Prompt-Level Red-Teaming: No systematic testing to ensure model rules are caught (enables more simplistic deterministic rules)

### Contributing to this architecture
Contributions to this technical architecture brief are very welcome! If you find a bug or have a suggestion, please open an issue in the repository. To submit changes, please fork the project and submit a pull request. Please ensure your contributions align with our project's goals and maintain the established security and privacy principles.

### Training Data Considerations
Training data is critical to the development and performance of AI models. It is stored in a secure location and is not accessible to the public. It is also encrypted at rest and in transit. Training data should be considered a valuable asset and be protected with the same level of security as the most valuable data in your organization. 

### Production Data Considerations
Production data is used to make decisions in real-time. This data is often multimodal, real-time streams from various sensors. Depending on the application, some or all production data may be captured and stored for audit or training purposes. This process needs to be well-defined to ensure it aligns with all policies and legal requirements. 

### Validation Data Considerations
Validation data is used to ensure the model performance is meeting expectations and not overfitting to the training data. This should be data the model has never seen before to ensure that it is able to generalize to new data.

### Assumptions
This document is intended for an audience with a working knowledge of the technologies and concepts discussed herein. It assumes the reader has a basic understanding of AI concepts, multi-agent systems, cloud infrastructure, and general software development practices. This document is not a tutorial, but rather a high-level overview of the architecture. It is intended to provide a foundation for the design of multi-agent systems that are both safe and effective in production settings. This document will focus on a combination of edge AI and cloud based AI, with edge AI being the primary method of data collection and processing in real-time. The cloud based AI will be used for training and long term storage and is assumed to be used primarily for offline learning and refinement of the edge based AI. 

### Dependencies
The deterministic harness relies on a number of supporting infrastructure and software components to function effectively. These include:
- Cloud infrastructure
- Edge AI infrastructure
- Digital Twins
- Data infrastructure
- Software development practices
- Multi-agent systems
- Small Language Models (SLMs)
- Data pipelines
- Observability frameworks

This architecture is designed to provide a framework for the development and deployment of safe and effective AI agents. It is not a one-size-fits-all solution, but rather a flexible framework that can be adapted to the specific needs of different applications. The deterministic harness is designed to be modular, so that different components can be added or removed as needed. 

## The Deterministic Harness Architecture Overview

The deterministic harness is a layered architecture designed to ensure that AI agents remain safe, controlled, and compliant while operating autonomously. The deterministic harness is built on the principle of "Defense in Depth," providing multiple layers of protection that work together to ensure the reliability and safety of AI systems. Each layer has a specific function, and the layers are designed to work in concert with each other to provide a comprehensive solution for AI safety.

### 1. The Network Layer (Foundation)
* **Function:** Isolation and Sovereignty.
* **Mechanism:** Leveraging **Sovereign Shards** to isolate agentic activities.
* **Constraint:** Automated Data Auditing prevents any non-anonymized data from leaving the sovereign boundary. It ensures that the learning loop (gradients) is decoupled from the personal/proprietary raw data.

### 2. The Model Output Layer (Intelligence)
* **Function:** Real-time Inference and Self-Healing.
* **Mechanism:** Small Language Models (SLMs) at the edge for low-latency decision-making.
* **Cross-Validation:** Utilizing **Sensor Fusion** (e.g., Video + Ultrasound) to cross-verify model outputs. If sensors disagree, the system defaults to a fail-safe state defined in the Harness.

### 3. The Deterministic Harness (Execution)
* **Function:** Governance-as-Code.
* **Mechanism:** A hard-coded validation layer that wraps the agentic workflow. 
* **Dynamic Updates:** The Harness is version-controlled and updated alongside the model. 

### 4. Observability: The Utility Monitor
The entire deterministic harness is wrapped in an observability framework that tracks **Utility Degradation**. If legal or safety constraints in the Harness begin to choke the agent's ability to drive commercial value, the system triggers a "Re-Architecture Alert," preventing the system from becoming a "compliant but useless" asset.

### Architectural Diagrams and Examples

### Network Layer — Detailed Design

![Network Layer](Figure-1.png)

The network layer is the foundation of the Deterministic Harness Architecture. It isolates agentic activities using hardware-based techniques — such as Trusted Execution Environments (TEEs) — to create a secure boundary around each agent and its data. Agents can only access authorized resources, and automated data auditing enforces that no raw or identifiable data leaves the sovereign boundary.

The core mechanism is the **Sovereign Shard**: a TEE-backed execution environment that hosts an agent or a cluster of related agents. Each shard encapsulates the agent code, model weights, and operational data within a single trust boundary. This boundary can be further solidified by deploying within a regional cloud environment — for example, colocating infrastructure with an EU provider to maintain GDPR compliance — ensuring that all data, models, and execution remain within the sovereign perimeter.

In practice, a container orchestrator runs each agent in its own isolated environment within a shard. NVIDIA's OpenShell project (https://github.com/NVIDIA/OpenShell) demonstrates this approach, applying declarative YAML policy — Landlock for filesystem restriction, per-sandbox network namespaces with a controlled egress point, and seccomp for syscall hardening — to enforce that agents reach only explicitly whitelisted endpoints such as an inference API. OpenShell provides the policy-enforced sandbox and egress controls at the network layer; the Model Output and Harness layers add finer-grained controls. Where hardware-level confidentiality is required, NVIDIA's separate Confidential Computing / TEE stack (encrypted memory and attestation) can be layered beneath OpenShell to complete the TEE-backed Sovereign Shard. Note that OpenShell is early-stage (alpha) software as of this writing, so its production-readiness should be validated against your requirements. Frontier providers are also offering similar isolated environments — for example, Anthropic's Claude Managed Agents and Google's Gemini Enterprise Agent Platform — though research should be done to ensure proper environment separation, as these are relatively new offerings as of mid-2026. Each offering has its own security posture.

Observability within the shard relies on cloud-native monitoring — Amazon CloudWatch, Google Cloud Monitoring, or equivalent — to verify that only authorized agents and services access the resources they need. These logs, combined with the OpenShell execution audit trail, provide the end-to-end traceability required for compliance and incident response.

#### Network Layer Capabilities

In a global organization, the network layer must solve for legal sovereignty before addressing technical latency. Three capabilities enable this:

**Sovereign Shards** partition data and compute into regions aligned to regulatory regimes — GDPR, CCPA, or specific national mandates. Each shard is a self-contained trust boundary: agents, models, and data all reside within the same jurisdictional perimeter, and nothing leaves without explicit policy approval.

**Federated Knowledge Synthesis** allows a global model to learn from regional patterns without raw data ever crossing a sovereign boundary. Instead of moving data, the network layer orchestrates the exchange of model gradients between shards. This is architecturally straightforward to describe but operationally complex to implement — each deployment will require custom infrastructure for secure gradient aggregation, cumulative privacy risk management, and resilient distributed agreement across participating shards. Federated Knowledge Synthesis introduces some nuanced, but very real security challenges. The Threat Matrix is a beginning (but not all-inclusive) point to build a comprehensive security architecture.

**Asynchronous Data Ingestion (Air-Gapped Patterns)** addresses environments where bandwidth is the bottleneck — remote manufacturing facilities, maritime operations, or field deployments with intermittent connectivity. These environments use a "Gold Set" ingestion pattern: high-fidelity raw data is captured and validated locally at the edge, while only compressed, anonymized insights or model weight updates are transported over the wire when connectivity is available. This ensures the sovereign shard remains current without depending on continuous network access.


#### Sovereign Cloud and Edge Computing Considerations

Separating agents into sovereign units creates a natural boundary for data governance: each unit controls what flows in and out, ensuring sensitive information stays within its jurisdictional perimeter.

Above the agent level, a sovereign cloud environment hosts core models and long-term data storage within a single regulatory region. This separation enables a tiered model strategy — lightweight SLMs at the edge for real-time decision-making, with larger LLMs in the sovereign cloud handling training and refinement — all without data leaving the compliance boundary. Training data, when required, can be colocated in the same sovereign environment, keeping the full model lifecycle (data, training, weights, inference) within a single jurisdiction.

This structure also simplifies regulatory maintenance. When regulations change, the sovereign boundary makes it straightforward to identify which systems, data, and models are affected — narrowing the scope of any re-architecture effort to a specific region rather than the entire global deployment.

One area requiring particular caution is edge computing. Edge devices operating at the periphery of a sovereign shard must be configured so that even failover and error-recovery communications do not inadvertently route data across sovereign boundaries. Failover paths should be validated against the same egress policies as primary paths.


### Model Output Layer — Detailed Design

![Model Output Layer](Figure-2.png)

The model output layer is the second layer of the deterministic harness. It is responsible for real-time inference and self-healing. It uses small language models (SLMs) at the edge for low-latency decision-making. This is where many organizations spend time securing the models with tools such as NeMo Guardrails (https://developer.nvidia.com/nemo-guardrails), an open-source toolkit for building safe, reliable, and transparent AI applications. The toolkit ensures the LLM or SLM is responding with appropriate information and is not disclosing sensitive information.

Additionally, prompt engineering is used to guide the SLM or LLM to make the correct decisions as well as ensure output format is consistent and predictable. The output from the SLM or LLM is then sent to the deterministic harness for further processing. This layer is focused primarily on the prompt input and the output from the model.

#### Example for the Model Output Layer

Inference at the Edge: Small Language Models (SLMs) and specialized model shards provide low-latency inference at the edge, cross-verified by multi-modal sensor fusion (e.g., ultrasound vs. video) to ensure the model’s "view" of reality is correct. For example, a manufacturing robot relying on a visual AI for defect detection must cross-check with tactile or thermal sensors to prevent false positives (or negatives) that could arise from lighting changes or lens obstructions. This fusion of data streams acts as the "eyes and ears" of the agent, ensuring its perception is grounded in physical reality.

### Deterministic Harness — Detailed Design

![Deterministic Harness](Figure-3.png)

The reliability of the architecture lies in the Deterministic Harness's ability to validate the output of the Model Output Layer and ensure that it is consistent with business rules. This is why clear rules must be defined in the beginning with clear expected outputs from the LLM/SLM layers. The deterministic harness is simply the implementation of the safety rules checking each output of the LLM/SLM with the rules defined at the beginning. Keeping rules simple and explicit allows for the harness to run fast and deterministic, preventing the harness from becoming a bottleneck for the entire system. 

The Deterministic Harness capabilities can range from the very simple, to the very complex. For instance, you have an autonomous vehicle with a vision sensor and an ultrasonic sensor both focused on the forward direction of travel. You may have individual models processing the data from each sensor. Now, in your business rules, you have a rule that prevents the vehicle from driving through an object in the forward direction. You may get a scenario where the vision model detects an object, while the ultrasonic sensor does not. The harness rules would step in and prevent the vehicle from driving through an object in the forward direction, while also logging the event for future review. 

The above rule is a simple example, but then raises the question: what if the models refuse to agree even after an object is removed and in the real world, it would be safe to proceed. This is where the deterministic harness shines. Perhaps, the rule allows the vehicle to proceed slowly and monitor an accelerometer to determine if any impact is made and defines how to handle that event (back up 5 feet and call for service). An alternate path could also be another SLM or LLM call that is independent to evaluate the scenario and make a decision. The rules drive the system in a reliable and safe manner. The rules also engage a human or other autonomous system when the rules are not clear for the given scenario.

The deterministic harness is the critical component that transforms the probabilistic output of the SLM or LLM into a reliable and safe decision. It does not need to make every decision as the ability of the AI systems can make decisions in scenarios that lack clean deterministic boundaries. However, in most multi-agent systems, there are rules that should NEVER be violated. The deterministic harness is the component that ensures these rules are followed and appropriate next steps are taken.

#### Real-World Use of the Deterministic Harness

The Harness is the bridge between AI research and industrial reality. It is a partner to the model, evolving alongside it rather than acting as a static cage.

The Dynamic Sandbox: The harness provides a safe execution environment where the agent’s actions are validated against a "Golden Rule" set in real-time. If an agent proposes an action that violates a safety or business constraint, the harness intercepts and remediates the command before execution. The Digital Twin referenced in this document is a specific simulation that will be custom built for the project and will be a significant undertaking in and of itself.

Compliance Hot-Patching (Legal Drift): In the event of an overnight regulatory shift, the harness acts as the primary "Hot-Patch." New legal parameters are injected into the harness immediately, acting as a compliance filter that blocks non-compliant outputs while the background model undergoes a formal clean-room retrain.

Automated Traceability: Every decision processed by the harness is logged with a full lineage, creating an audit-ready "Evidence of Compliance" for regulators and external stakeholders.

#### Example Ruleset for the Deterministic Harness (illustrative pseudocode)

RULE: forward_obstruction_conflict
  IF vision.detects_object AND NOT ultrasonic.detects_object
  THEN action = PROCEED_SLOW, monitor = ACCELEROMETER
  ON_IMPACT: action = REVERSE_5FT, escalate = SERVICE_CALL
  LOG: severity=WARNING, sensors=[vision, ultrasonic], confidence=[v.conf, u.conf]



### Utility Monitor — Detailed Design

![Utility Monitor](Figure-4.png)

Observability and monitoring is a critical underpinning of the Deterministic Harness. This allows an organization to effectively monitor the activities of a safety system. Primarily, this is a combination of logs from the AI agents, the systems they react with, the network monitoring and the observability of what decisions agents make and why they make them. The system requires multiple tools to provide meaningful insights into the behavior of the AI agents:
1. Log Analysis Tools
2. Amazon Cloud Watch or Google Cloud's operations suite 
3. SIEM tools to correlate logs and provide a unified view of security events.
4. Standard network and endpoint security tools
5. Observability Tools (Prometheus, Grafana, etc.)

The combination of these tools, both log and data analysis, as well as security tools give the organization the needed observability and monitoring capabilities to effectively monitor the activities of a safety system.

#### Practical use of the Utility Monitor in the Deterministic Harness

Observability in this architecture moves beyond simple error logging to monitor the Commercial Health of the AI system.

Reality Gap Monitoring (Parity): The monitor continuously reconciles the Digital Twin against the physical production floor. If telemetry (e.g., vibration, throughput, power draw) diverges from the simulation beyond a 2-sigma threshold (illustrative threshold), the system flags "Reality Gap" drift and pauses the agentic loop for recalibration.

Economic Drift & Utility Score: The monitor tracks the delta between the Agent’s proposed optimizations and the Harness’s allowed actions. If the harness becomes so restrictive that it rejects a majority of high-value agentic activities, the monitor flags Utility Decay, signaling that a re-architecture of the legal or safety constraints is required to maintain ROI.

Immune System Alerts: Rather than simple alarms, the monitor triggers automated drift remediation, tasking the agentic loop with re-aligning the twin to the production reality.

## Details for Building the Deterministic Harness

### Interface Contracts

The Interface Contracts are a critical part of the system as they define the boundaries between layers. This is example code designed to depict the implementation. A full engineering specification should be developed for a production implementation.

#### Network to Model (LLM/SLM) Interface Contract

As there is not a physical data exchange between the layers, the contract is the capabilities and rules defined at deployment. This configuration defines consistent rule sets for each layer and should be aligned in the context of the complete system. 

The code in the Interface Contract is example code, not a complete implementation for deploying an OpenShell container on Google Kubernetes Engine. The OpenShell container will contain the agent code and deterministic harness code with connectivity to Google's Vertex AI service (now Gemini Enterprise Agent Platform).
 
An example of the [Network to Model Interface Contract](network-to-model-interface.md) is linked here.

Breakdown of the Security Layers (code blocks reference the network-to-model-interface.md linked above)
1. Filesystem "Jailing"
In a standard container, an agent might accidentally (or maliciously) read configuration files or environment variables. By using allowed_reads, you create a whitelist. If the agent tries to look at /etc/kubernetes/node-kubeconfig, the OpenShell kernel-level interceptor will block the request before it even reaches the disk.

2. Network "Zero-Trust" Egress
A compromised agent isn't just a data risk — it's a potential foothold into your entire infrastructure. Without egress controls, a hijacked agent can freely reach internal databases, exfiltrate data, or phone home to an attacker's command-and-control server.
The fix is a default-deny posture:
```yaml
default_action: "BLOCK"
```
This keeps the container dark unless explicitly permitted. Domain whitelisting then carves out only what the agent legitimately needs — for example, allowing aiplatform.googleapis.com to reach Gemini Flash while blocking everything else. The agent can do its job, but an attacker who compromises it hits a wall.

3. Resource & Process Hardening
LLMs can sometimes generate code that loops infinitely or spawns thousands of processes.
```yaml
process:
  max_processes: 10        # Prevents the agent from crashing the GKE node by spawning too many processes.

capabilities:
  drop:
    - ALL                  # Drop all Linux capabilities by default.
```
Dropping all capabilities is a "Defense in Depth" move. Even if an attacker finds a way to execute code, they won't have the Linux privileges (like CAP_NET_ADMIN) required to change network settings or escape the container.

4. Non-Root Execution
Notice the 
```yaml
user_id: 1000
```
A secure OpenShell config should never run as root (UID 0). Running as a standard user ensures that even if there is a vulnerability in the container runtime (like containerd), the blast radius is significantly reduced.


#### Model to Harness Interface

The Model to Harness interface is the direct connection between the agent and the production floor. It is a critical part of the system because it defines the boundary between the agent and the physical world, so it must be constructed deliberately alongside the rules-engine code itself. The interface defines how the model's output is presented to, and acted on by, the harness.

The full worked example — the agent prompt contract, the model response, and the harness validation decision — is provided in the linked [Model to Harness Interface Contract](model-to-harness-interface.md). The points below summarize what that example illustrates.

Sovereign shard routing happens before the model ever sees data. In the example, the temperature and equipment location are not merely prompt variables — they are the routing keys that determine which shard, which model version, and which harness rule set apply. A Munich sensor reading never touches a US-region shard. This is where the Network Layer earns its "foundation" designation.

The response schema is embedded in the prompt contract, not just documented externally. This lets the harness validate structurally (did the model return valid JSON matching the schema?) before it validates semantically (does the recommended action comply with the safety rules?). That two-phase validation matters: a malformed response is caught cheaply, before expensive rule evaluation.

The harness does not merely approve or reject — it can augment. The APPROVED_WITH_MODIFICATION decision type is central to the architecture's "partner, not cage" principle. In the linked example, the model makes a sound primary recommendation but omits a maintenance-escalation rule; the harness adds the missing action without overriding the model's primary recommendation.

The monitor payload at the end of the example is what feeds the Utility Score. If rules_blocked climbs relative to rules_total across thousands of decisions, that is the Utility Decay signal described earlier. If model_action_preserved drops to false frequently, the harness is overriding the model too often — indicating that either the model needs retraining or the ruleset needs review.

![End-to-End System Architecture](Figure-5.png) 

## Failure Mode Matrix Integration

The Failure Mode Matrix addresses accidental faults — hardware failures, software bugs, configuration errors, and operational degradation — that can compromise the Deterministic Harness Architecture without any adversarial intent. Where the Threat Matrix asks "how could this system be attacked?", the Failure Mode Matrix asks "how could this system break on its own?" Each of the 12 identified faults maps to a specific Utility Monitor metric with a defined alert threshold, a detection latency, and a three-tier response protocol (immediate automated fallback, short-term remediation, and root cause resolution). This structure ensures that when a fault occurs at 2 AM on a holiday weekend, the system's automated responses buy time while the on-call team mobilizes.

The faults escalate in subtlety as they move up the layers in the deterministic harness, and this escalation pattern is itself an architectural insight. Network layer faults (TEE attestation failure, egress violations, sensor authentication errors) are infrastructure problems that operations teams already know how to detect and respond to — they're serious but familiar. Model output layer faults (schema violations, sensor fusion disagreement, confidence collapse, prompt injection) are harder because the system can appear to be functioning normally while producing unreliable outputs. Harness layer faults (rule misconfiguration, latency spikes, version mismatch) are the most dangerous because they compromise the last line of defense. And the monitor meta-failures (observability gaps, utility decay) affect the organization's ability to detect everything else. Understanding this escalation pattern informs where to invest the most in automated detection — the layers where failures are hardest to see deserve the most sophisticated monitoring.

The matrix also reveals a critical architectural requirement: every layer must have a defined deterministic fallback that the system can fall back to when a component becomes untrustworthy. For most faults this is an explicit SAFE_STATE — but the appropriate fallback varies by fault. When a sensor stream loses authentication (NET-003), the harness escalates to a degraded-mode ruleset that requires human confirmation before any non-trivial action; when the model's confidence collapses (MOD-003), validation thresholds tighten and borderline actions are escalated to a human; when the harness itself experiences a latency spike (HAR-002), it sheds non-critical rules and falls back to a verified CRITICAL-only ruleset. In each case the system has a deterministic, pre-validated response rather than undefined behavior. This fail-safe default is what separates the architecture from systems that log an error and wait for someone to notice. When a component becomes untrustworthy, the system drops into a defined state instead of undefined behavior.

An example of the [Failure Matrix](failure-matrix.md) is linked here.

## Threat Matrix Integration

The Deterministic Harness Architecture is designed around the principle of defense-in-depth, but defense-in-depth only works if you understand what each layer is defending against. The threat matrix maps 15 identified threats — adversarial, systemic, and emergent — to the specific layer responsible for detection and the specific mechanism responsible for remediation. Each threat traces a path through the architecture: where the attack enters, which layer's safety invariant it violates, how the Utility Monitor detects it, and what the harness does in response. Every threat in the matrix maps to a control, and no control is in the design without a threat behind it.

Critically, the threat analysis reveals that the most dangerous failure modes are not single-layer breaches but cross-layer interactions. A compromised sensor (Model Output Layer) is caught by the harness. A weakened harness rule is caught by the Utility Monitor. But a coordinated attack that blinds the monitor, feeds false sensor data, and exploits a previously loosened rule can bypass all three layers simultaneously. The cascading threat category (CAS-T001, CAS-T002) exists specifically to stress-test the architecture's assumption that layers fail independently — and to drive architectural requirements (infrastructure separation, independent safety verification, immutable safety floors) that hold even when that assumption breaks down.

The threat matrix also surfaces a class of risk that no purely technical architecture can solve: governance threats. The Safety-Utility Death Spiral (CAS-T002) and Utility Score Manipulation (MON-T002) demonstrate that organizational pressure to relax safety constraints can erode margins as effectively as any external attacker — and more quietly, because each individual relaxation passes review. These threats drive the architecture's requirement for immutable safety floors, independent safety authority, and cumulative drift tracking, ensuring that the Deterministic Harness Architecture addresses not just how the system can be broken, but how it can be slowly, legitimately negotiated into ineffectiveness.

An example of the [Threat Matrix](threat-matrix.md) is linked here.

## Status and Validation

This document describes an architectural proposal, not a validated system. As of publication there is no production deployment, and no benchmark or empirical performance data; any specific performance figures elsewhere in this work are design targets, not measurements. The Failure Mode and Threat Matrices map the failure surface analytically, a priori — they are a structured hypothesis about where this architecture breaks, to be confirmed or revised against a real implementation. The architecture is offered as a starting point for that work.

## Conclusion

The Deterministic Harness Architecture is a framework for deploying autonomous AI agents in environments where probabilistic intelligence must coexist with deterministic guarantees. Its four layers — network isolation through sovereign shards, model output validated by sensor fusion and guardrails, a hard-coded harness that turns governance into code, and a utility monitor that watches both the agent and the constraints around it — work together as a defense-in-depth system rather than as independent controls. The Failure Mode Matrix and Threat Matrix extend this design by mapping every layer to specific accidental faults and adversarial threats, so the failure surface is mapped rather than assumed. The architecture takes firm positions on a few things — fail-safe defaults, immutable safety floors, independent observability — and stays open on others, where prescriptive guidance would mislead more than help. It is a starting point, and implementers should expect real work to fit it to their environment. Implementers should expect to adapt the network topology to their regulatory regime, tune the harness ruleset to their domain, and harden the federated learning and TEE assumptions against threats specific to their adversary model. What the architecture provides is a structure for those decisions: a vocabulary, a layering, and a discipline for separating the parts of an agentic system that must adapt from the parts that must never drift. Used well, it allows AI agents to operate with genuine autonomy while preserving the predictability, auditability, and legal defensibility that high-stakes deployments demand.


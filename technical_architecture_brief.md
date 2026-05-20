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
4. Failure Mode Matrix
5. Threat Matrix
6. Conclusion



## Executive Summary
The "Deterministic Harness Architecture" is a multi-layered architectural pattern designed to solve the critical challenge of deploying powerful, autonomous AI agents in high-stakes or regulated environments. It ensures that while the AI can learn and adapt (agility), its behavior remains predictable, controllable, and legally compliant (safety). This is a pattern that can be applied to a wide range of AI applications, including autonomous vehicles, medical diagnosis, and financial trading. This architecture also ensures the data used for training as well as the output of the model, can be separated to maintain policy or legal requirements. The network layer is responsible for isolating the AI from the outside world (zero trust), the model output layer is responsible for the AI's intelligence (probabilistic), and the deterministic harness is responsible for the AI's behavior (deterministic). 

### Training Data Considerations
Training data is critical to the development and performance of AI models. It is stored in a secure location and is not accessible to the public. It is also encrypted at rest and in transit. Training data should be considered a valuable asset and be protected with the same level of security as the most valuable data in your organization. 

### Production Data Considerations
Production data is used to make decisions in real-time. This data is often multimodal, real-time streams from various sensors. Depending on the application, some or all production data may be captured and stored for audit or training purposes. This process needs to be well-defined to ensure it aligns with all policies and legal requirements. 

### Validation Data Considerations
Validation data is used to ensure the model performance is meeting expectations and not overfitting to the training data. This should be data the model has never seen before to ensure that it is able to generalize to new data.

### Assumptions
This document is intended for an audience with a working knowledge of the technologies and concepts discussed herein. It assumes the reader has a basic understanding of AI concepts, multi-agent systems, cloud infrastructure, and general software development practices. This document is not a tutorial, but rather a high-level overview of the architecture. It is intended to provide a foundation for the design of multi-agent systems that are both safe and effective in production settings. This document will focus on a combination of edge AI and cloud based AI, with edge AI being the primary method of data collection and processing in real-time. The cloud based AI will be used for training and long term storage and is assumed to be used primarily for offline learning and refinement of the edge based AI. The concepts should apply to multi-agent system deployed only in cloud environments as well.

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

This architecture is designed to provide a framework for the development and deployment of safe and effective AI agents. It is not a one-size-fits-all solution, but rather a flexible framework that can be adapted to the specific needs of different applications. The stack is designed to be modular, so that different components can be added or removed as needed. 

## The Deterministic Harness

The deterministic harness is a layered architecture designed to ensure that AI agents remain safe, controlled, and compliant while operating autonomously. The stack is built on the principle of "Defense in Depth," providing multiple layers of protection that work together to ensure the reliability and safety of AI systems. Each layer has a specific function, and the layers are designed to work in concert with each other to provide a comprehensive solution for AI safety.

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
The entire stack is wrapped in an observability framework that tracks **Utility Degradation**. If legal or safety constraints in the Harness begin to choke the agent's ability to drive commercial value, the system triggers a "Re-Architecture Alert," preventing the system from becoming a "compliant but useless" asset.

### Architectural Diagrams and Examples

#### 1. The Network Layer (Foundation)

![Network Layer](network_layer.jpg)

The network layer is the foundation of the Deterministic Harness Architecture. It isolates agentic activities using hardware-based techniques — such as Trusted Execution Environments (TEEs) — to create a secure boundary around each agent and its data. Agents can only access authorized resources, and automated data auditing enforces that no raw or identifiable data leaves the sovereign boundary.

The core mechanism is the **Sovereign Shard**: a TEE-backed execution environment that hosts an agent or a cluster of related agents. Each shard encapsulates the agent code, model weights, and operational data within a single trust boundary. This boundary can be further solidified by deploying within a regional cloud environment — for example, colocating infrastructure with an EU provider to maintain GDPR compliance — ensuring that all data, models, and execution remain within the sovereign perimeter.

In practice, a container orchestrator runs each agent in its own isolated environment within a shard. NVIDIA's OpenShell project (https://build.nvidia.com/openshell) demonstrates this approach, using Confidential Computing to enforce that agents have network access only to explicitly whitelisted endpoints such as an inference API. OpenShell provides coarse-grained isolation at the network layer; the Model Output and Harness layers add finer-grained controls. Frontier providers are also providing similar isolated environments such as Claude Managed Agents and Gemini Managed Agents, though research should be done to ensure proper environment separation as these are relatively new offerings as of May 2026.

Observability within the shard relies on cloud-native monitoring — Amazon CloudWatch, Google Cloud Monitoring, or equivalent — to verify that only authorized agents and services access the resources they need. These logs, combined with the OpenShell execution audit trail, provide the end-to-end traceability required for compliance and incident response.

##### Network Layer Capabilities

In a global organization, the network layer must solve for legal sovereignty before addressing technical latency. Three capabilities enable this:

**Sovereign Shards** partition data and compute into regions aligned to regulatory regimes — GDPR, CCPA, or specific national mandates. Each shard is a self-contained trust boundary: agents, models, and data all reside within the same jurisdictional perimeter, and nothing leaves without explicit policy approval.

**Federated Knowledge Synthesis** allows a global model to learn from regional patterns without raw data ever crossing a sovereign boundary. Instead of moving data, the network layer orchestrates the exchange of model gradients between shards. This is architecturally straightforward to describe but operationally complex to implement — each deployment will require custom infrastructure for secure gradient aggregation, differential privacy accounting, and Byzantine-fault-tolerant consensus across participating shards.

**Asynchronous Data Ingestion (Air-Gapped Patterns)** addresses environments where bandwidth is the bottleneck — remote manufacturing facilities, maritime operations, or field deployments with intermittent connectivity. These environments use a "Gold Set" ingestion pattern: high-fidelity raw data is captured and validated locally at the edge, while only compressed, anonymized insights or model weight updates are transported over the wire when connectivity is available. This ensures the sovereign shard remains current without depending on continuous network access.


##### Sovereign Cloud and Edge Computing Considerations

Separating agents into sovereign units creates a natural boundary for data governance: each unit controls what flows in and out, ensuring sensitive information stays within its jurisdictional perimeter.

Above the agent level, a sovereign cloud environment hosts core models and long-term data storage within a single regulatory region. This separation enables a tiered model strategy — lightweight SLMs at the edge for real-time decision-making, with larger LLMs in the sovereign cloud handling training and refinement — all without data leaving the compliance boundary. Training data, when required, can be colocated in the same sovereign environment, keeping the full model lifecycle (data, training, weights, inference) within a single jurisdiction.

This structure also simplifies regulatory maintenance. When regulations change, the sovereign boundary makes it straightforward to identify which systems, data, and models are affected — narrowing the scope of any re-architecture effort to a specific region rather than the entire global deployment.

One area requiring particular caution is edge computing. Edge devices operating at the periphery of a sovereign shard must be configured so that even failover and error-recovery communications do not inadvertently route data across sovereign boundaries. Failover paths should be validated against the same egress policies as primary paths.


#### 2. The Model Output Layer (Intelligence)

![Model Output Layer](model_output_layer.jpg)

The model output layer is the second layer of the AI Safety Stack. It is responsible for real-time inference and self-healing. It uses small language models (SLMs) at the edge for low-latency decision-making. This is where many organizations spend time securing the models with tools such as NeMo Guardrails (https://developer.nvidia.com/nemo-guardrails), an open-source toolkit that helps developers build safe, reliable, and transparent AI applications. The toolkit ensures the LLM or SLM is responding with appropriate information and is not disclosing sensitive information.

Additionally, prompt engineering is used to guide the SLM or LLM to make the correct decisions as well as ensure output format is consistent and predictable. The output from the SLM or LLM is then sent to the deterministic harness for further processing. This layer is focused primarily on the prompt input and the output from the model.

##### Example for the Model Output Layer

Inference at the Edge: Small Language Models (SLMs) and specialized model shards provide low-latency inference at the edge, cross-verified by multi-modal sensor fusion (e.g., ultrasound vs. video) to ensure the model’s "view" of reality is correct. For example, a manufacturing robot relying on a visual AI for defect detection must cross-check with tactile or thermal sensors to prevent false positives (or negatives) that could arise from lighting changes or lens obstructions. This fusion of data streams acts as the "eyes and ears" of the agent, ensuring its perception is grounded in physical reality.

#### 3. The Deterministic Harness (Execution)

![Deterministic Harness](deterministic_harness.jpg)

The reliability of the AI safety stack lies in the Deterministic Harness's ability to validate the output of the Model Output Layer and ensure that it is consistent with business rules. This is why clear rules must be defined in the beginning with clear expected outputs from the LLM/SLM layers. The deterministic harness is simply the implementation of the safety rules checking each output of the LLM/SLM with the rules defined at the beginning. Keeping rules simple and explicit allows for the harness to run fast and deterministic, preventing the harness from becoming a bottleneck for the entire system. 

The Deterministic Harness capabilities can range from the very simple, to the very complex. For instance, you have an autonomous vehicle with a vision sensor and an ultrasonic sensor both focused on the forward direction of travel. You may have individual models processing the data from each sensor. Now, in your business rules, you have a rulet that prevents the vehicle from driving through an object in the forward direction. You may get a scenario where the vision model detects an object, while the ultrasonic sensor does not. The harness rules would step in and prevent the vehicle from driving through an object in the forward direction, while also logging the event for future review. 

The above rule is a simple example, but then raises the question, what if the models refuse to agree even after an object is removed and in the real world, it would be safe to proceed. This is where the deterministic harness shines. Perhaps, the rule allows the vehicle to proceed slowly and monitor an accelerometer to determine if any impact is made and defines how to handle that event (back up 5 feet and call for service). An alternate path could also be another SLM or LLM call that is independent to evaluate the scenario and make a decision. The rules drive the system in a reliable and safe manner. The rules also engage a human or other autonomous system when the rules are not clear for the given scenario.

The deterministic harness is the critical component that transforms the probabilistic output of the SLM or LLM into a reliable and safe decision. It does not need to make every decision as the ability of the AI systems is to make a decision that may not have clear deterministic traits. However, in most multi-agent systems, there is some level of rules that should NEVER be violated. The deterministic harness is the component that ensures these rules are followed and appropriate next steps are taken.

##### Real-World Use of the Deterministic Harness

The Harness is the bridge between AI research and industrial reality. It is a partner to the model, evolving alongside it rather than acting as a static cage.

The Dynamic Sandbox: The harness provides a safe execution environment where the agent’s actions are validated against a "Golden Rule" set in real-time. If an agent proposes an action that violates a safety or business constraint, the harness intercepts and remediates the command before execution. The Digital Twin referenced later in this document is a specific simulation that will be custom built for the project and will be a significant undertaking in and of itself.

Compliance Hot-Patching (Legal Drift): In the event of an overnight regulatory shift, the harness acts as the primary "Hot-Patch." New legal parameters are injected into the harness immediately, acting as a compliance filter that blocks non-compliant outputs while the background model undergoes a formal clean-room retrain.

Automated Traceability: Every decision processed by the harness is logged with a full lineage, creating an audit-ready "Evidence of Compliance" for regulators and external stakeholders.

##### Example Ruleset for the Deterministic Harness

RULE: forward_obstruction_conflict
  IF vision.detects_object AND NOT ultrasonic.detects_object
  THEN action = PROCEED_SLOW, monitor = ACCELEROMETER
  ON_IMPACT: action = REVERSE_5FT, escalate = SERVICE_CALL
  LOG: severity=WARNING, sensors=[vision, ultrasonic], confidence=[v.conf, u.conf]



#### 4. The Utility Monitor

![Utility Monitor](utility_monitor.jpg)

Observability and monitoring is a critical underpinning of the Deterministic Harness. This allows an organization to effectively monitor the activities of a safety system. Primarily, this is a combination of logs from the AI agents, the systems they react with, the network monitoring and the observability of what decisions agents make and why they make them. The system requires multiple tools to provide meaningful insights into the behavior of the AI agents:
1. Log Analysis Tools
2. Amazon Cloud Watch or Google Cloud's operations suite 
3. SIEM tools to correlate logs and provide a unified view of security events.
4. Standard network and endpoint security tools
5. Observability Tools (Prometheus, Grafana, etc.)

The combination of these tools, both log and data analysis, as well as security tools give the organization the needed observability and monitoring capabilities to effectively monitor the activities of a safety system.

##### Practical use of the Utility Monitor in the Deterministic Harness

Observability in this stack moves beyond simple error logging to monitor the Commercial Health of the AI system.

Reality Gap Monitoring (Parity): The monitor continuously reconciles the Digital Twin against the physical production floor. If telemetry (e.g., vibration, throughput, power draw) diverges from the simulation beyond a 2-sigma threshold, the system flags "Reality Gap" drift and pauses the AutoResearch loop for recalibration.

Economic Drift & Utility Score: The monitor tracks the delta between the Agent’s proposed optimizations and the Harness’s allowed actions. If the harness becomes so restrictive that it rejects a majority of high-value agentic activities, the monitor flags Utility Decay, signaling that a re-architecture of the legal or safety constraints is required to maintain ROI.

Immune System Alerts: Rather than simple alarms, the monitor triggers automated drift remediation, tasking the agentic loop with re-aligning the twin to the production reality.

### Details to Build-out for Production-Ready Architecture

##### Interface Contracts

The Interface Contracts are a critical part of the system as they define the boundaries between layers. This is example code designed to depict the implementation. A full engineering specification should be developed for a production implementation.

###### Network to Model (LLM/SLM) Interface Contract

As there is not a physical data exchange between the layers, the contract is the capabilities and rules defined at deployment. This configuration defines consistent rule sets for each layer and should be aligned in the context of the complete system. 

Below is example code, not a complete implementation for deploying an OpenShell container on Google Kubernetes Engine. The OpenShell container will contain the agent code and deterministic harness code with connectivity to Google's Vertex AI service (now Gemini Enterprise).
 
An example of the [Network to Model Interface Contract](network_to_model_interface.md) is linked here.

Breakdown of the Security Layers
1. Filesystem "Jailing"
In a standard container, an agent might accidentally (or maliciously) read configuration files or environment variables. By using allowed_reads, you create a whitelist. If the agent tries to look at /etc/kubernetes/node-kubeconfig, the OpenShell kernel-level interceptor will block the request before it even reaches the disk.

2. Network "Zero-Trust" Egress
This is the most common failure point in AI deployments.```yaml site source   ``` If your agent is compromised, an attacker might try to use it to scan your internal network.

default_action: "BLOCK": Ensures that the container is "dark" by default.

Domain Whitelisting: By specifying aiplatform.googleapis.com, you ensure the agent can talk to Gemini Flash, but it cannot talk to a Command & Control (C2) server or your internal database.

3. Resource & Process Hardening
LLMs can sometimes generate code that loops infinitely or spawns thousands of processes.

max_processes: Prevents the agent from crashing the GKE node by spawning too many threads.

capabilities: drop: - ALL: This is a "Defense in Depth" move. Even if an attacker finds a way to execute code, they won't have the Linux privileges (like CAP_NET_ADMIN) required to change network settings or escape the container.

4. Non-Root Execution
Notice the user_id: 1000. A secure OpenShell config should never run as root (UID 0). Running as a standard user ensures that even if there is a vulnerability in the container runtime (like containerd), the blast radius is significantly reduced.

##### Model to Harness Interface

The Model to Harness interface is the direct connection between the agent and the production floor. It is a critical part of the system as it defines the boundaries between the agent and the physical world. Therefore, the system requires the construction of the interface in addition to the actual rules engine code. The interface defines how the model behaves with the harness.

An example of the [Model to Harness Interface Contract](model_to_harness_interface.md) is linked here.

The sovereign shard routing happens before the model ever sees data. The temperature and equipment location aren't just prompt variables — they're the routing keys that determine which shard, which model version, and which harness rule set apply. A Munich sensor reading never touches a US-region shard. This is where the network layer earns its "foundation" label.

The response schema is embedded in the prompt contract, not just documented externally. This means the harness can validate structurally (did the model return valid JSON matching the schema?) before it validates semantically (does the recommended action comply with safety rules?). That two-phase validation is important — a malformed response gets caught cheaply before expensive rule evaluation.

The harness doesn't just approve or reject — it can augment. The APPROVED_WITH_MODIFICATION decision type is critical for your document's argument. In the example, the model made a good call but missed a maintenance escalation rule. The harness added it without overriding the model's primary recommendation. This is your "partner, not cage" concept made concrete.

The monitor payload at the bottom is what feeds your Utility Score. If rules_blocked starts climbing relative to rules_total across thousands of decisions, that's your Utility Decay signal. If model_action_preserved drops to false frequently, the harness is overriding the model too often, and either the model needs retraining or the rules need review.

![End-to-End System Architecture](end_to_end_system_architecture.jpg) 

### Failure Mode Matrix Integration

The Failure Mode Matrix addresses accidental faults — hardware failures, software bugs, configuration errors, and operational degradation — that can compromise the Deterministic Harness Architecture without any adversarial intent. Where the Threat Matrix asks "how could this system be attacked?", the Failure Mode Matrix asks "how could this system break on its own?" Each of the 12 identified faults maps to a specific Utility Monitor metric with a defined alert threshold, a detection latency, and a three-tier response protocol (immediate automated fallback, short-term remediation, and root cause resolution). This structure ensures that when a fault occurs at 2 AM on a holiday weekend, the system's automated responses buy time while the on-call team mobilizes.

The faults escalate in subtlety as they move up the stack, and this escalation pattern is itself an architectural insight. Network layer faults (TEE attestation failure, egress violations, sensor authentication errors) are infrastructure problems that operations teams already know how to detect and respond to — they're serious but familiar. Model output layer faults (schema violations, sensor fusion disagreement, confidence collapse, prompt injection) are harder because the system can appear to be functioning normally while producing unreliable outputs. Harness layer faults (rule misconfiguration, latency spikes, version mismatch) are the most dangerous because they compromise the last line of defense. And the monitor meta-failures (observability gaps, utility decay) affect the organization's ability to detect everything else. Understanding this escalation pattern informs where to invest the most in automated detection — the layers where failures are hardest to see deserve the most sophisticated monitoring.

The matrix also reveals a critical architectural requirement: every layer must have a defined SAFE_STATE that the harness can fall back to when any component becomes untrustworthy. Whether a sensor stream loses authentication (NET-003), the model's confidence collapses (MOD-003), or the harness itself experiences a latency spike (HAR-002), the system always has a deterministic, pre-validated fallback action. This fail-safe-by-default property is what distinguishes the Deterministic Harness Architecture from systems that simply log errors and hope for human intervention — the architecture degrades gracefully rather than failing silently.

An example of the [Failure Matrix](failure_matrix.md) is linked here.

### Threat Matrix Integration

The Deterministic Harness Architecture is designed around the principle of defense-in-depth, but defense-in-depth only works if you understand what each layer is defending against. The threat matrix maps 15 identified threats — adversarial, systemic, and emergent — to the specific layer responsible for detection and the specific mechanism responsible for remediation. Each threat traces a path through the architecture: where the attack enters, which layer's safety invariant it violates, how the Utility Monitor detects it, and what the harness does in response. This mapping ensures that no threat exists in a conceptual vacuum — every identified risk has a corresponding architectural control, and every control exists because of an identified risk.

Critically, the threat analysis reveals that the most dangerous failure modes are not single-layer breaches but cross-layer interactions. A compromised sensor (Model Output Layer) is caught by the harness. A weakened harness rule is caught by the Utility Monitor. But a coordinated attack that blinds the monitor, feeds false sensor data, and exploits a previously loosened rule can bypass all three layers simultaneously. The cascading threat category (CAS-T001, CAS-T002) exists specifically to stress-test the architecture's assumption that layers fail independently — and to drive architectural requirements (infrastructure separation, independent safety verification, immutable safety floors) that hold even when that assumption breaks down.

The threat matrix also surfaces a class of risk that no purely technical architecture can solve: governance threats. The Safety-Utility Death Spiral (CAS-T002) and Utility Score Manipulation (MON-T002) demonstrate that organizational pressure to relax safety constraints can erode margins as effectively as any external attacker — and more quietly, because each individual relaxation passes review. These threats drive the architecture's requirement for immutable safety floors, independent safety authority, and cumulative drift tracking, ensuring that the Deterministic Harness Architecture addresses not just how the system can be broken, but how it can be slowly, legitimately negotiated into ineffectiveness.

An example of the [Threat Matrix](threat_matrix.md) is linked here.

## Conclusion
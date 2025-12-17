# AI Infra Engineer Framework

## Section 0: Pre-requisites

### Linux internals – cgroups, namespaces, NUMA, file descriptors
### Containers deep – runtimes, cgroups v2, device passthrough
### Kubernetes internals – scheduler, eviction, PDBs, probes
### Networking – TCP basics, latency vs throughput, DNS, load-balancing
### Infra as Code – Terraform, state, drift, lifecycle
### On-call mindset – debugging live systems, postmortems

---

## Section 1: Why GPUs Exist

### 1.1 Why CPUs broke and GPUs emerged
* Explains the strengths and limits of CPUs as general-purpose, low-parallel compute.
* Explains GPUs as throughput-oriented systems built for massive parallelism.
* Shows how workload characteristics shifted toward parallel execution.
* Establishes GPUs as a response to structural compute limits, not a "faster CPU".

### 1.2 Why AI workloads require GPUs
* Explains the computational patterns common in AI workloads.
* Shows why these patterns map poorly to CPUs but efficiently to GPUs.
* Establishes GPUs as a requirement for practical AI, not an optimization.

### 1.3 Why traditional infra assumptions break once GPUs are introduced
* Explains why GPU resources are scarce, fixed, and expensive compared to CPUs.
* Shows how retry-based, stateless, elastic infra patterns fail with GPUs.
* Establishes the need for different scheduling, scaling, and failure handling.

### 1.4 Activities
* Analyze historical CPU scaling limits and explain why parallelism, not clock speed, became the bottleneck.
* Explain how GPUs changed infra assumptions without referencing specific AI frameworks.
* Design a hypothetical infra assuming GPUs are “faster CPUs” and document where it fails.
* Compare CPU-first vs GPU-first system designs and identify failure modes.
* Present: “Why GPUs Exist and Why Infra Must Change”

---

## Section 2: GPU Resource Model (Device Level)

### 2.1 What resources a GPU exposes
* Explains a GPU as a set of constrained resources, not a single opaque unit.
* Introduces VRAM, compute units, and memory bandwidth as independent limits.
* Establishes what infra is truly allocating and running out of.

### 2.2 How GPU memory allocation works
* Explains how GPU memory is allocated and held by processes.
* Introduces fragmentation as a first-order constraint.
* Establishes why "free VRAM" does not guarantee new workloads can start.

### 2.3 What GPU utilization actually measures
* Explains what GPU utilization metrics represent.
* Shows why high utilization can still mean poor performance.
* Establishes why utilization alone is an unreliable signal.

### 2.4 Why GPUs are not preemptible like CPUs
* Explains why GPU workloads cannot be cheaply paused or rescheduled.
* Shows how long-running GPU tasks block resources.
* Establishes why retries and preemption behave differently on GPUs.

### 2.5 CPU–GPU Interaction & Hidden Bottlenecks
* Explains how GPUs depend on CPUs for feeding data and launching work.
* Shows why CPU saturation can leave GPUs underutilized.
* Establishes CPU–GPU balance as a core performance constraint.

### 2.6 Activities
* Explain the GPU resource model purely in terms of constraints, not capabilities.
* Analyze why VRAM fragmentation causes failures even when memory appears available.
* Reproduce GPU memory pressure scenarios and document allocation behavior.
* Design a monitoring system that relies only on utilization and explain why it misleads.
* Present: “What a GPU Resource Actually Is”

---

## Section 3: From One GPU to Many (Topology Basics)

### 3.1 Why multi-GPU nodes exist
* Explains why some workloads cannot fit or run efficiently on a single GPU.
* Introduces scale-up (more GPUs per node) as a necessity, not an optimization.
* Establishes multi-GPU nodes as a response to workload size and communication needs.

### 3.2 What "GPU topology" means
* Explains that GPUs inside a node are not equally connected.
* Introduces the idea that communication paths affect performance.
* Establishes topology as a property infra must account for, not the application.

### 3.3 PCIe vs NVLink
* Explains the difference between general-purpose and high-bandwidth GPU interconnects.
* Shows why communication speed becomes a bottleneck at scale.
* Establishes why hardware choices directly affect workload scalability.

### 3.4 Why some multi-GPU jobs scale and others don’t
* Explains why adding GPUs does not guarantee linear speedup.
* Introduces communication overhead as a dominant limiting factor.
* Establishes that infra topology can make or break multi-GPU performance.

### 3.5 Distributed Training Internals & Collective Communication
* Explains how data is exchanged across GPUs during training.
* Shows why communication patterns dominate performance at scale.
* Establishes distributed training behavior as topology-dependent.

### 3.6 Activities
* Explain why multi-GPU systems are not linear extensions of single-GPU systems.
* Analyze how topology influences communication cost and scaling limits.
* Map GPU interconnect topology on a real or simulated node.
* Compare scaling behavior across different GPU topologies.
* Present: “Why Topology Limits Scale Before Compute”

---

## Section 4: GPU Sharing Models

### 4.1 Why sharing GPUs is hard
* Explains why GPUs lack fine-grained, cheap isolation like CPUs.
* Shows how memory, execution, and bandwidth are tightly coupled.
* Establishes sharing as a trade-off problem, not a default win.

### 4.2 Full GPU allocation (one job, one GPU)
* Explains the simplest and safest sharing model.
* Shows why it maximizes isolation and predictability.
* Establishes the baseline against which other models are judged.

### 4.3 GPU time-slicing (process-level sharing)
* Explains how multiple processes share a GPU over time.
* Shows where interference and performance variance come from.
* Establishes why time-slicing improves utilization but reduces predictability.

### 4.4 MIG (hardware-level partitioning)
* Explains how GPUs can be split into hardware-isolated slices.
* Shows what MIG guarantees and what it does not.
* Establishes MIG as a middle ground between isolation and utilization.

### 4.5 Trade-offs: isolation vs utilization vs complexity
* Explains why no sharing model is universally “best”.
* Shows how operational complexity grows with utilization goals.
* Establishes sharing decisions as infra policy, not application choice.

### 4.6 Activities
* Explain why GPU sharing is fundamentally harder than CPU sharing.
* Compare isolation guarantees across full GPU, time-slicing, and MIG.
* Design a GPU sharing policy for mixed training and inference workloads.
* Evaluate utilization gains versus operational complexity for each sharing model.
* Present: “GPU Sharing as an Infra Trade-off”

---

## Section 5: GPU Failure Modes & Debugging

### 5.1 Common GPU failure classes
* Explains the broad categories of failures GPUs experience.
* Separates memory, compute, driver, and interconnect issues.
* Establishes a mental model for classifying GPU problems before debugging.

### 5.2 What GPU OOM actually looks like
* Explains how GPU out-of-memory failures occur.
* Shows why GPU OOMs are abrupt and non-recoverable.
* Establishes why partial progress or graceful degradation is rare.

### 5.3 Xid errors, ECC errors, and GPU resets
* Explains what hardware-level GPU errors represent.
* Shows how GPUs report faults differently than CPUs.
* Establishes why some errors poison a GPU until reset or replacement.

### 5.4 Why GPU failures are noisy, sticky, and hard to recover
* Explains why GPU failures often cascade across jobs.
* Shows how errors persist beyond a single process.
* Establishes why automated recovery is difficult.

### 5.5 What signals exist for detecting GPU health
* Explains what observability signals GPUs expose.
* Shows the limits of logs and metrics for early detection.
* Establishes why GPU health monitoring requires specialized signals.

### 5.6 Activities
* Classify GPU failures by root cause and persistence.
* Analyze why GPU OOMs behave differently from CPU OOMs.
* Simulate a poisoned GPU scenario and document recovery limitations.
* Build a failure decision tree for GPU incidents.
* Present: “Why GPU Failures Are Sticky and Expensive”

---

## Section 6: GPU Scheduling & Placement

### 6.1 Why GPU scheduling is harder than CPU scheduling
* Explains why GPUs cannot be treated as interchangeable, fungible resources.
* Shows how placement mistakes lead to idle GPUs or failed jobs.
* Establishes scheduling as a first-order concern in GPU infra.

### 6.2 How GPUs enter Kubernetes
* Explains how GPUs are exposed to the scheduler.
* Shows that GPUs are not native K8s resources by default.
* Establishes why GPU scheduling depends on additional integration layers.

### 6.3 Node selection, taints, and tolerations for GPUs
* Explains how GPU-capable nodes are separated from general-purpose nodes.
* Shows how workloads are constrained to specific hardware.
* Establishes placement control as an infra responsibility.

### 6.4 Why “any free GPU” is a lie
* Explains why GPU count alone is insufficient for correct placement.
* Shows how topology, sharing mode, and memory shape matter.
* Establishes why naive bin-packing fails for GPUs.

### 6.5 Scheduling trade-offs that impact cost and reliability
* Explains how scheduling decisions directly affect GPU utilization.
* Shows the tension between throughput, fairness, and isolation.
* Establishes scheduling as a cost and reliability lever, not just correctness.

### 6.6 Organizational & Policy Constraints in GPU Scheduling
* Explains how quotas, priorities, and policies shape scheduling outcomes.
* Shows why many “scheduler bugs” are actually policy decisions.
* Establishes GPU scheduling as both a technical and organizational system.

### 6.7 Activities
* Explain why GPU scheduling cannot rely on naive bin-packing.
* Analyze how placement decisions affect cost and reliability.
* Design a GPU scheduling policy under competing workload demands.
* Simulate scheduling outcomes under different policy constraints.
* Present: “Why ‘Any Free GPU’ Is a Lie”

---

## Section 7: AI Workload Execution Models

### 7.1 Training vs inference (infra impact)
* Explains how training and inference place fundamentally different demands on infra.
* Shows why uptime, scaling, and failure tolerance differ between the two.
* Establishes why they cannot be operated the same way.

### 7.2 Batch jobs vs long-running services
* Explains the execution patterns of finite vs continuous workloads.
* Shows how scheduling, retries, and resource holding differ.
* Establishes why infra must distinguish between job types.

### 7.3 Why checkpoints exist and when they matter
* Explains checkpoints as a response to long-running, expensive computation.
* Shows how checkpoints change failure recovery and scheduling behavior.
* Establishes checkpoints as an infra concern, not just an ML feature.

### 7.4 Execution phases: startup, warmup, steady-state, teardown
* Explains that GPU workloads have distinct lifecycle phases.
* Shows why resource usage and failure risk vary by phase.
* Establishes why infra behavior must adapt across phases.

### 7.5 What “job completion” means for GPU workloads
* Explains completion beyond “process exited successfully”.
* Shows how partial progress, retries, and recovery factor in.
* Establishes correctness and cost as part of completion semantics.

### 7.6 Activities
* Explain why training and inference require different execution semantics.
* Analyze retry behavior across different workload types.
* Design checkpointing strategies for long-running GPU jobs.
* Compare batch jobs and long-running services under failure.
* Present: “Execution Semantics of AI Workloads”

---

## Section 8: Data, Storage & I/O for AI Workloads

### 8.1 Why data access is often the real bottleneck
* Explains why GPUs frequently wait on data instead of compute.
* Shows how poor I/O hides behind “slow model” symptoms.
* Establishes data access as a primary limiter of GPU efficiency.

### 8.2 Object storage vs local disks vs network filesystems
* Explains the trade-offs between common storage backends.
* Shows how latency, throughput, and consistency differ.
* Establishes why storage choice must match workload behavior.

### 8.3 Dataset locality and GPU utilization
* Explains why where data lives matters as much as how fast GPUs are.
* Shows how remote data access reduces effective GPU usage.
* Establishes locality as an infra-level optimization lever.

### 8.4 Checkpoints, artifacts, and model storage
* Explains the different types of data produced and consumed by AI workloads.
* Shows why durability and access patterns differ across them.
* Establishes storage strategy as part of execution design.

### 8.5 Why fast GPUs + slow I/O wastes money
* Explains how I/O bottlenecks directly translate to GPU idle time.
* Shows the cost impact of mismatched compute and storage.
* Establishes balanced infra as a cost-control requirement.

### 8.6 Activities
* Explain why data access, not compute, often limits GPU efficiency.
* Analyze how storage latency and throughput affect utilization.
* Profile GPU idle time caused by slow data pipelines.
* Redesign a data pipeline to improve locality and throughput.
* Present: “Why Fast GPUs Don’t Matter With Slow I/O”

---

## Section 9: Inference Infrastructure & Serving

### 9.1 What makes inference different from training
* Explains why inference prioritizes latency and availability over throughput.
* Shows how failure tolerance is lower than in training.
* Establishes inference as a production-critical workload.

### 9.2 Online vs batch inference
* Explains synchronous vs asynchronous inference patterns.
* Shows how scaling and resource usage differ.
* Establishes why infra choices depend on access patterns.

### 9.3 Common inference runtimes
* Explains why specialized runtimes exist for serving models.
* Shows how runtimes differ in memory use and batching behavior.
* Establishes runtime choice as an infra decision.

### 9.4 Runtime trade-offs: latency, throughput, memory
* Explains the inherent trade-offs between serving goals.
* Shows why optimizing one dimension hurts another.
* Establishes tuning as an ongoing infra task.

### 9.5 Model loading, warmup, and cold starts
* Explains why models are expensive to load into memory.
* Shows how cold starts impact latency and availability.
* Establishes warmup as a production concern.

### 9.6 Why autoscaling inference is harder than it looks
* Explains why request rate alone is insufficient for scaling.
* Shows how model memory and startup time constrain scaling.
* Establishes careful autoscaling as critical for cost and reliability.

### 9.7 Activities
* Explain why inference prioritizes different infra guarantees than training.
* Analyze failure tolerance differences between online and batch inference.
* Design an inference system optimized for latency and cost.
* Compare cold-start and warm-start serving strategies.
* Present: “Running Inference Systems in Production”

---

## Section 10: Model Packaging & Deployment Artifacts

### 10.1 Why models are not “just files”
* Explains why models include code, configuration, and runtime assumptions.
* Shows how incompatibilities surface at deployment time.
* Establishes packaging as a source of failure if ignored.

### 10.2 Model formats and their implications
* Explains common model formats and what they encode.
* Shows how formats affect portability and performance.
* Establishes format choice as an infra constraint.

### 10.3 Quantization and precision choices
* Explains why models are deployed at different precisions.
* Shows how precision affects memory, speed, and accuracy.
* Establishes quantization as an infra-performance trade-off.

### 10.4 Containerizing models and runtimes
* Explains how models are bundled with serving infrastructure.
* Shows why containers must align with drivers and runtimes.
* Establishes container builds as part of deployment correctness.

### 10.5 Versioning, rollbacks, and compatibility
* Explains why model changes must be reversible.
* Shows how mismatches cause silent failures.
* Establishes disciplined rollout as an infra responsibility.

### 10.6 Model Release Pipelines & CI/CD for AI
* Explains how models move from experimentation to production.
* Shows why model rollout carries different risks than code rollout.
* Establishes CI/CD as a safety mechanism for model deployment.

### 10.7 Activities
* Explain why models are deployment artifacts, not static files.
* Analyze compatibility risks across model formats and runtimes.
* Package the same model in multiple formats and compare behavior.
* Design a rollback strategy for a breaking model change.
* Present: “Model Packaging and Deployment Risk”

---

## Section 11: Security & Multi-Tenancy for AI Infra

### 11.1 GPU isolation boundaries
* Explains where isolation can and cannot be enforced.
* Shows the limits of pod-, node-, and hardware-level isolation.
* Establishes isolation as a layered concern.

### 11.2 Secrets and access control
* Explains why models and data require protection.
* Shows how credentials leak through poor infra design.
* Establishes access control as part of infra, not application code.

### 11.3 Supply chain risks
* Explains risks introduced by images, dependencies, and model weights.
* Shows how trust boundaries are crossed during deployment.
* Establishes supply chain security as an infra responsibility.

### 11.4 Abuse prevention for inference endpoints
* Explains how public inference can be misused.
* Shows cost and availability risks from abuse.
* Establishes guardrails as necessary for production exposure.

### 11.5 Activities
* Explain why GPU isolation is inherently imperfect.
* Analyze security risks in multi-tenant GPU environments.
* Design a security model for shared inference infrastructure.
* Evaluate blast radius of a compromised model endpoint.
* Present: “Security Boundaries in AI Infrastructure”

---

## Section 12: GPU Cluster Provisioning & Lifecycle

### 12.1 How GPU nodes are provisioned
* Explains how GPU hardware, drivers, and runtimes come together as a usable node.
* Shows why GPU nodes are not interchangeable with CPU nodes.
* Establishes provisioning as the foundation for everything above it.

### 12.2 Why GPU clusters are version-sensitive
* Explains tight coupling between drivers, CUDA, runtimes, and workloads.
* Shows how mismatches cause subtle or catastrophic failures.
* Establishes version control as a stability requirement.

### 12.3 Node pools, upgrades, and draining
* Explains how GPU nodes are grouped and managed.
* Shows why upgrades risk killing active workloads.
* Establishes careful lifecycle management as mandatory.

### 12.4 Capacity expansion vs replacement
* Explains different ways to grow or refresh GPU capacity.
* Shows trade-offs between risk, cost, and downtime.
* Establishes planning as part of scaling.

### 12.5 Why “just upgrade the cluster” is dangerous
* Explains why GPU clusters resist in-place changes.
* Shows how upgrades cascade across layers.
* Establishes caution as an operational principle.

### 12.6 Cloud Provider Constraints & GPU Reality
* Explains external limits imposed by cloud providers.
* Shows why capacity planning is constrained by availability and quotas.
* Establishes cloud constraints as a hard boundary on design.

### 12.7 Activities
* Explain why GPU clusters are more fragile than CPU clusters.
* Analyze version coupling across drivers, runtimes, and workloads.
* Design a safe GPU cluster upgrade strategy.
* Simulate capacity expansion under cloud provider constraints.
* Present: “Operating GPU Clusters in the Real World”

---

## Section 13: Experimentation, Iteration & Velocity

### 13.1 Why AI infra changes more frequently
* Explains why AI workloads evolve faster than traditional services.
* Shows how infra must adapt to constant change.
* Establishes flexibility as a core requirement.

### 13.2 Config churn and parameter exploration
* Explains frequent changes in batch sizes, precision, and runtime flags.
* Shows how small config changes have large infra effects.
* Establishes safe change management as critical.

### 13.3 Fast rollback vs slow correctness
* Explains why quick reversibility beats perfect first deployments.
* Shows how rollback speed reduces blast radius.
* Establishes rollback as a first-class capability.

### 13.4 Supporting many experiments safely
* Explains how concurrent experiments stress shared infra.
* Shows why isolation and quotas matter.
* Establishes guardrails for experimentation.

### 13.5 What developer velocity means for AI infra
* Explains velocity as time-to-feedback, not feature count.
* Shows how infra can accelerate or block iteration.
* Establishes infra as an enabler, not a bottleneck.

### 13.6 Activities
* Explain why AI infra evolves faster than traditional infra.
* Analyze the impact of configuration churn on stability.
* Design guardrails for safe experimentation at scale.
* Evaluate rollback speed versus correctness trade-offs.
* Present: “Supporting Velocity Without Chaos”

---

## Section 14: Cost Management & Capacity Planning

### 14.1 Why GPU cost dominates everything
* Explains the disproportionate cost of GPU resources.
* Shows why inefficiency is immediately visible.
* Establishes cost awareness as unavoidable.

### 14.2 Idle GPUs, fragmentation, and silent waste
* Explains common sources of unused GPU capacity.
* Shows how waste hides behind “healthy” systems.
* Establishes utilization as a cost signal.

### 14.3 Spot GPUs and preemption trade-offs
* Explains cost savings vs reliability risks.
* Shows when preemption is acceptable.
* Establishes policy-driven usage of spot capacity.

### 14.4 Capacity planning for launches and spikes
* Explains why GPU capacity cannot be spun up instantly.
* Shows how demand spikes stress planning assumptions.
* Establishes forecasting as an infra skill.

### 14.5 Cost signals AI infra teams must track
* Explains which metrics correlate with spend.
* Shows how to attribute cost to workloads.
* Establishes feedback loops between usage and cost.

### 14.6 Activities
* Explain why GPU inefficiency is immediately visible in cost.
* Analyze hidden cost drivers like fragmentation and idle time.
* Attribute cost to failed or inefficient workloads.
* Redesign infra to reduce silent GPU burn.
* Present: “Cost as a First-Class Signal”

---

## Section 15: Observability for AI Infrastructure

### 15.1 Why CPU metrics are insufficient for GPUs
* Explains why traditional infra metrics fail to describe GPU behavior.
* Shows how GPU bottlenecks remain invisible with CPU-only observability.
* Establishes the need for GPU-aware signals.

### 15.2 GPU-level metrics that actually matter
* Explains which GPU metrics correlate with real performance.
* Shows why many exposed metrics are misleading or incomplete.
* Establishes metric selection as an infra design decision.

### 15.3 Job-level vs system-level visibility
* Explains the difference between per-workload and aggregate views.
* Shows how lack of correlation obscures root causes.
* Establishes multi-level visibility as necessary for debugging.

### 15.4 Correlating infra, model, and performance signals
* Explains why GPU metrics alone are insufficient.
* Shows how infra state, model behavior, and performance interact.
* Establishes correlation as the basis for diagnosis.

### 15.5 Alerting without noise in GPU systems
* Explains why naive alerting causes fatigue in GPU-heavy environments.
* Shows how GPU behavior produces transient anomalies.
* Establishes signal quality over alert quantity.

### 15.6 Activities
* Explain why GPU observability requires different signals than CPU systems.
* Analyze misleading metrics and false alerts.
* Correlate infra, workload, and cost metrics to diagnose issues.
* Design alerting rules that avoid GPU noise.
* Present: “Observability for GPU-Based Systems”

---

## Section 16: Production Operations & Incident Response

### 16.1 Common AI infra incidents
* Explains recurring failure patterns in production AI systems.
* Shows how these differ from traditional service incidents.
* Establishes incident familiarity as preparedness.

### 16.2 Debugging slow, stuck, or failing GPU jobs
* Explains systematic approaches to diagnosing runtime issues.
* Shows how to distinguish infra faults from workload behavior.
* Establishes structured debugging over guesswork.

### 16.3 Safe restarts, retries, and recovery
* Explains when recovery actions are safe or dangerous.
* Shows how GPU workloads complicate restart semantics.
* Establishes caution in automated recovery.

### 16.4 Incident response when GPUs burn money
* Explains why time-to-mitigation matters financially.
* Shows how cost escalates during incidents.
* Establishes cost-aware incident handling.

### 16.5 Writing postmortems for AI failures
* Explains why AI incidents require different analysis.
* Shows how infra, workload, and cost intersect in failures.
* Establishes learning and prevention as outcomes.

### 16.6 Activities
* Analyze common AI infra incidents and their root causes.
* Debug a simulated slow or stuck GPU workload end-to-end.
* Design safe recovery strategies under cost pressure.
* Write a full postmortem covering infra, workload, and cost.
* Present: “Operating AI Infrastructure Under Fire”
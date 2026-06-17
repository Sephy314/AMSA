🧠 ACPA (Actor Control Plane Architecture)
Conceptual Architecture Specification (Prototype-free)
0. Abstract

ACPA (Actor Control Plane Architecture) is a control-plane-based architecture for service-oriented systems that removes the traditional model of fixed long-running service processes and instead builds the system around actor-based execution units (Labori) that are created dynamically per request.

This model structurally separates execution decision and execution execution, organizing responsibilities into layered components to reduce system complexity.

1. Design Motivation
1.1 Structural Limitations of Existing Systems

Modern microservice architectures have the following structural problems:

Services must remain constantly running
Increased network dependency between services
Larger blast radius during failures
Misalignment between execution units and deployment units
Increased operational complexity (observability / tracing / scaling)
1.2 Goals of ACPA

ACPA aims to achieve:

Dynamic creation of execution units
Centralized execution decision via control plane
Lightweight actor-based execution model
Strict separation of routing, decision, and execution
Failure containment through execution isolation
2. Core Concept

ACPA is composed of three core concepts:

2.1 Control Plane Driven Execution

In ACPA, execution never occurs directly. It always results from a control-plane decision.

Execution is always a consequence of control-plane decision, not direct request invocation.

2.2 Actor-based Execution Unit (Labori)

The minimal execution unit in ACPA is an Actor (Labori).

Labori has the following properties:

Execution unit created or assigned per request
Message-based processing structure
Independent execution context
Ephemeral or pooled lifecycle
2.3 Layered Execution Architecture

ACPA clearly separates execution into layers:

Routing Layer
Decision Layer
Execution Orchestration Layer
Execution Layer (Actor)
3. System Layers (Conceptual Model)
3.1 Edge Routing Layer (Vergilius)

Vergilius is the entry layer of the system, responsible for receiving external requests and performing initial filtering and routing.

Responsibilities:

Request ingestion
Traffic filtering (WAF-like behavior)
Load distribution
Routing to control plane entry

Characteristics:

Stateless
No identity verification responsibility
No execution logic
3.2 Control Decision Layer (Superior)

Superior is the system’s control plane responsible for execution decisions.

Responsibilities:

Request interpretation
Service mapping decision
Execution policy evaluation
Orchestration trigger decision

Characteristics:

Execution is not performed here
Pure decision-oriented layer
3.3 Execution Orchestration Layer

The Execution Orchestrator configures the execution environment and manages Actor execution.

Responsibilities:

Execution context creation
Actor lifecycle coordination
Execution scheduling
Resource allocation for execution

Characteristics:

Execution is prepared and managed, not decided
3.4 Execution Layer (Labori)

Labori is an actor-based execution unit that performs actual work.

Definition:

Labori is a lightweight execution entity that processes a single logical unit of work within an isolated execution context.

Responsibilities:

Business logic execution
Data processing (CRUD etc.)
Response generation

Characteristics:

Ephemeral or pooled lifecycle
Message-driven execution model
4. Internal Secure Execution Path (VIP Path)

VIP Path is a conceptual execution route, not a physical component.

Definition:

VIP Path represents a trusted internal execution routing channel between control and execution layers.

Meaning:

Internal trust boundary
Secure service-to-service execution flow
Controlled communication channel between orchestration and runtime layers
5. Execution Flow Model

ACPA request flow:

Request
  ↓
Edge Routing Layer (Vergilius)
  ↓
Control Decision Layer (Superior)
  ↓
Execution Orchestration Layer
  ↓
Execution Layer (Labori)
  ↓
Response
6. Architectural Properties
6.1 Separation of Concerns

ACPA separates responsibilities into:

Routing responsibility
Decision responsibility
Execution orchestration responsibility
Execution responsibility
6.2 Control-Plane Centralization

All execution flows begin based on decisions made by the control plane (Superior).

6.3 Execution Isolation

Each execution unit (Labori) runs in an independent context, ensuring failure containment.

6.4 Dynamic Execution Model

Execution units are not always running; they are created on demand per request.

7. Conceptual Comparison with MSA
Dimension	MSA	ACPA
Execution Unit	long-running service	dynamic actor (Labori)
Execution Trigger	direct request	control-plane decision
Execution Model	static deployment	dynamic lifecycle
Coupling	service-level	execution-level isolation
Runtime Structure	fixed services	generated execution contexts
8. Conceptual Positioning

ACPA sits at the intersection of:

Actor model systems (e.g., BEAM conceptual inspiration)
Control plane architectures (e.g., Kubernetes)
Serverless execution models (request-driven execution)
9. Conclusion

ACPA is a control-plane driven execution architecture where system behavior is not defined by persistent services, but by dynamically generated actor-based execution units (Labori), orchestrated through a layered decision and execution pipeline.

🔥 Core One-Liner

ACPA is not a system of always-running services, but an actor-based execution architecture where execution units are created per request and coordinated by a control plane.

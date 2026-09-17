# AI-Powered System Design Evaluator

> **Design. Simulate. Evaluate. Learn.**

An interactive system design platform that allows users to visually build distributed system architectures, simulate realistic production workloads and failures, and receive AI-powered architectural feedback backed by simulation telemetry and reference architectures.

The platform combines **real-time collaboration, event-driven microservices, graph compilation, distributed simulation, RAG, and AI agents** into a single system-design learning environment.

---

## 🚀 Overview

Traditional system design practice usually ends with drawing an architecture and receiving subjective feedback.

This project takes a different approach:

```text
Design
  ↓
Compile
  ↓
Simulate
  ↓
Measure
  ↓
Retrieve
  ↓
Evaluate
  ↓
Improve
```

Users create an architecture on a visual canvas. Once submitted, the platform converts the visual topology into an executable **Directed Acyclic Graph (DAG)**, runs synthetic workloads against it, collects performance telemetry, and uses an AI evaluation agent to analyze the design.

The AI evaluation is grounded in:

* Actual simulation metrics
* Reference architectures
* Historical failure patterns
* Architectural constraints
* Tool-generated performance data

---

# 🏗️ Architecture

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                           CLIENT / EDGE                                 │
│                                                                         │
│                    Next.js + React Flow                                 │
│                                                                         │
│              Visual Architecture Canvas                                 │
│          Real-time Collaboration / WebSockets                           │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │
                               │ WebSocket
                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         API / SESSION LAYER                              │
│                                                                         │
│                    Spring Cloud Gateway                                  │
│                         │                                               │
│                         ▼                                               │
│                       Valkey                                             │
│                                                                         │
│       Ephemeral State • Sessions • Locks • Rate Limits                  │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │
                               │ Design Submission
                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         EVENT STREAMING                                  │
│                                                                         │
│                         Apache Kafka                                     │
│                                                                         │
│  design.submissions ──→ simulation.tasks ──→ simulation.telemetry       │
└──────────────────┬──────────────────────┬───────────────────────────────┘
                   │                      │
                   ▼                      ▼
       ┌─────────────────────┐   ┌─────────────────────┐
       │  Graph Compiler     │   │ Simulation Engine   │
       │                     │   │                     │
       │ JSON → DAG          │   │ Traffic Simulation  │
       │ Schema Validation   │   │ Failure Injection   │
       │ Dependency Analysis │   │ Telemetry           │
       └──────────┬──────────┘   └──────────┬──────────┘
                  │                         │
                  └────────────┬────────────┘
                               ▼
                    ┌─────────────────────┐
                    │    Kafka Streams    │
                    │                     │
                    │ Telemetry           │
                    │ Aggregation          │
                    │ Performance Windows │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ AI Evaluation Agent │
                    │                     │
                    │ Spring AI           │
                    │ RAG                 │
                    │ Tool Calling        │
                    │ Architecture Eval   │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
             PostgreSQL +           WebSocket
               pgvector                  │
                                         ▼
                                   Frontend/User
```

---

# ✨ Key Features

### 🎨 Visual System Design

Build distributed architectures using an interactive React Flow canvas.

* Drag-and-drop architecture components
* Create connections between services
* Visualize data flow
* Save architecture configurations
* Real-time canvas updates

### 👥 Real-Time Collaboration

Multiple users can collaborate on the same architecture.

* WebSocket-based communication
* Ephemeral state synchronization
* Distributed locking
* Session management
* Low-latency updates

### ⚙️ Graph Compilation

Convert a visual React Flow topology into an executable graph.

```text
React Flow JSON
      ↓
Schema Validation
      ↓
Topology Parsing
      ↓
Dependency Analysis
      ↓
DAG
      ↓
Simulation Task
```

### 🧪 Distributed System Simulation

Test architectures against synthetic production scenarios.

Example workloads:

* Flash sales
* Traffic spikes
* Sudden traffic drops
* Database overload
* Cache failures
* Kafka consumer failures
* Service outages
* Network degradation

Metrics include:

* Throughput
* Average latency
* P95/P99 latency
* Error rate
* Queue depth
* Kafka partition lag
* Cache hit rate
* Database utilization
* Failure propagation

### 🤖 AI-Powered Evaluation

The AI agent evaluates architectures using real simulation data instead of relying exclusively on generic LLM reasoning.

The evaluation combines:

```text
Candidate Architecture
        +
Simulation Telemetry
        +
Reference Architectures
        +
Historical Failure Modes
        +
AI Reasoning
        ↓
Architectural Critique
```

### 🔎 RAG-Based Architectural Knowledge

The platform uses PostgreSQL + pgvector to retrieve relevant reference architectures and known failure patterns.

Example:

```text
Candidate:
API → Kafka → Workers → Database

        ↓ Embedding

Vector Search

        ↓

Similar:
Event-driven e-commerce architecture

Known issue:
Consumer lag during traffic spikes

        ↓

AI Evaluation
```

### 🔧 AI Tool Calling

The AI agent can retrieve actual simulation metrics through backend tools.

Example tools:

```text
getKafkaPartitionLag()
getP99Latency()
getDatabaseUtilization()
getCacheHitRate()
getFailureImpact()
```

This allows the AI to ground its feedback in measurable system behavior.

---

# 🔄 End-to-End Data Flow

## 1. User Creates Architecture

The user builds a topology using React Flow.

```text
Client
  │
  ├── API Gateway
  │       │
  │       └── Valkey
  │
  └── WebSocket
```

The canvas state is represented as JSON.

Example:

```json
{
  "nodes": [
    {
      "id": "api",
      "type": "service"
    },
    {
      "id": "redis",
      "type": "cache"
    },
    {
      "id": "db",
      "type": "database"
    }
  ],
  "edges": [
    {
      "source": "api",
      "target": "redis"
    },
    {
      "source": "api",
      "target": "db"
    }
  ]
}
```

---

## 2. Submit Design

When the user finalizes the architecture:

```text
Frontend
    ↓
WebSocket
    ↓
Spring Cloud Gateway
    ↓
Kafka
```

The topology is published to:

```text
design.submissions
```

Kafka provides durable asynchronous communication between services.

---

## 3. Compile Graph

The Graph Compiler consumes the submission.

Responsibilities:

* Validate schema
* Validate node types
* Validate connections
* Detect invalid dependencies
* Construct the DAG
* Enrich nodes with simulation metadata

```text
Raw JSON
   ↓
Validation
   ↓
Graph Construction
   ↓
Dependency Resolution
   ↓
Executable DAG
```

The resulting task is published to:

```text
simulation.tasks
```

---

## 4. Run Simulation

The Simulation Engine consumes the compiled DAG.

It executes synthetic scenarios such as:

```text
Normal Traffic
      ↓
Traffic Spike
      ↓
Service Failure
      ↓
Database Overload
      ↓
Recovery
```

The engine generates telemetry continuously.

```text
Simulation Engine
        ↓
simulation.telemetry
        ↓
Kafka
```

---

## 5. Aggregate Telemetry

Raw telemetry can contain millions of events.

Kafka Streams aggregates these events into meaningful performance windows.

Example:

```text
10-second window

Requests:        100,000
Throughput:      10,000 req/sec
Avg Latency:     83 ms
P99 Latency:     240 ms
Error Rate:      1.7%
DB Utilization:  91%
Kafka Lag:       12,000
```

This prevents the AI agent from having to process every individual telemetry event.

---

## 6. Retrieve Knowledge

The AI Evaluation Agent converts the architecture into a semantic representation.

Example:

```text
Compute:
- API service
- Worker service

Messaging:
- Kafka

Storage:
- PostgreSQL

Flow:
API → Kafka → Worker → PostgreSQL

Constraints:
- High throughput
- Asynchronous processing
```

This representation is embedded and searched against the pgvector knowledge base.

---

## 7. AI Evaluation

The AI agent combines retrieved knowledge with actual simulation results.

```text
                    ┌──────────────────┐
                    │ Candidate Design │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Semantic Graph   │
                    │ Representation   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Vector Retrieval │
                    │    pgvector      │
                    └────────┬─────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
    Reference Architectures          Failure Patterns
              │                             │
              └──────────────┬──────────────┘
                             ▼
                    ┌──────────────────┐
                    │   AI Agent       │
                    │   Spring AI      │
                    └────────┬─────────┘
                             │
                       Tool Calling
                             │
                             ▼
                    Simulation Metrics
                             │
                             ▼
                    Architectural Critique
```

---

# 🧠 AI Evaluation Example

Suppose a candidate creates:

```text
                    ┌─────────┐
                    │  Users  │
                    └────┬────┘
                         ▼
                  ┌─────────────┐
                  │ Load        │
                  │ Balancer    │
                  └──────┬──────┘
                         ▼
                  ┌─────────────┐
                  │ API Servers │
                  └──┬──────┬───┘
                     │      │
                     ▼      ▼
                  Redis   PostgreSQL
```

The simulator may discover:

```text
Traffic:
100,000 req/sec

P99 latency:
850 ms

Database utilization:
98%

Cache hit rate:
42%
```

The AI can then generate feedback such as:

```text
Primary bottleneck:
PostgreSQL saturation.

Evidence:
Database utilization reached 98% while P99 latency
increased to 850ms during the flash-sale workload.

Potential improvements:
- Introduce read replicas
- Add database partitioning where appropriate
- Review cache strategy
- Consider asynchronous processing for non-critical writes

Failure observation:
The architecture lacks an explicit fallback strategy
for cache unavailability.
```

The important distinction is that the feedback is based on **observed simulation behavior**, not merely an LLM's generic opinion.

---

# 🗄️ Data Storage

The system deliberately uses different storage technologies for different workloads.

## PostgreSQL

Used for structured and durable information.

Stores:

```text
Users
Submissions
Submission Metadata
Evaluation Results
Grading Rubrics
Reference Architectures
```

### pgvector

Used for semantic retrieval.

```text
Reference Architecture
        ↓
Embedding
        ↓
pgvector
        ↓
Similarity Search
```

---

## MongoDB

Used for flexible document-oriented data.

Stores:

```text
React Flow Canvas JSON
Historical Topologies
Simulation Logs
Variable Node Configurations
```

MongoDB is useful because architecture configurations can vary significantly between submissions.

---

## Valkey

Used for high-speed ephemeral data.

```text
Session State
Canvas State
Distributed Locks
Rate Limiting
Deduplication
```

Valkey is not intended to be the primary persistent datastore.

---

# 📨 Kafka Topics

| Topic                  | Producer          | Consumer           | Purpose                  |
| ---------------------- | ----------------- | ------------------ | ------------------------ |
| `design.submissions`   | Gateway           | Graph Compiler     | Raw topology submissions |
| `simulation.tasks`     | Graph Compiler    | Simulation Engine  | Compiled DAG tasks       |
| `simulation.telemetry` | Simulation Engine | Kafka Streams / AI | Simulation metrics       |
| Compacted topics       | Services          | Materialized views | Rebuildable state        |

Kafka partitions provide scalable parallel processing while partition keys can preserve ordering where required.

---

# 🧩 Backend Services

## API Gateway

**Technology:** Spring Cloud Gateway

Responsibilities:

* WebSocket routing
* Authentication
* Rate limiting
* Request routing
* Connection management

---

## Canvas Session Service

Responsibilities:

* Collaborative sessions
* Canvas synchronization
* Valkey interaction
* Distributed locking
* Session lifecycle

---

## Graph Compiler Service

**Technology:** Spring Boot

Responsibilities:

* JSON validation
* Topology parsing
* DAG construction
* Dependency analysis
* Simulation task creation

---

## Simulation Engine

**Technology:** Spring Boot

Responsibilities:

* Execute compiled DAGs
* Generate workloads
* Inject failures
* Measure system behavior
* Produce telemetry

---

## Telemetry Aggregator

**Technology:** Kafka Streams

Responsibilities:

* Consume telemetry
* Window metrics
* Aggregate performance data
* Detect simulation completion
* Trigger evaluation

---

## AI Evaluation Agent

**Technology:** Spring AI

Responsibilities:

* Architecture interpretation
* RAG retrieval
* Reference architecture comparison
* Tool calling
* Simulation metric analysis
* Generate architectural feedback

---

# 🔐 Reliability & Scalability

The architecture is designed around several distributed-system principles.

### Asynchronous Processing

Long-running simulations do not block the user's request.

```text
Request
  ↓
Kafka
  ↓
Async Processing
```

### Horizontal Scaling

Services can scale independently.

```text
              simulation.tasks
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   Simulator 1  Simulator 2  Simulator 3
```

### Fault Isolation

A temporary failure in the AI layer doesn't require the entire simulation pipeline to restart.

Kafka provides durable event storage so consumers can recover.

### Idempotency

Submission IDs and event IDs can be used to prevent duplicate processing.

```text
submission_id
event_id
simulation_id
```

Valkey can additionally maintain short-lived deduplication state.

### Backpressure

Kafka naturally buffers workloads between producers and consumers.

If the simulation engine becomes overloaded:

```text
Producer
   ↓
Kafka
   ↓
Queue
   ↓
Simulation Workers
```

rather than overwhelming downstream services.

---

# 📈 Scalability Model

The most computationally expensive component is expected to be the simulation layer.

Therefore, simulation workers can be scaled horizontally:

```text
                         Kafka
                           │
                  simulation.tasks
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
    Simulation-1     Simulation-2     Simulation-3
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                  simulation.telemetry
```

This allows the system to process many independent design simulations concurrently.

---

# 🛠️ Technology Stack

### Frontend

* Next.js
* React
* React Flow
* WebSockets

### Backend

* Java
* Spring Boot
* Spring Cloud Gateway
* Spring AI
* Kafka Streams

### Messaging

* Apache Kafka

### Databases

* PostgreSQL
* pgvector
* MongoDB
* Valkey

### AI

* LLM
* RAG
* Vector embeddings
* Tool calling
* LangGraph-style state routing

---

# 📁 Suggested Repository Structure

```text
system-design-evaluator/
│
├── frontend/
│   ├── components/
│   ├── canvas/
│   ├── hooks/
│   ├── services/
│   └── package.json
│
├── services/
│   │
│   ├── api-gateway/
│   │
│   ├── canvas-session/
│   │
│   ├── graph-compiler/
│   │
│   ├── simulation-engine/
│   │
│   ├── telemetry-aggregator/
│   │
│   └── ai-evaluator/
│
├── infrastructure/
│   ├── docker/
│   ├── kafka/
│   ├── postgres/
│   ├── mongodb/
│   └── valkey/
│
├── knowledge-base/
│   ├── architectures/
│   ├── failure-patterns/
│   └── embeddings/
│
├── docs/
│   ├── architecture.md
│   ├── api.md
│   └── simulation.md
│
├── docker-compose.yml
└── README.md
```

---

# 🚦 Getting Started

## Prerequisites

Make sure the following are installed:

```text
Java 17+
Node.js 20+
Docker
Docker Compose
Maven
```

---

## Start Infrastructure

```bash
docker compose up -d
```

This starts the infrastructure dependencies:

```text
Kafka
PostgreSQL
MongoDB
Valkey
```

---

## Start Backend Services

Each Spring Boot service can be started independently.

```bash
cd services/api-gateway
./mvnw spring-boot:run
```

Similarly:

```bash
cd services/graph-compiler
./mvnw spring-boot:run
```

```bash
cd services/simulation-engine
./mvnw spring-boot:run
```

```bash
cd services/ai-evaluator
./mvnw spring-boot:run
```

---

## Start Frontend

```bash
cd frontend
npm install
npm run dev
```

The application can then be accessed through the local development server.

---

# 🧪 Example Evaluation Flow

```text
User
 │
 │ Creates architecture
 ▼
React Flow
 │
 │ WebSocket
 ▼
Spring Cloud Gateway
 │
 ▼
Kafka
 │
 │ design.submissions
 ▼
Graph Compiler
 │
 │ simulation.tasks
 ▼
Simulation Engine
 │
 │ simulation.telemetry
 ▼
Kafka Streams
 │
 │ Aggregated Metrics
 ▼
AI Evaluation Agent
 │
 ├──────→ pgvector
 │
 └──────→ Simulation Tools
 │
 ▼
Architectural Critique
 │
 │ WebSocket
 ▼
User
```

---

# 🎯 Design Goals

The project is designed around four primary goals:

### 1. Make system design interactive

Instead of writing architecture diagrams manually, users construct systems visually.

### 2. Make evaluation measurable

Architectures are tested using simulated workloads and failure scenarios.

### 3. Make AI feedback grounded

The AI receives actual metrics and retrieves relevant architectural knowledge.

### 4. Make the platform extensible

New:

* Architecture components
* Simulation scenarios
* Failure modes
* Reference architectures
* Evaluation criteria
* AI tools

can be added without redesigning the entire pipeline.

---

# 🔮 Future Enhancements

Potential future extensions include:

* Multi-user collaborative editing
* More realistic network simulation
* Kubernetes-based simulation workers
* Custom workload definitions
* Cost estimation
* Cloud architecture mapping
* AWS/Azure/GCP component libraries
* Architecture security analysis
* Automated capacity planning
* Chaos engineering scenarios
* Adaptive AI-generated test cases
* Personalized system-design difficulty
* Interview-mode evaluation
* Architecture versioning
* Leaderboards and benchmarking
* Automated architecture optimization

---

# 💡 Core Architectural Principle

The central principle behind the platform is:

```text
Don't just ask whether an architecture looks good.

Build it.
Compile it.
Stress it.
Measure it.
Compare it.
Then explain why it works or fails.
```

---

## 📜 License

This project is currently intended as a learning and experimental system-design platform.

Add the appropriate license before distributing the project publicly.

---

## 👨‍💻 Author

**Kumar Shresth**

Built as an exploration of:

**Distributed Systems × Event-Driven Architecture × Simulation × RAG × AI Agents**

---

# FlowForge

## 1. Overview

FlowForge is a developer-focused **distributed workflow orchestration platform** that enables applications to define and execute complex, multi-step business processes as configurable workflows instead of hardcoding orchestration logic directly into the application codebase.

Developers can visually or programmatically define workflows consisting of actions, conditions, delays, API calls, and other tasks, and trigger them through events such as REST requests, scheduled events, or messaging systems. FlowForge then manages the complete execution lifecycle, including state persistence, conditional branching, retries, timeouts, failure handling, and recovery from system or worker failures.

The platform is designed to keep an application's core business logic independent from its workflow orchestration logic while providing reliable and observable execution of long-running workflows across distributed workers.


## 2. Problem Statement

As applications grow, business processes often evolve from simple operations into complex, multi-step workflows involving conditions, retries, delays, external services, and failure handling. Developers commonly implement this orchestration logic directly within the application's codebase.

Over time, this can lead to tightly coupled and difficult-to-maintain code, where even a small change in a business process requires modifying and redeploying the core application. Handling failures, retries, long-running operations, and recovery from system failures also adds significant complexity to the application.

There is a need for a reliable and flexible system that separates workflow orchestration from core application logic while allowing developers to define, modify, monitor, and execute complex workflows without repeatedly implementing the orchestration logic from scratch.


## 3. Proposed Solution

FlowForge provides a centralized workflow orchestration layer that separates complex business-process orchestration from an application's core codebase.

Developers can define workflows as configurable graphs of interconnected nodes, where each node represents a generic operation such as an HTTP request, condition, delay, database operation, notification, or approval. Workflows can be created through the visual workflow builder or programmatically through REST APIs.

When an application generates a configured event, FlowForge creates a workflow execution and executes the required nodes through its distributed execution engine. The platform persists execution state, manages dependencies and conditional branching, and coordinates tasks across distributed workers.

FlowForge also provides built-in reliability mechanisms such as configurable retry policies, timeouts, failure handling, compensation, idempotency, and execution recovery. This allows workflows to continue from their persisted state after worker or system failures without requiring developers to implement these orchestration and recovery mechanisms repeatedly within their applications.


## 4. Goals & Objectives

* **Decouple orchestration:** Keep complex workflow logic separate from the application's core business logic.
* **Reliable execution:** Ensure workflows can handle failures, retries, delays, and unexpected worker or system failures.
* **Flexible workflows:** Support conditional, sequential, parallel, and long-running workflow execution.
* **Scalable execution:** Execute workflows across multiple distributed workers and scale execution independently.
* **Extensibility:** Provide a generic execution framework that can support new node types, triggers, and integrations.
* **Observability:** Provide clear visibility into workflow executions, task states, failures, retries, and execution history.
* **Developer-focused experience:** Make workflows easy to define, integrate, version, monitor, and manage through APIs and a visual builder.



## 5. Key Features

* **Visual Workflow Builder** — Create workflows using a drag-and-drop interface.
* **Workflow APIs** — Create, update, publish, and manage workflows programmatically.
* **Multiple Triggers** — Start workflows through REST/webhooks, schedules, manual execution, and event streams.
* **Generic Node System** — Support reusable nodes such as HTTP requests, conditions, delays, database operations, notifications, transformations, and approvals.
* **Conditional & Parallel Execution** — Support branching and parallel workflow paths.
* **Workflow Versioning** — Maintain immutable published versions while allowing new versions to be developed independently.
* **Configurable Retries** — Define retry attempts, delays, and backoff strategies per task.
* **Failure Handling & Compensation** — Stop execution, follow failure paths, or execute compensation actions after failures.
* **Long-Running Workflows** — Support workflows that remain active across delays and extended execution periods.
* **Fault Recovery** — Persist execution state and recover interrupted workflows after worker or system failures.
* **Distributed Workers** — Execute workflow tasks across multiple workers for scalability.
* **Execution Monitoring** — Track workflow and task status, attempts, failures, outputs, and execution history.
* **Multi-Tenant Security** — Isolate organizations, users, workflows, executions, and credentials.
* **Secure Credential Management** — Store and access integration credentials through a dedicated credential system without exposing secrets in workflows or logs.




## 6. System Architecture

FlowForge follows a distributed architecture in which workflow management, execution coordination, task processing, and persistence are separated into distinct components.

```text
                         ┌──────────────────────┐
                         │   React Web Client   │
                         │  Visual Workflow UI  │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    FlowForge API     │
                         │     Spring Boot      │
                         └──────────┬───────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
        ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
        │ Workflow       │ │ Execution      │ │ Authentication │
        │ Management     │ │ Engine         │ │ & Authorization│
        └───────┬────────┘ └───────┬────────┘ └────────────────┘
                │                  │
                └──────────┬───────┘
                           ▼
                    ┌───────────────┐
                    │  PostgreSQL   │
                    │ Persistent    │
                    │ State & Data  │
                    └───────────────┘
                           ▲
                           │
                    ┌──────┴───────┐
                    │    Kafka     │
                    │ Task/Event   │
                    │   Broker     │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        ┌─────────┐   ┌─────────┐   ┌─────────┐
        │ Worker  │   │ Worker  │   │ Worker  │
        │    1    │   │    2    │   │    3    │
        └────┬────┘   └────┬────┘   └────┬────┘
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                  External Services
```

### Core Components

* **React Web Client:** Provides the visual workflow builder and execution monitoring interface.
* **FlowForge API:** Exposes REST APIs for workflow management, execution, authentication, and integration.
* **Workflow Management:** Handles workflow definitions, nodes, connections, and versions.
* **Execution Engine:** Determines workflow state, schedules tasks, evaluates conditions, and coordinates execution.
* **Kafka:** Provides asynchronous task and event communication between the execution engine and workers.
* **Workers:** Execute individual workflow tasks and report their results.
* **PostgreSQL:** Stores workflow definitions, versions, execution state, task state, users, organizations, and audit data.
* **Authentication & Authorization:** Controls access to organizations, workflows, executions, and credentials.



## 7. How the System Works

1. **Define Workflow**
   A developer creates a workflow using the visual builder or REST API and publishes a workflow version.

2. **Receive Trigger**
   The workflow is triggered through a REST request, webhook, schedule, manual execution, or supported event source.

3. **Create Execution**
   FlowForge creates a unique execution instance associated with the published workflow version and persists its initial state.

4. **Schedule Tasks**
   The execution engine determines the next executable node and publishes the task to Kafka.

5. **Execute Task**
   An available worker consumes the task and executes the corresponding generic node executor.

6. **Persist Result**
   The worker reports the result, and FlowForge persists the node state, output, attempt count, and execution status.

7. **Determine Next Step**
   The execution engine evaluates the workflow graph and determines the next node or branch based on the result.

8. **Handle Failures**
   Failed tasks follow their configured retry policy. After retries are exhausted, the workflow can stop, follow a failure path, or execute configured compensation actions.

9. **Recover Interrupted Executions**
   If a worker or system fails, persisted execution state allows FlowForge to identify incomplete tasks and safely resume execution.

10. **Complete Execution**
    Once all required nodes finish successfully, the execution is marked as `COMPLETED` and its execution history remains available for monitoring and auditing.



## 8. Technology Stack

| Layer                  | Technology          |
| ---------------------- | ------------------- |
| Backend                | Java, Spring Boot   |
| API                    | REST                |
| Frontend               | React               |
| Database               | PostgreSQL          |
| Messaging              | Apache Kafka        |
| Caching & Coordination | Redis               |
| Build Tool             | Maven               |
| Containerization       | Docker              |
| Orchestration          | Kubernetes          |
| Monitoring             | Prometheus, Grafana |
| Version Control        | Git, GitHub         |



## 9. Project Structure

FlowForge follows a **feature-based package structure**, where code is organized around business capabilities rather than technical layers. This keeps related functionality together and makes the system easier to maintain and extend as the platform grows.

```text
src/
└── main/
    └── java/
        └── com/
            └── flowforge/
                ├── workflow/
                │   ├── controller/
                │   ├── service/
                │   ├── repository/
                │   ├── entity/
                │   └── dto/
                │
                ├── execution/
                │   ├── controller/
                │   ├── service/
                │   ├── repository/
                │   ├── entity/
                │   └── dto/
                │
                ├── trigger/
                │   ├── rest/
                │   ├── schedule/
                │   ├── manual/
                │   └── kafka/
                │
                ├── worker/
                │   ├── consumer/
                │   ├── executor/
                │   ├── heartbeat/
                │   └── model/
                │
                ├── credential/
                │   ├── controller/
                │   ├── service/
                │   ├── repository/
                │   ├── entity/
                │   └── encryption/
                │
                ├── authentication/
                │   ├── controller/
                │   ├── service/
                │   ├── security/
                │   ├── entity/
                │   └── dto/
                │
                ├── organization/
                │   ├── controller/
                │   ├── service/
                │   ├── repository/
                │   ├── entity/
                │   └── dto/
                │
                └── common/
                    ├── exception/
                    ├── response/
                    ├── validation/
                    ├── util/
                    └── constant/
```

### Module Responsibilities

| Module           | Responsibility                                                         |
| ---------------- | ---------------------------------------------------------------------- |
| `workflow`       | Workflow definitions, nodes, edges, validation, and versioning         |
| `execution`      | Workflow execution, state management, retries, branching, and recovery |
| `trigger`        | REST, scheduled, manual, and Kafka-based workflow triggers             |
| `worker`         | Task consumption, node execution, result reporting, and worker health  |
| `credential`     | Secure credential storage, encryption, access, and auditing            |
| `authentication` | User authentication and authorization                                  |
| `organization`   | Organizations, members, roles, and tenant isolation                    |
| `common`         | Shared exceptions, utilities, validation, responses, and constants     |



## 10. Database Design

FlowForge uses **PostgreSQL** as its primary relational database and follows a domain-oriented design that separates workflow definitions, execution state, organizational data, and security-related information. The database is designed to support multi-tenancy, workflow versioning, distributed execution, failure recovery, and execution tracking while maintaining clear relationships between the different components of the platform.

```text
Organization
    │
    ├── Users / Members
    ├── API Keys
    ├── Credentials
    └── Workflows
          │
          ├── Workflow Triggers
          └── Workflow Versions
                │
                ├── Workflow Nodes
                └── Workflow Edges
                      │
                      ▼
               Workflow Executions
                      │
                      ▼
                Node Executions

Workers
    │
    └── Execute workflow tasks

Audit Logs
    │
    └── Track important system and security actions
```

### Entity Responsibilities

| Entity                 | Responsibility                                                                            |
| ---------------------- | ----------------------------------------------------------------------------------------- |
| `organizations`        | Stores organizations using FlowForge and provides the tenant boundary for their resources |
| `users`                | Stores FlowForge user accounts and authentication-related information                     |
| `organization_members` | Associates users with organizations and defines their roles and permissions               |
| `api_keys`             | Stores securely hashed API keys used by external applications to access FlowForge         |
| `workflows`            | Represents the logical workflow and its metadata                                          |
| `workflow_versions`    | Maintains immutable versions of workflow definitions                                      |
| `workflow_triggers`    | Defines how and when a workflow can be started                                            |
| `workflow_nodes`       | Stores the individual nodes belonging to a workflow version                               |
| `workflow_edges`       | Defines the connections and execution paths between workflow nodes                        |
| `workflow_executions`  | Represents an individual execution of a specific workflow version                         |
| `node_executions`      | Tracks the runtime state, attempts, results, and failures of individual nodes             |
| `credentials`          | Stores encrypted credentials required by workflow integrations                            |
| `workers`              | Tracks distributed worker instances and their health status                               |
| `audit_logs`           | Records important security, administrative, and workflow-related actions                  |


## 11. API Documentation


FlowForge exposes a versioned REST API for managing workflows, executions, triggers, credentials, organizations, and other platform resources. The API follows standard HTTP methods and returns consistent JSON responses. Protected APIs use JWT-based authentication, while application-to-FlowForge integrations can use organization-specific API keys.

### API Base URL

```text
/api/v1


| Resource    | Method   | Endpoint                                             | Purpose                       |
| ----------- | -------- | ---------------------------------------------------- | ----------------------------- |
| Workflows   | `POST`   | `/workflows`                                         | Create a workflow             |
| Workflows   | `GET`    | `/workflows`                                         | List workflows                |
| Workflows   | `GET`    | `/workflows/{workflowId}`                            | Get workflow details          |
| Workflows   | `PUT`    | `/workflows/{workflowId}`                            | Update a workflow             |
| Workflows   | `DELETE` | `/workflows/{workflowId}`                            | Delete a workflow             |
| Versions    | `POST`   | `/workflows/{workflowId}/versions`                   | Create a new workflow version |
| Versions    | `GET`    | `/workflows/{workflowId}/versions`                   | List workflow versions        |
| Versions    | `GET`    | `/workflows/{workflowId}/versions/{version}`         | Get a specific version        |
| Versions    | `POST`   | `/workflows/{workflowId}/versions/{version}/publish` | Publish a workflow version    |
| Executions  | `POST`   | `/workflows/{workflowId}/execute`                    | Start a workflow execution    |
| Executions  | `GET`    | `/executions/{executionId}`                          | Get execution status          |
| Executions  | `GET`    | `/executions`                                        | List executions               |
| Executions  | `POST`   | `/executions/{executionId}/cancel`                   | Cancel an execution           |
| Triggers    | `POST`   | `/workflows/{workflowId}/triggers`                   | Create a workflow trigger     |
| Triggers    | `GET`    | `/workflows/{workflowId}/triggers`                   | List workflow triggers        |
| Triggers    | `PUT`    | `/triggers/{triggerId}`                              | Update a trigger              |
| Triggers    | `DELETE` | `/triggers/{triggerId}`                              | Delete a trigger              |
| Credentials | `POST`   | `/credentials`                                       | Create a credential           |
| Credentials | `GET`    | `/credentials`                                       | List credentials              |
| Credentials | `GET`    | `/credentials/{credentialId}`                        | Get credential metadata       |
| Credentials | `PUT`    | `/credentials/{credentialId}`                        | Update a credential           |
| Credentials | `DELETE` | `/credentials/{credentialId}`                        | Delete a credential           |
```


## 12. Authentication & Authorization

FlowForge uses **JWT-based authentication** for user access and **API keys** for application-to-FlowForge integrations. Authorization is managed through role-based access control (RBAC), while organization-level isolation ensures that users can access only the resources belonging to their organization.

### Authentication

- **User Authentication:** Users authenticate using their credentials and receive a JWT access token.
- **API Authentication:** External applications use organization-specific API keys to interact with FlowForge.
- **Password Security:** User passwords are stored only as secure hashes and are never persisted in plain text.

### Authorization

FlowForge uses role-based access control with the following initial roles:

| Role | Permissions |
|---|---|
| `ADMIN` | Manage organization members, workflows, credentials, and platform settings |
| `DEVELOPER` | Create, modify, publish, execute, and monitor workflows |
| `VIEWER` | View workflows and execution history without modification access |

### Multi-Tenant Access Control

Every organization-owned resource is associated with an organization. Before processing a request, FlowForge verifies both the user's identity and their membership and permissions within the requested organization.

```text
Request
   ↓
Authentication
   ↓
Identify User & Organization
   ↓
Check Role & Permission
   ↓
Access Resource
```


## 13. Error Handling

FlowForge uses a consistent error-handling strategy across APIs, workflow execution, and distributed task processing. Errors are classified, persisted, and handled according to their type and the configuration of the affected workflow or node.

### API Errors

REST APIs return standardized error responses containing an error code, message, and relevant details.

```json
{
  "error": {
    "code": "WORKFLOW_NOT_FOUND",
    "message": "Workflow does not exist"
  }
}
```
### Workflow Errors

Node-level failures are recorded against the corresponding execution. Based on the configured retry policy, FlowForge can retry the task, follow a failure path, execute compensation actions, or mark the workflow as failed.

### System Failures

Worker crashes, service interruptions, and unexpected failures are handled through persisted execution state and task recovery mechanisms. Interrupted tasks can be detected and reprocessed without restarting the entire workflow.

### Error Logging

Errors are logged with sufficient context for debugging and monitoring while preventing sensitive information such as credentials, API keys, and other secrets from being exposed in logs.





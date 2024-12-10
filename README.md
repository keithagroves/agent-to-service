# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow) ![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg) [![Discord](https://img.shields.io/badge/Discord-A2S_PROTOCOL-blue?logo=discord&logoColor=white)](https://discord.gg/mMfxvMtHyS)

The **Agent-to-Service Protocol (A2S)** provides a standardized framework for AI agents to dynamically discover, understand, and interact with arbitrary services at runtime. It enables agents to go beyond their pre-programmed functionalities by discovering new capabilities, orchestrating external APIs, and handling complex workflows—all while ensuring structured, auditable, and secure operations.

## Overview

A2S is designed for scenarios where AI agents must operate in dynamic environments, discovering and invoking new services as they become available. By defining capabilities, parameters, tasks, and flow controls, A2S standardizes how agents interface with services and compose functionalities. 

**Example Scenario:**

```
┌────────────────── A2S Chat Session ──────────────────┐

👤 [USER] > Check the weather for my picnic in Central Park and tweet it.

🤖 [AGENT] > Discovered capability: WeatherUpdateCapability
             Services: api.weather.com, api.twitter.com

   [AGENT] > Proposed plan:
             1. Retrieve the weather forecast
             2. Post a weather update on Twitter

   [AGENT] > Do you want a specific message for the tweet?

👤 [USER] > Yes, mention it's for a weekend picnic.

🤖 [AGENT] > Tweet posted:
             "Weekend picnic weather update for Central Park:
              Sunny with light clouds, high of 75°F.
              Perfect picnic weather! 🧺☀️"

└────────────────────────────────────────────────────────┘
```

In this example, the agent dynamically identifies relevant capabilities, executes tasks in sequence, and completes the user's request without prior hardcoding of the involved services.

## Core Features

- **Agent-First Design:** Tailored for AI agents to discover and orchestrate capabilities programmatically.
- **Dynamic Discovery:** Agents can query registries for new capabilities and incorporate them at runtime.
- **Atomic Operations:** Each API operation is modeled as a discrete, auditable, and reliable request.
- **Secure Parameter Management:** Strong controls over input/output parameters, data lifecycles, and credential handling.
- **Flow Control:** Built-in constructs for conditional logic, looping, parallelism, and error handling.
- **Auditing & Permissions:** Integrated auditing, security levels, and permissions to ensure trust and compliance.
- **Composability:** Simple atomic capabilities can be composed into aggregate capabilities for complex workflows.

## Core Concepts

### Capabilities

A **capability** represents a unit of functionality—either a simple (atomic) operation or a complex (aggregate) workflow. Capabilities are defined in YAML or JSON metadata, which includes versioning, authorship, security posture, and dependencies.

**Example Capability Metadata:**

```yaml
a2s: 1.0.0            # Protocol version (required)
id: "CapabilityID"
description: "Handles weather updates"
version: 1.0.0
authors:
  - name: "Author Name"
type: "aggregate"     # aggregate or atomic
checksum: "<sha256>"  # SHA-256 integrity check
last_updated: Date
tags: []
security:
  audit:
    status: "audited" # audited | unaudited | in-progress
    provider: "SecurityFirm Inc."
    id: "AUDIT-2024-001"
    url: "https://security.example.com/audits/AUDIT-2024-001"
  permissions:
    level: "elevated" # basic | elevated | admin
    description: "May access user location"
    capabilities:
      - "capability_name"
```

### Capability Types

**Atomic Capabilities:**  
- Self-contained, minimal functionality.  
- No external imports.  
- Serve as building blocks, e.g., making a single API request.

**Aggregate Capabilities:**  
- Can import and orchestrate multiple atomic capabilities.  
- Cannot import other aggregates (to avoid overly complex dependency chains).  
- Ideal for multi-step workflows, e.g., retrieving data and then transforming or posting it elsewhere.

### Dependencies and Registries

Capabilities declare their dependencies and their source registries. Agents can query registries to find and integrate capabilities:

```yaml
registries:
  default: "https://registry.a2s.dev"
  custom: "https://custom-registry.example.com"

dependencies:
  weather:
    namespace: "core/weather"
    id: "WeatherCapability"
    version: "^1.0.0"
    checksum: "sha256:abc123..."
    registry: registries.custom
```


### Parameter Management

A2S strictly defines parameter handling to ensure data integrity, clarity, and security:

**Parameter Scope and Lifecycle:**
- **Inputs/Outputs at Capability Level:** Required at capability start (inputs) and produced at completion (outputs).
- **Inputs/Outputs at Task Level:** Required and produced at each step within a capability.
- **Data Lifecycles:**  
  - **Persistent:** Data persists beyond capability execution.  
  - **Session:** Data available for the user's ongoing session.  
  - **Capability:** Data valid only during capability execution.  
  - **Task:** Data scoped solely to a single task.

### Parameter References

A2S supports multiple referencing modes to streamline data access:

1. **Schema References:**  
   Link directly to schema definitions:
   ```yaml
   inputs:
     user_type:
       $ref: "#/schemas/UserType"
   ```

2. **Value References:**  
   Dynamically reference outputs of other tasks, inputs, or service credentials:
   ```yaml
   mappings:
     temperature: {getWeather.outputs.temperature}
     userId: {inputs.user.id}
     token: {services.weather-api.auth.token}
     version: {capability.version}
   ```

3. **String Templates:**  
   Inline references within strings:
   ```yaml
   message: "Current weather in {inputs.city}: {getWeather.outputs.conditions}"
   ```

4. **Conditional References:**  
   Control flow based on dynamic conditions:
   ```yaml
   condition:
     if: {decideToPost.outputs.shouldPost}
   ```

### Service Integration

Define how capabilities interact with external APIs, including credential management and rate limits:

```yaml
services:
  api.weather.com:
    credentials_url: "https://weather.com/api/signup"
    storage:
      oauth2:
        read:
          - client_id
          - client_secret
        write:
          - access_token
          - refresh_token
    rate_limits:
      requests_per_second: 10
      burst: 20
```

**Suggestions for Improvement:**  
- Standardize authentication methods (OAuth2, API keys, etc.) through schemas.
- Introduce optional fields for Service-Level Agreements (SLAs) or expected latency bounds.

### Flow Control

A2S supports a rich set of flow control constructs for orchestrating tasks:

```yaml
flow:
  steps:
    # Sequential
    - task: getWeather

    # Conditional Branch
    - if: {weather.outputs.temperature} > 25
      task: sendAlert
    else:
      task: logNormal

    # Loops
    - repeat: 3
      task: retryOperation
    
    - while: {status.outputs.pending}
      task: checkStatus
      timeout: 300000 # 5 minutes

    # Parallel Execution
    - parallel:
        - task: task1
        - task: task2
      maxConcurrent: 2

    # Error Handling
    - try:
        task: riskyOperation
      catch:
        task: handleError
        onError:
          - RATE_LIMIT
          - TIMEOUT
```

### Tasks

Tasks are fundamental units of work within a capability. They can involve making requests, evaluating conditions, composing prompts, or invoking other capabilities.

**Example Task:**

```yaml
tasks:
  - id: getWeatherTask
    type: request
    service: "api.example.com"
    request: "#/requests/getWeatherRequest"
    input:
      mappings:
        cityId: {inputs.cityId}
        apiKey: {services.weather.auth.apiKey}
    output:
      mappings:
        temperature: response.temperature
        conditions: response.conditions
    error:
      mappings:
        message: error.message
        code: error.code
```

**Suggestions for Improvement:**  
- Introduce standardized error codes and categories for uniform handling.
- Consider task annotations for debugging and tracing.

### A2S Registry

A2S registries are centralized repositories that store and organize capabilities. Agents can query these registries using semantic search, enabling them to find capabilities that match user intentions or required functionalities.

**Visual Overview:**

```
User Query
    ↓
Agent → Registry (semantic search) → Capabilities
    ↓
       Execute best-matching capabilities to fulfill the request
```

**Suggestions for Improvement:**  
- Offer advanced search filters (e.g., domain, author, security level).
- Implement rating or reliability scores for capabilities.

## Best Practices

- **Security & Auditing:**  
  Always specify audit status, required permissions, and ensure data is securely managed.
  
- **Service Integration:**  
  One distinct service per integration. Document credentials, rate limits, and authentication explicitly.
  
- **Error Handling:**  
  Define fallback strategies, provide meaningful error messages, and handle timeouts, rate limits, and unexpected responses gracefully.
  
- **Capability Design:**  
  Keep atomic capabilities focused. Use aggregates to combine multiple steps. Validate checksums and dependencies rigorously.

## SDK Usage

A2S provides SDKs for simplifying integration:

```typescript
import { A2SRegistry, A2SAgent } from '@a2s/core';

const registry = new A2SRegistry();
const agent = new A2SAgent();

const capabilities = registry.findCapability("User query");

async function executeCapability(capability) {
  if (!agent.verifyChecksum(capability)) return;
  
  const parameterStore = new ParameterStore();
  await agent.resolveParameters(capability, parameterStore);
  
  const executor = new CapabilityExecutor(parameterStore);
  return await executor.execute(capability);
}
```

## License

This project is licensed under the [MIT License](LICENSE).


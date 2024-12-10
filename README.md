# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow) ![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg) [![Discord](https://img.shields.io/badge/Discord-A2S_PROTOCOL-blue?logo=discord&logoColor=white)](https://discord.gg/mMfxvMtHyS)

The **Agent-to-Service Protocol (A2S)** provides a standardized framework for AI agents to dynamically discover, understand, and interact with services at runtime. It enables agents to transcend their initial programming by discovering new capabilities, orchestrating external APIs, and handling complex workflows—while ensuring structured, auditable, and secure operations.

## Overview

A2S is designed for scenarios where AI agents operate in dynamic environments, discovering and invoking new services as they emerge. By defining capabilities, parameters, tasks, and flows, A2S standardizes how agents interface with services and compose functionalities.

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

- **Agent-First Design:** Specifically tailored for AI agents to discover and orchestrate capabilities programmatically.
- **Dynamic Discovery:** Agents can query registries at runtime to find and incorporate new capabilities.
- **Atomic Operations:** Each API operation is represented as a discrete, auditable, and reliable request.
- **Secure Parameter Management:** Provides strong controls over inputs, outputs, data lifecycles, and credential handling.
- **Flow Control:** Built-in constructs support conditional logic, looping, parallelism, and error handling.
- **Auditing & Permissions:** Integrated audit trails, security levels, and permission management for trustworthy operations.
- **Composability:** Simple atomic capabilities can be combined into aggregate capabilities for more complex workflows.

## Core Concepts

### Capabilities

A **capability** is a unit of functionality—either a simple (atomic) operation or a complex (aggregate) workflow. Capabilities are defined in YAML or JSON, including versioning, authorship, security posture, and dependencies.

**Example Capability Metadata:**

```yaml
a2s: 1.0.0            # Protocol version (required)
id: "CapabilityID"
description: "Handles weather updates"
version: 1.0.0
authors:
  - name: "Author Name"
type: "aggregate"     # can be 'aggregate' or 'atomic'
checksum: "<sha256>"  # SHA-256 integrity hash
last_updated: 2024-12-10
tags: ["weather", "social"]
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
- Self-contained and minimal in scope.
- No external imports.
- Ideal for a single API call or a simple, isolated action.

**Aggregate Capabilities:**  
- Can import and orchestrate multiple atomic capabilities.
- Cannot import other aggregates to prevent complex nesting.
- Suitable for multi-step workflows or orchestrations.

### Dependencies and Registries

Capabilities can define dependencies and reference external registries:

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

Agents can query these registries (via semantic search or other methods) to discover and integrate new capabilities dynamically.

### Parameter Management

A2S provides structured, lifecycle-aware parameter management:

**Parameter Scopes:**
- **Inputs/Outputs (Capability Level):** Inputs are supplied at the start; outputs are produced upon completion.
- **Inputs/Outputs (Task Level):** Each task can have its own required inputs and produce its own outputs.
- **Data Lifecycles:**  
  - **Persistent:** Data persists beyond capability execution.  
  - **Session:** Data is available during the user's session.  
  - **Capability:** Data is scoped to capability execution.  
  - **Task:** Data is scoped to a single task execution.

### Parameter References

A2S supports various referencing modes to access and manipulate data:

1. **Schema References:** Link directly to definitions within schemas.

   ```yaml
   inputs:
     user_type:
       $ref: "#/schemas/UserType"
   ```

2. **Value References:** Dynamically reference outputs of previous tasks, inputs, or stored credentials.

```yaml
   mappings:
     temperature: {getWeather.outputs.temperature}
     userId: {inputs.user.id}
     token: {services.weather-api.auth.token}
     version: {capability.version}
   ```
3. **String Templates:** Interpolate data within string parameters.


   ```yaml
   message: "Current weather in {inputs.city}: {getWeather.outputs.conditions}"
   ```
4. **Conditional References:** Base execution paths on dynamic conditions.

   ```yaml
   condition:
     if: {decideToPost.outputs.shouldPost}
   ```

### Service Integration

Define external services and their credentials, rate limits, and other properties:

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


### Flow Control

A2S supports advanced flow control patterns:

- **Sequential Steps:** Execute tasks in sequence.
- **Conditionals:** Branch logic based on conditions.
- **Loops:** Repeat steps until conditions are met.
- **Parallel Execution:** Run multiple tasks concurrently.
- **Error Handling:** Implement try/catch blocks and fallback actions.

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

Tasks are the atomic units of work. A single capability can have multiple tasks, such as requesting data from an API, running an agent decision, or performing conditional checks.

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
```

### A2S Registry

A2S registries store capabilities and allow agents to discover them using semantic queries. Agents break down user queries into capabilities and execute them to fulfill requests.

**Example:**

```
User Query: "Find a capability that retrieves weather and posts tweets."
Agent → Registry → Finds WeatherUpdateCapability → Executes Tasks
```

## Best Practices

- **Security & Auditing:** Always define audit status, required permissions, and follow data lifecycle rules.
- **Service Integration:** Keep service definitions clear and secure. Document authentication and rate limits.
- **Error Handling:** Provide meaningful error messages, implement fallback strategies, and handle common error conditions gracefully.
- **Capability Design:** Keep atomic capabilities small and reusable. Use aggregates for complex orchestration. Verify checksums and handle dependencies carefully.

## SDK Usage

A2S SDKs simplify integration, enabling easy discovery and execution of capabilities:

```typescript
import { A2SRegistry, A2SAgent } from '@a2s/core';

const registry = new A2SRegistry();
const agent = new A2SAgent();

// Query the registry for capabilities
const capabilities = registry.findCapability("Check weather and tweet");

// Execute a capability
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
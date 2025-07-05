# Deprecated
This repository has been deprecated. This was my initial stab at creating **Enact** which I started in Nov 2024. Please refer to the new repository at: [ENACT SPEC](https://github.com/EnactProtocol/specification)


# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow) ![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg) [![Discord](https://img.shields.io/badge/Discord-A2S_PROTOCOL-blue?logo=discord&logoColor=white)](https://discord.gg/mMfxvMtHyS)

The **Agent-to-Service Protocol (A2S)** provides a standardized framework for AI agents to dynamically discover, understand, and interact with services at runtime. It allows agents to transcend their initial programming by discovering new capabilities, orchestrating external APIs, and handling complex workflows—all while maintaining a structured, auditable, and secure operational environment.

## Overview

A2S is designed for scenarios where AI agents operate in dynamic environments, discovering and invoking new services as they emerge. By defining capabilities, parameters, tasks, and flow controls, A2S standardizes how agents interface with services and compose functionalities.

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
- **Secure Parameter Management:** Strong controls over inputs, outputs, data lifecycles, and credential handling.
- **Flow Control:** Built-in constructs support conditional logic, looping, parallelism, and error handling.
- **Auditing & Permissions:** Integrated audit trails, security levels, and permission management ensure trustworthy operations.
- **Composability:** Simple atomic capabilities can be combined into aggregate capabilities for more complex workflows.

## Core Concepts

### Capabilities

A **capability** is a unit of functionality—either a simple (atomic) operation or a complex (aggregate) workflow. Capabilities are defined in YAML or JSON, including versioning, authorship, security posture, and dependencies.

**Example Capability Metadata:**

```yaml
a2s: 1.0.0
id: "CapabilityID"
description: "Handles weather updates"
version: 1.0.0
authors:
  - name: "Author Name"
type: "aggregate"     # 'aggregate' or 'atomic'
checksum: "<sha256>"  # SHA-256 integrity hash
last_updated: 2024-12-10
tags: ["weather", "social"]
security:
  audit:
    status: "audited"     # audited | unaudited | in-progress
    provider: "SecurityFirm Inc."
    id: "AUDIT-2024-001"
    url: "https://security.example.com/audits/AUDIT-2024-001"
  permissions:
    level: "elevated"     # basic | elevated | admin
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
- Cannot import other aggregates to prevent overly complex chains.  
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
- **Inputs/Outputs (Capability Level):** Inputs are supplied at the start; outputs are produced at completion.
- **Inputs/Outputs (Task Level):** Each task defines its own inputs and outputs.
- **Data Lifecycles:**
  - **Persistent:** Data persists beyond capability execution.  
  - **Session:** Data is available during the user’s session.  
  - **Capability:** Data is valid only during capability execution.  
  - **Task:** Data is valid only during a single task execution.

### Parameter References

A2S supports multiple referencing modes to access and manipulate data:

1. **Schema References:** Link to schema definitions.
   ```yaml
   inputs:
     user_type:
       $ref: "#/schemas/UserType"
   ```
2. **Value References:** Dynamically reference other tasks, inputs, or stored credentials.
   ```yaml
   mappings:
     temperature: {getWeather.outputs.temperature}
     userId: {inputs.user.id}
     token: {services.weather-api.auth.token}
     version: {capability.version}
   ```
3. **String Templates:** Interpolate data directly into strings.
   ```yaml
   message: "Current weather in {inputs.city}: {getWeather.outputs.conditions}"
   ```
4. **Conditional References:** Control flow based on dynamic conditions.
   ```yaml
   condition:
     if: {decideToPost.outputs.shouldPost}
   ```

### Service Integration

Define how capabilities interact with external services, including credentials and rate limits:

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

- **Sequential Steps:** Execute tasks in order.
- **Conditionals:** Branch logic based on conditions.
- **Loops:** Repeat steps until conditions are met.
- **Parallel Execution:** Run multiple tasks concurrently.
- **Error Handling:** Use try/catch blocks with fallback tasks.

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

    # Override Tasks Inputs/Ouputs
    - task: getWeather
      override_mappings:
        input:
          location: {custom_storage.custom_vars1}
```

```yaml
custom_storage:
  custom_vars1:

```

### Tasks

Tasks are atomic units of work. A single capability may have multiple tasks—fetching data, making decisions, or running conditions.

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

### Tasks Types

Tasks come in several types:

- `request`: Execute an API operation.
- `agent_decision`: Enable agent decision-making.
- `condition`: Implement conditional branching.
- `capability`: Execute an imported capability.
- `prompt`:  load prompt for agent.

More tasks can be added in future.

### A2S Registry

A2S registries enable semantic search and retrieval of capabilities. Agents can find and execute capabilities that match user intentions.

**Example:**

```
User Query: "Find a capability that retrieves weather and posts tweets."
Agent → Registry → Finds WeatherUpdateCapability → Executes Tasks
```

![A2S Flow](diagram.png)

## Best Practices

- **Security & Auditing:** Always specify audit status, required permissions, and adhere to defined data lifecycles.
- **Service Integration:** Keep service definitions consistent and secure. Document authentication and rate limits clearly.
- **Error Handling:** Provide meaningful error messages, fallback strategies, and handle common conditions like timeouts or rate limits gracefully.
- **Capability Design:** Keep atomic capabilities focused and reusable. Employ aggregates for complex workflows. Validate checksums and dependency integrity.

## SDK Usage

A2S SDKs simplify integration by helping with discovery and execution of capabilities:

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

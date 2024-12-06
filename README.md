# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow) ![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg) [![Discord](https://img.shields.io/badge/Discord-A2S_PROTOCOL-blue?logo=discord&logoColor=white)](https://discord.gg/mMfxvMtHyS)

The **Agent-to-Service Protocol (A2S)** enables AI agents to dynamically discover and execute service capabilities at runtime. A2S defines how agents find various capabilities and securely interact with them, making it a powerful framework for integrating AI with APIs.

## Overview

**Why A2S?** Create AI agents that can dynamically discover and execute new capabilities at runtime, extending beyond their initial programming.

## Example

Here's how A2S works in a chat-based interaction:

```
┌──────────────────── A2S Chat Session ────────────────────┐

👤 [USER] > Check the weather for my picnic in Central Park and tweet it.

🤖 [AGENT] > Found capability: WeatherUpdateCapability
            └─ Services: api.weather.com, api.twitter.com

   [AGENT] > Task breakdown:
            ├─ 1. Get weather forecast
            └─ 2. Post weather update

   [AGENT] > Would you like a specific message with the weather?

👤 [USER] > Yes, mention it's for a weekend picnic.

🤖 [AGENT] > Tweet posted: "Weekend picnic weather update for Central Park: 
            Sunny with light clouds, high of 75°F. Perfect picnic weather! 🧺☀️"
└───────────────────────────────────────────────────────────┘
```

## Core Features

- **Agent-First Design**: Purpose-built for AI agents to understand and orchestrate API services.
- **Dynamic Discovery**: Runtime service and capability discovery without pre-programming.
- **Atomic Operations**: One request per API operation for clarity and reliability.
- **Secure Parameter Management**: Data handling with well-defined inputs, outputs, and lifecycles.
- **Flow Control**: Support for conditional execution and agent decision-making.
- **Security & Auditing**: Built-in audit trail and permission management.
- **Capability Composition**: Support for atomic and aggregate capabilities.

## Core Concepts

### Capabilities

Each capability is specified in YAML or JSON format with mandatory metadata:

```yaml
a2s: 1.0.0               # Protocol version (required)
id: "CapabilityID"       # Unique identifier
description: | 
  Capability description
  This is the description
version: 1.0.0           # Capability version
authors:
  - name: "Author Name"
type: "aggregate"        # aggregate | atomic
checksum: "<sha256>"     # SHA-256 hash

audit:
  status: "audited"      # audited | unaudited | in-progress
  provider: "SecurityFirm Inc."
  id: "AUDIT-2024-001"
  url: "https://security.example.com/audits/AUDIT-2024-001"

permissions:
  level: "elevated"      # basic | elevated | admin
  description: "Required permissions description"
  capabilities:
    - "capability_name"
```

### Capability Types and Composition

A2S supports two types of capabilities:

1. **Atomic Capabilities**
   - Self-contained with no external dependencies.
   - Cannot import other capabilities.
   - Building blocks for larger functionalities.

2. **Aggregate Capabilities**
   - Can import and compose atomic capabilities.
   - Cannot import other aggregate capabilities (prevents circular dependencies).
   - Used for complex workflows and multi-step processes.

### Dependencies and Registries

Capabilities can define their dependencies and registry sources:

```yaml
registries:
  default: "https://registry.a2s.dev"
  custom: "https://custom-registry.company.com"

dependencies:
  weather:
    namespace: "core/weather"
    id: "WeatherCapability"
    version: "^1.0.0"
    checksum: "sha256:abc123..."
    registry: registries.custom
```

### Tasks

Tasks come in several types:

- `request`: Execute an API operation.
- `agent_decision`: Enable agent decision-making.
- `condition`: Implement conditional branching.
- `capability`: Execute an imported capability.
- `prompt`:  load prompt for agent.

```yaml
tasks:
  - id: analyzeWeather
    type: prompt
    state:
    input:
      - temperature: 
          mapping: "getWeather.outputs.temperature"
      - conditions: 
          mapping: "getWeather.outputs.conditions"
      - threshold: 
          mapping: "inputs.threshold"
        output:
          analysis: string
          severity: string
    prompt:
      template: |
        Analyze the following weather conditions:
        Temperature: {temperature}°C
        Conditions: {conditions}
        Alert Threshold: {threshold}°C

        Provide a brief analysis of these conditions and determine their severity.
        Return your response in this format:
        Analysis: [your weather analysis]
        Severity: [high|medium|low]
      format:
        analysis: string
        severity: 
          type: string
          enum: [high, medium, low]

### Requests

Requests represent single API operations with:

- One endpoint.
- One HTTP method.
- Defined input/output contract.

Example request specification:

```yaml
getWeatherRequest:
  format: OpenAPI
  specification:
    openapi: 3.0.1
    info:
      title: Weather API
      version: 1.0.0
    servers:
      - url: https://api.weather.com
    paths:
      /weather/{city}:    # Single endpoint
        get:              # Single operation
          summary: "Get weather for a city"
          parameters:
            - in: path
              name: city
              required: true
              schema:
                type: string
```

### Parameter Management and Lifecycles

A2S handles data through well-defined parameters at both the capability and task levels. Using OpenAPI conventions ensures consistency and clarity in data definitions. Lifecycles are specified for outputs to inform clients how to store and manage data.

- **Capability-Level Parameters**:
  - **Inputs**: Initial parameters required to start the capability.
  - **Outputs**: Final parameters after the capability completes, with defined lifecycles.

- **Task-Level Parameters**:
  - **Inputs**: Data required for the task to execute.
  - **Outputs**: Data produced by the task, with defined lifecycles.

#### Lifecycles

- **Persistent**: Data that should be stored and persists beyond the execution of the capability.
- **Session**: Data that persists for the duration of the user's session.
- **Capability**: Data that exists only during the capability's execution.
- **Task**: Data that exists only during the execution of a single task.

By specifying lifecycles, clients know how long to retain outputs and how to manage data securely.


### Parameter References

A2S uses three mechanisms for referencing parameters:

1. **Schema References** (`$ref`): References type definitions and schemas
```yaml
inputs:
  user_type:
    $ref: "#/schemas/UserType"    # References schema definition
```

2. **Runtime Values** (`mapping`): References values from other tasks or capability inputs
```yaml
inputs:
  temperature:
    mapping: "getWeather.outputs.temperature"  # References actual value
  message:
    mapping: "Current weather: {getWeather.outputs.conditions}"  # Template with value
```

3. **Conditional References**: Used in flow control and conditions
```yaml
condition: "{decideToPost.outputs.shouldPost} == true"
```

#### Reference Paths
- `inputs.*` - Capability-level input values
- `outputs.*` - Capability-level output values
- `taskId.outputs.*` - Specific task's output values
- `capability.*` - Access to capability metadata

#### Best Practices
- Use `$ref` when referencing schema definitions
- Use `mapping` when referencing runtime values
- Use single curly braces `{value}` for interpolation in mappings and conditions
- Always use full paths to make data flow explicit
```

### Services

Services define the external APIs that the capability interacts with:

```yaml
services:
  "api.example.com":
    type: "weather-api"
    oauth2:                    # Authentication method
      read:
        - client_id
        - client_secret
      write:
        - access_token
        - refresh_token
    tasks: ["getWeatherTask"]  # Associated tasks
```

### Example Task Definition

```yaml
tasks:
  - id: getWeatherTask
    type: request
    requires_service: "api.example.com"
    inputs:
      city:
        type: string
        description: "Name of the city"
    outputs:
      temperature:
        type: number
        format: float
        description: "Temperature in Celsius"
        lifecycle: capability  # Data exists during capability execution
      conditions:
        type: string
        description: "Weather conditions"
        lifecycle: capability
    definition:
      request: "#/requests/getWeatherRequest"
    error_handling:
      on_failure:
        action: "continue"
        message: "Error fetching weather data"
```

### Flow Control

Define execution flow with sequential, parallel, or conditional steps:

```yaml
flow:
  type: sequence
  steps:
    - task: getWeatherTask 
    - task: decideToPost
    - condition:
        if: "decideToPost.outputs.shouldPost"
        then:
          - task: postToSocialMedia
        else:
          - task: logDecision
    - parallel:
        - task: task1
        - task: task2
```

### A2S Registry

The A2S Registries enable agents to query and discover new capabilities. The default registry uses a graph database that allows agents to perform semantic searches for relevant capabilities.

Agents can break down user queries into multiple intents and utilize the registry to determine the capabilities that will provide the desired outcome.

![A2S Flow](diagram.png)

## Best Practices

1. **Security First**
   - Always specify audit status.
   - Define minimum required permissions.
   - Use secure parameter management.
   - Specify lifecycles for outputs to manage data retention.

2. **Service Integration**
   - One service per distinct API.
   - Explicit task associations.
   - Clear authentication requirements.

3. **Parameter Management**
   - Use OpenAPI conventions for data definitions.
   - Clearly define inputs and outputs.
   - Define lifecycles for outputs.

4. **Error Handling**
   - Define failure actions.
   - Include meaningful messages.
   - Consider fallback options.

5. **Capability Design**
   - Keep atomic capabilities focused and reusable.
   - Use aggregate capabilities for complex workflows.
   - Carefully manage dependency chains.
   - Verify checksums for all dependencies.

## SDK Usage

### TypeScript/JavaScript

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

## Example

Below is an example of an aggregate capability that fetches weather information and posts an update to social media if the weather meets a certain condition. Parameters are defined at both the capability and task levels using OpenAPI conventions, and lifecycles are specified for outputs.

```yaml
a2s: 1.0.0
id: WeatherUpdateCapability
description: |
  Gets weather and posts updates based on a threshold temperature
version: 1.0.0
type: aggregate

authors:
  - name: Jane Smith
checksum: "<calculated_checksum>"
source_url: https://github.com/org/repo/weather/WeatherUpdateCapability.yaml

audit:
  status: audited
  provider: SecurityFirm Inc.
  id: AUDIT-2024-001
  url: https://security.example.com/audits/AUDIT-2024-001

permissions:
  level: elevated
  description: Requires ability to post on social media
  capabilities:
    - post_social_media

services:
  api.weather.com:
    type: weather-api
    oauth2:
      read:
        - client_id
        - client_secret
      write:
        - access_token
        - refresh_token
    tasks: [getWeather]

  api.twitter.com:
    type: social-media
    oauth2:
      read:
        - client_id
        - client_secret
      write:
        - access_token
    tasks: [postSocialMedia]

# Capability-level parameters
inputs:
  services:
    api.weather.com:
      oauth2:
        client_id:
          type: string
          description: "Weather API client ID"
        client_secret:
          type: string
          description: "Weather API client secret"
    api.twitter.com:
      oauth2:
        client_id:
          type: string
          description: "Twitter API client ID"
        client_secret:
          type: string
          description: "Twitter API client secret"
  city:
    type: string
    description: "City name to fetch weather for"
    example: "San Francisco"
  threshold:
    type: number
    format: float
    description: "Temperature threshold in Celsius"
    example: 25

outputs:
  post_id:
    type: string
    description: "ID of the posted tweet"
    lifecycle: persistent  # Client should store this data
  weather:
    type: object
    properties:
      temperature:
        type: number
        format: float
        description: "Temperature in Celsius"
      conditions:
        type: string
        description: "Weather conditions"
    lifecycle: capability  # Data exists during capability execution

tasks:
  - id: getWeather
    type: request
    requires_service: "api.weather.com"
    inputs:
      city:
        $ref: "#/inputs/city"
      oauth2:
        client_id:
          $ref: "#/inputs/services/api.weather.com/oauth2/client_id"
        client_secret:
          $ref: "#/inputs/services/api.weather.com/oauth2/client_secret"
    outputs:
      temperature:
        type: number
        format: float
        description: "Temperature in Celsius"
        lifecycle: capability
      conditions:
        type: string
        description: "Weather conditions"
        lifecycle: capability
    definition:
      request: "#/requests/getWeatherRequest"
    error_handling:
      on_failure:
        action: "continue"
        message: "Weather data fetch failed"

  - id: decideToPost
    type: agent_decision
     inputs:
      threshold:
        $ref: "#/inputs/threshold"      # Schema reference
      temperature:
        mapping: "getWeather.outputs.temperature"  # Runtime value
    outputs:
      shouldPost:
        type: boolean
        description: "Whether to post based on the threshold"
        lifecycle: flow
    definition:
      logic: |
        shouldPost = temperature >= threshold

  - id: postSocialMedia
    type: request
    requires_service: "api.twitter.com"
    condition: "{decideToPost.outputs.shouldPost} == true"
    inputs:
      oauth2:
        client_id:
          $ref: "#/inputs/services/api.twitter.com/oauth2/client_id"
        client_secret:
          $ref: "#/inputs/services/api.twitter.com/oauth2/client_secret"
      message:
        type: string
         mapping: "Current weather in {inputs.city}: {getWeather.outputs.conditions}, {getWeather.outputs.temperature}°C"
    outputs:
      post_id:
        type: string
        description: "ID of the posted tweet"
        lifecycle: persistent
    definition:
      request: "#/requests/postTweetRequest"
    error_handling:
      on_failure:
        action: "continue"
        message: "Failed to post on social media"

flow:
  type: sequence
  steps:
    - task: getWeather
    - task: decideToPost
    - task: postSocialMedia

requests:
  getWeatherRequest:
    format: OpenAPI
    specification:
      openapi: 3.0.1
      info:
        title: "Weather API"
        version: 1.0.0
      servers:
        - url: https://api.weather.com
      paths:
        /weather/{city}:
          get:
            summary: "Get current weather"
            parameters:
              - name: "city"
                in: "path"
                required: true
                schema:
                  type: "string"
            responses:
              '200':
                description: "Successful weather data retrieval"
                content:
                  application/json:
                    schema:
                      type: "object"
                      properties:
                        temperature:
                          type: "number"
                          format: "float"
                          description: "Current temperature in Celsius"
                        conditions:
                          type: "string"
                          description: "Weather conditions"

  postTweetRequest:
    format: OpenAPI
    specification:
      openapi: 3.0.1
      info:
        title: "Twitter API"
        version: 1.0.0
      servers:
        - url: https://api.twitter.com
      paths:
        /tweets:
          post:
            summary: "Post a new tweet"
            requestBody:
              required: true
              content:
                application/json:
                  schema:
                    type: "object"
                    properties:
                      message:
                        type: "string"
            responses:
              '201':
                description: "Tweet posted successfully"
                content:
                  application/json:
                    schema:
                      type: "object"
                      properties:
                        post_id:
                          type: "string"
                          description: "Unique identifier for the tweet"

examples:
  - description: "Post weather update when temperature exceeds threshold"
    inputs:
      services:
        api.weather.com:
          oauth2:
            client_id: "weather_client_id"
            client_secret: "weather_client_secret"
        api.twitter.com:
          oauth2:
            client_id: "twitter_client_id"
            client_secret: "twitter_client_secret"
      city: "San Francisco"
      threshold: 25
    expected_outputs:
      post_id: "123456789"
      weather:
        temperature: 28
        conditions: "Sunny"
    tasks:
      getWeather:
        outputs:
          temperature: 28
          conditions: "Sunny"
      decideToPost:
        outputs:
          shouldPost: true

  - description: "Do not post when temperature is below threshold"
    inputs:
      services:
        api.weather.com:
          oauth2:
            client_id: "weather_client_id"
            client_secret: "weather_client_secret"
        api.twitter.com:
          oauth2:
            client_id: "twitter_client_id"
            client_secret: "twitter_client_secret"
      city: "Seattle"
      threshold: 25
    expected_outputs:
      post_id: null
      weather:
        temperature: 18
        conditions: "Cloudy"
    tasks:
      getWeather:
        outputs:
          temperature: 18
          conditions: "Cloudy"
      decideToPost:
        outputs:
          shouldPost: false
```
## License

This project is licensed under the [MIT License](LICENSE).
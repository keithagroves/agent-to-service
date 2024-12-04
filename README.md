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
- **Secure State Management**: Data handling with storage types and lifecycles.
- **Flow Control**: Support for conditional execution and agent decision-making.
- **Security & Auditing**: Built-in audit trail and permission management.
- **Capability Composition**: Support for atomic and aggregate capabilities.

## Core Concepts

### Capabilities

Each capability is specified in YAML or JSON format with mandatory metadata:

```yaml
a2s: 1.0.0               # Protocol version (required)
id: "CapabilityID"       # Unique identifier
description: "Capability description"
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
- `sampling`: Request LLM completions.
- `capability`: Execute an imported capability.

```yaml
tasks:
  - id: getWeather
    type: capability
    definition:
      - id: getCurrentWeather
        type: capability
        uses: "weather/getWeather"
```

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

### State Management

A2S handles data through storage types and lifecycles:

- **Storage Types**:
  - **Persistent**: Data that persists across multiple sessions or executions.
  - **Transient**: Data that exists only during the execution of the capability.

- **Lifecycles**:
  - **Capability**: Data exists for the duration of the capability's execution.
  - **Session**: Data persists for the user's session.
  - **Execution**: Data exists only for the current execution step.

Example state management definition:

```yaml
state:
  input:
    city:
      type: string
      lifecycle: capability
      required: true

  output:
    persistent:
      - services["api.example.com"].oauth2.access_token
    transient:
      weatherData:
        type: object
        properties:
          temperature:
            type: number

  runtime:
    transient:
      tempData:
        type: object
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
    tasks: ["getWeatherTask"]        # Associated tasks
```

### Example Task Definition

```yaml
tasks:
  - id: getWeatherTask
    type: request
    requires_service: "api.example.com"
    definition:
      request: "#/requests/getWeatherRequest"
    input_mapping:
      city: "#/state/input/city"
    output_mapping:
      temperature: "#/state/output/weatherData/temperature"
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
    - type: condition
      if: "#/tasks/decideToPost/output/shouldPost"
      then:
        tasks:
          - task: postToSocialMedia
      else:
        task: logDecision
```

### A2S Registry

The A2S Registries enable agents to query and discover new capabilities. The default registry uses a graph database that allows agents to perform semantic searches for relevant capabilities.

Agents can break down user queries into multiple intents and utilize the registry to determine the capabilities that will provide the desired outcome.

![A2S Flow](diagram.png)

## Best Practices

1. **Security First**
   - Always specify audit status.
   - Define minimum required permissions.
   - Use secure state management.

2. **Service Integration**
   - One service per distinct API.
   - Explicit task associations.
   - Clear authentication requirements.

3. **State Management**
   - Minimize persistent state.
   - Use appropriate lifecycles.
   - Clear data schemas.

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

  const stateStore = new StateStore();
  await agent.resolveState(capability, stateStore);

  const executor = new CapabilityExecutor(stateStore);
  return await executor.execute(capability);
}
```

## Example

Below is an example of an aggregate capability that fetches weather information and posts an update to social media if the weather is noteworthy.

```yaml
a2s: 1.0.0
id: WeatherUpdateCapability
description: Gets weather and posts noteworthy updates
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
  description: Requires ability to post on social media and send alerts
  capabilities:
    - post_social_media
    - send_alerts

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

  twilio.com:
    type: messaging
    apiKey:
      read:
        - key
    tasks: [sendAlert]

state:
  input:
    - "#/services/api.weather.com/oauth2/client_id"
    - "#/services/api.weather.com/oauth2/client_secret"
    - "#/services/api.twitter.com/oauth2/client_id"
    - "#/services/api.twitter.com/oauth2/client_secret"
    - "#/services/twilio.com/apiKey/key"
    city:
      type: string
      description: City name to fetch weather for
      required: true
      example: San Francisco

  output:
    persistent:
      - "#/services/api.weather.com/oauth2/access_token"
      - "#/services/api.weather.com/oauth2/refresh_token"
      - "#/services/api.twitter.com/oauth2/access_token"
    transient:
      weatherDetails:
        type: object
        description: Current weather information
        properties:
          location:
            type: string
            description: City name
            example: San Francisco
          weather:
            type: string
            description: Weather description
            example: Sunny
          temperature:
            type: number
            description: Temperature in Celsius
            example: 22.5
        required: [location, weather, temperature]

  runtime:
    transient:
      shouldPost:
        type: boolean
        description: Whether the weather warrants a post
        example: true 
      coordinates:
        type: object
        description: Geographic coordinates
        properties:
          long:
            type: number
            description: Longitude coordinate
            format: float
            example: -122.4194
          lat:
            type: number
            description: Latitude coordinate
            format: float
            example: 37.7749
        required: [long, lat]

requests:
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
        /weather/{city}:
          get:
            summary: Get current weather
            description: Retrieves current weather for a specified city
            parameters:
              - name: city
                in: path
                required: true
                schema:
                  type: string
                  example: San Francisco
            responses:
              '200':
                description: Successful weather data retrieval
                content:
                  application/json:
                    schema:
                      type: object
                      properties:
                        temperature:
                          type: number
                          format: float
                          description: Current temperature in Celsius
                          example: 22.5

tasks:
  - id: getWeather
    type: request
    requires_service: "#/services/api.weather.com"
    definition:
      request: "#/requests/getWeatherRequest"
    input_mapping:
      city: "#/state/input/city"
    output_mapping:
      temperature: "#/state/output/weatherDetails/temperature"
      location: "#/state/output/weatherDetails/location"
      weather: "#/state/output/weatherDetails/weather"
    error_handling:
      on_failure:
        action: continue
        message: "Weather data fetch failed"

  - id: decideToPost
    type: agent_decision
    description: Evaluate if weather is noteworthy
    output_mapping:
      shouldPost: "#state/runtime/transient/shouldPost

  - id: postSocialMedia
    type: request
    requires_service: "#/services/api.twitter.com"
    description: Post to social media
    input_mapping:
      message: "Today's weather in ${#/state/output/weatherDetails/location}: ${#/state/output/weatherDetails.weather}, ${#/state/output/weatherDetails.temperature}°C"

  - id: sendAlert
    type: request
    requires_service: "#/services/twilio.com"
    description: Send weather alert
    input_mapping:
      message: "Alert: Noteworthy weather in ${#/state/output/weatherDetails/location}"

  - id: logDecision
    type: request
    description: Log that no action was taken

flow:
  type: sequence
  steps:
    - task: getWeather
    - task: decideToPost
    - type: condition
      if: "#/tasks/decideToPost/output/shouldPost"
      then:
        type: parallel
        tasks: [postSocialMedia, sendAlert]
      else:
        task: logDecision

examples:
  - description: Post weather alert for high temperature
    input:
      services:
        api.weather.com:
          oauth2:
            client_id: example_id
            client_secret: example_secret
        api.twitter.com:
          oauth2:
            client_id: twitter_id
            client_secret: twitter_secret
        twilio.com:
          apiKey:
            key: twilio_key
      city: San Francisco
    expected_output:
      weatherDetails:
        location: San Francisco
        weather: "Sunny"
        temperature: 32
      result: "Posted: High temperature alert for San Francisco. Current temperature: 32°C"

  - description: Normal weather update without alert
    input:
      services:
        api.weather.com:
          oauth2:
            client_id: example_id
            client_secret: example_secret
        api.twitter.com:
          oauth2:
            client_id: twitter_id
            client_secret: twitter_secret
      city: Seattle
    expected_output:
      weatherDetails:
        location: Seattle
        weather: "Cloudy"
        temperature: 18
      result: "Weather conditions normal, no alert needed"
```

## License

This project is licensed under the [MIT License](LICENSE).
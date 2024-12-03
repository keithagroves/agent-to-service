# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow) ![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg) [![Discord](https://img.shields.io/badge/Discord-A2S_PROTOCOL-blue?logo=discord&logoColor=white)](https://discord.gg/mMfxvMtHyS)

The **Agent-to-Service Protocol (A2S)** enables AI agents to dynamically discover and execute service capabilities at runtime. A2S defines how agents find various capabilities, and securely interact with them, making it a powerful framework for integrating AI with APIs.

## Overview

**Why A2S?** Create AI Agents that can dynamically discover and execute new capabilities at runtime, extending beyond their initial programming.

## Example

Here's how A2S works in a chat-based interaction:

```
┌──────────────────── A2S Chat Session ────────────────────┐

👤 [USER] > Check the weather for my picnic in Central Park and tweet it.

🤖 [AGENT] > Found capability: PostWeatherTweet
            └─ Services: WeatherAPI.com, Twitter API

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

- **Agent-First Design**: Purpose-built for AI agents to understand and orchestrate API services
- **Dynamic Discovery**: Runtime service and capability discovery without pre-programming
- **Atomic Operations**: One request per API operation for clarity and reliability
- **Secure State Management**: Three-tier system for secure data handling
- **Flow Control**: Support for conditional execution and agent decision-making

## Core Concepts

### Capabilities

Capabilities are the foundational units of A2S that define agent-service interactions. Each capability is specified in YAML or JSON format with:

```yaml
a2s: 1.0.0               # Protocol version (required)
id: "WeatherCapability"  # Unique identifier
description: "Fetches weather data and evaluates its impact on user activities."
domains:                 # Supported domains
  - "api.weather.com"
version: 1.0.0           # Capability version
status: "stable"         # stable, beta, deprecated
checksum: "<sha256>"     # SHA-256 hash (excluding this field)
authors:
  - name: "Author Name"
execution:
  type: "sequence"       # sequence|parallel|condition
  tasks:
    # Task definitions
```

### Tasks

Tasks come in several types:
- `request`: Execute an API operation
- `agent_decision`: Enable agent decision-making
- `condition`: Implement conditional branching
- `sampling`: Request LLM completions

### Requests

Requests represent single API operations with:
- One endpoint
- One HTTP method
- Defined input/output contract

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
        get: {}           # Single operation
```

### State Management

Three-tier state system:
```yaml
temperature:
  type: number          # string, number, boolean, date, object, array
  value: $.temp         # Direct value or reference
  storage: temporary    # service, shared, temporary
```

- **Service Variables**: Encrypted domain-specific values
- **Shared Variables**: Cross-capability accessible data
- **Temporary Variables**: Execution-scoped transient data

### A2S Registry

The A2S Registries enable agents to query and discover new capabilities. The default registry uses a graph database that allows agents to perform semantic searches for relevant capabilities.

Agents can break down user queries into multiple intents and utilize the registry to determine the capabilities that will provide the desired outcome.

![A2S Flow](diagram.png)

### Creating a New Capability

Each capability begins with a header containing essential metadata:

```yaml
a2s: "<protocol_version>"
id: "<capability_id>"
description: "<capability_description>"
domains:
  - "<domain1>"
  - "<domain2>"
version: "<capability_version>"
status: "stable"          # stable, beta, deprecated
support:
  url: "https://github.com/org/a2s-capabilities/issues/new?template=capability_issue.md&title=[my-capability-name]"
checksum: "<checksum_value>"
authors:
  - name: "<author_name>"
execution:
  type: "<execution_type>"
  tasks:
```

#### Example Capability

Here's a complete example of a weather-related capability:

```yaml
a2s: 1.0.0
id: WeatherUpdateCapability
description: Gets weather and posts noteworthy updates
domains:
  - "api.weather.com"
version: 1.0.0
authors:
  - name: "Jane Smith"
checksum: "<calculated_checksum>"
scope: 
source_url: https://github.com/org/repo/weather/analysis.yaml  # Optional direct link

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
            parameters:
              - name: city
                in: path
                required: true
            responses:
              '200':
                content:
                  application/json:
                    schema:
                      type: object
                      properties:
                        temperature:
                          type: number

execution:
  type: sequence
  tasks:
    - id: getWeather
      type: request
      definition:
        format: "OpenAPI"
        request: #/requests/getWeatherRequest
      input_mapping:
        city:
          type: string
          value: ${location}
          storage: temporary
      output_mapping:
        temperature:
          type: number
          value: $.weather.temp
          storage: temporary

    - id: decideToPost
      type: agent_decision
      description: Evaluate if weather is noteworthy
```

### Advanced Features

#### Conditional Tasks

Tasks can specify their next execution step using the `next` field:

```yaml
- id: checkAuth
  type: condition
  definition:
    condition: ${!auth_token || auth_token_expired}
  next:
    true: getToken
    false: getWeather

- id: getToken
  type: request
  definition:
    request: #/requests/getTokenRequest
  next: getWeather
```

## SDK Usage

### TypeScript/JavaScript
```typescript
import { A2SRegistry, A2SAgent } from '@a2s/core';

const registry = new A2SRegistry("https://registry.example.com");
const agent = new A2SAgent();

agent.useRegistries([registry]);

async function handleRequest(query: string) {
  const { intents, parameters } = agent.parseQuery(query);
  const capabilities = await registry.searchCapabilities(intents); 

  for (const capability of capabilities) {
    if (!agent.verifyChecksum(capability)) continue;
    
    const stateStore = new StateStore();
    await agent.resolveState(capability, stateStore);
    
    const executor = new CapabilityExecutor(stateStore);
    const response = await executor.executeCapabilityTasks(capability);
    
    return agent.processResponse(response);
  }
}
```

### Python
```python
from a2s import A2SRegistry, A2SAgent

registry = A2SRegistry("https://registry.example.com")
agent = A2SAgent()

# Similar usage pattern to TypeScript/JavaScript
```

## Best Practices

1. **Request Design**
   - One operation per request
   - Explicit contracts
   - Clear error handling

2. **State Management**
   - Prefer temporary storage
   - Explicit typing
   - Minimal shared state

3. **Agent Decisions**
   - Clear decision criteria
   - Explicit outcome handling
   - Thorough documentation

## Future Development

- GraphQL, AsyncAPI, and gRPC support
- Enhanced decision framework
- Capability composition tools
- Additional language SDKs

## License

This project is licensed under the [MIT License](LICENSE).
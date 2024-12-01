# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow)
![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)

A2S is a protocol that enables AI agents to discover and execute service capabilities at runtime. It defines how agents can find services, understand their capabilities, and securely interact with them.

## Key Features

- 🤖 **Agent-First Design**: Built for AI agents to understand and use services
- 🔍 **Runtime Discovery**: Find and use new capabilities dynamically
- 🎯 **Single Operation Actions**: Each action maps to exactly one API operation
- 🔒 **Secure State Management**: Three-tier state management system
- 🧩 **Agent Decisions**: Built-in support for agent reasoning during execution

## Core Concepts

### Actions
An action represents a single API operation. For OpenAPI specifications:
- One endpoint
- One HTTP method
- Clear input/output contract

```yaml
getWeatherAction:
  format: OpenAPI
  specification:
    paths:
      /weather/{city}:    # Single endpoint
        get: {}           # Single operation
```

### Tasks
Tasks orchestrate actions and agent decisions. Types include:
- `action`: Execute a single API operation
- `agent_decision`: Let the agent make a choice
- `condition`: Branch based on a condition
- `parallel`: Run multiple tasks simultaneously

### State Management
Every state value must specify its type and storage level:
```yaml
temperature:
  type: number          # string, number, boolean, date, object, array
  value: $.temp         # value or reference
  storage: temporary    # service, shared, or temporary
```

Storage levels:
1. **Service Variables**
   - Domain-specific encryption (e.g. Client_Secret)

2. **Shared Variables**
   - Stored and shared across cababilities

3. **Temporary Variables**
   - Clear after execution

### Capability Structure

Each capability requires essential metadata:
```yaml
a2s: 1.0.0              # Protocol version (required)
id: "WeatherCapability" # Unique identifier
description: "..."      # Human-readable explanation
charset: "utf-8"        # Default utf-8
domains:                # At least one domain
  - "api.weather.com"
version: 1.0.0          # Capability version
checksum: "<sha256>"    # SHA-256 hash (excluding this field)
authors:
  - name: "Author Name"
execution:
  type: "sequence"      # sequence|parallel|condition
  tasks:
    # Task definitions
```

## Example Capability

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

actions:
  getWeatherAction:
    format: OpenAPI
    specification:
      paths:
        /weather/{city}:
          get:
            parameters:
              - name: city
                in: path
                required: true

execution:
  type: sequence
  tasks:
    - id: getWeather
      type: action
      format: "OpenAPI"
      action_ref: #/actions/getWeatherAction
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

## Interactive Example

```
┌──────────────────── A2S Chat Session ────────────────────┐

👤 [USER] > Check the weather for my picnic in Central Park and tweet it.

🤖 [AGENT] > Found capability: PostWeatherTweet
            └─ Compatible with: Weather API, Twitter API

   [AGENT] > Task breakdown:
            ├─ 1. Get weather forecast
            └─ 2. Post weather update

   [AGENT] > Would you like a specific message with the weather?

👤 [USER] > Yes, mention it's for a weekend picnic.

🤖 [AGENT] > Tweet posted: "Weekend picnic weather update for Central Park: 
            Sunny with light clouds, high of 75°F. Perfect picnic weather! 🧺☀️"
└───────────────────────────────────────────────────────────┘
```

## SDK Usage

```typescript
// Initialize
const registry = new A2SRegistry("https://registry.example.com");
const agent = new A2SAgent();
agent.useRegistries([registry]);

agent.handleRequest(query);


// Execute capability
await agent.executeCapability(capability);
```

## Creating a new capability

Each capability task begins with a header containing essential metadata:

```yaml
a2s: "<protocol_version>"
id: "<capability_id>"
description: "<capability_description>"
domains:
  - "<domain1>"
  - "<domain2>"
version: "<capability_version>"
checksum: "<checksum_value>"
authors:
  - name: "<author_name>"
execution:
  type: "<execution_type>"
  tasks:
```

## Best Practices

1. **Action Design**
   - One operation per action
   - Clear input/output contracts
   - Explicit error responses

2. **State Management**
   - Default to temporary storage
   - Explicit type definitions
   - Clear security boundaries

3. **Agent Decisions**
   - Clear decision criteria
   - Document context needed
   - Handle all outcomes

## Future Plans

- GraphQL, AsyncAPI, and gRPC support
- Enhanced agent decision framework
- Capability composition tools
- Multi-language SDKs

```python
# Python implementation
from a2s import A2SRegistry, A2SAgent

registry = A2SRegistry("https://registry.example.com")
agent = A2SAgent()
```

```typescript
// TypeScript implementation
import { A2SRegistry, A2SAgent } from '@a2s/core';

const registry = new A2SRegistry("https://registry.example.com");
const agent = new A2SAgent();
```

```go
// Go implementation
import "github.com/a2s/core"

registry := a2s.NewRegistry("https://registry.example.com")
agent := a2s.NewAgent()
```

## Capability Chaining

Capabilities can depend on other capabilities, allowing for complex workflows. [read more](Chaining.md).

## License

This project is licensed under the [MIT License](LICENSE).
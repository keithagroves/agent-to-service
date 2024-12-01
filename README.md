# Agent-to-Service Protocol (A2S)

![Status: Alpha](https://img.shields.io/badge/Status-Alpha-yellow)  
![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)

The **Agent-to-Service Protocol (A2S)** enables AI agents to dynamically discover and execute service capabilities at runtime. A2S defines how agents find services, understand their capabilities, and securely interact with them, making it a powerful framework for integrating AI with APIs.

---

## **Key Features**

### 🤖 **Agent-First Design**
Built specifically for AI agents to understand, utilize, and orchestrate API services seamlessly.

### 🔍 **Dynamic Runtime Discovery**
Agents can find new services and capabilities on the fly without requiring pre-programmed knowledge.

### 🎯 **Single Operation Requests**
Each request corresponds to exactly one API operation, ensuring atomicity and clarity.

### 🔒 **Secure State Management**
A three-tier state management system for secure handling of data:
- **Service Variables** for sensitive, domain-specific storage.
- **Shared Variables** for cross-capability sharing.
- **Temporary Variables** for transient, execution-specific data.

### 🧩 **Flow Control**
Built-in support for conditional execution, branching, and agent decision-making.

---

## **Core Concepts**

### **Requests**
Requests represent single API operations, designed for clarity and simplicity. Each request is defined by:
- One endpoint
- One HTTP method
- A well-defined input/output contract

Example:
```yaml
getWeatherRequest:
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

### **Tasks**
Tasks orchestrate requests and logic, supporting multiple types:
- **`request`**: Execute a single API operation.
- **`agent_decision`**: Allow the agent to make a decision based on available data.
- **`condition`**: Branch execution based on a condition.

### **State Management**
State values are strictly typed and explicitly define their storage level:
```yaml
temperature:
  type: number          # Data types: string, number, boolean, date, object, array
  value: $.temp         # Direct value or reference
  storage: temporary    # Levels: service, shared, temporary
```
- **Service Variables**: Domain-specific encrypted values (e.g., `Client_Secret`).
- **Shared Variables**: Available across cababilities
- **Temporary Variables**: Cleared after execution for transient data.

### **Capability Structure**
Each capability must define essential metadata for discovery and execution:
```yaml
a2s: 1.0.0               # Protocol version (required)
id: "WeatherCapability"  # Unique identifier
description: "Fetches weather data and evaluates its impact on user activities."
domains:                 # Supported domains
  - "api.weather.com"
version: 1.0.0           # Capability version
checksum: "<sha256>"     # SHA-256 hash (excluding this field)
authors:
  - name: "Author Name"
execution:
  type: "sequence"       # sequence|parallel|condition
  tasks:
    # Task definitions
```

---

## **Example Capability**

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

---

## **Interactive Example**

Here’s how A2S could work in a chat-based interaction:

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

---

## **SDK Usage**

### JavaScript / TypeScript
```typescript
import { A2SRegistry, A2SAgent } from '@a2s/core';

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

## advanced

Each task can specify its next task using the next field:

```
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


### Python
```python
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

---

## **Best Practices**

1. **Request Design**:
   - Limit each request to one operation.
   - Ensure input/output contracts are explicit.
   - Include clear error responses.

2. **State Management**:
   - Default to temporary storage unless persistence is necessary.
   - Explicitly type all state values.
   - Avoid excessive shared variables to maintain security boundaries.

3. **Agent Decisions**:
   - Clearly define decision criteria.
   - Handle all possible outcomes explicitly.
   - Include documentation for context-sensitive decisions.

---

## **Future Plans**

- **GraphQL, AsyncAPI, and gRPC Support**: Expand supported protocols for greater flexibility.
- **Advanced Decision Framework**: Enhance agent decision-making capabilities.
- **Capability Composition Tools**: Enable intuitive chaining of complex workflows.
- **Multi-Language SDKs**: Broaden language support for easier integration.

---

## **FAQs**

### **How does A2S ensure secure interactions?**
A2S uses a three-tier state management system, with encryption for sensitive data in `Service Variables`. Agents and services communicate via secure protocols like OAuth2 or mTLS.

### **Can A2S handle non-OpenAPI services?**
Yes! While A2S primarily supports OpenAPI, it plans to include support for GraphQL, gRPC, and custom specifications.

---

## **License**
This project is licensed under the [MIT License](LICENSE).

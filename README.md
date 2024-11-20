# Agent to Service Protocol (A2S)

A standardized protocol enabling AI agents to discover and execute web service capabilities through raw HTTP requests, eliminating the need for parsing documentation or websites.

## Problem

Currently, AI agents struggle to interact with web services because they must:
- Parse complex websites or API documentation
- Handle different authentication methods
- Navigate varying API structures
- Understand multi-step processes
- Keep up with API changes

## Solution

A2S provides a central registry of services with:
1. Standardized capability discovery
2. Pre-configured raw HTTP request sequences
3. Clear variable templates for customization

Example Service:
```yaml
serviceName: "Dominos Pizza"
serviceDescription: "Online pizza ordering and delivery service"
capabilities:
  - name: "Order Pizza"
    description: "Create a new pizza delivery order"
    executionPath: "/execution/order-pizza"
    requiredScopes: ["order:write"]
  - name: "See Specials"
    description: "List available promotions"
    executionPath: "/execution/specials"
    requiredScopes: ["specials:read"]
```

Example Execution Path:
```yaml
path: "/execution/order-pizza"
description: "Request sequence for ordering a pizza"
requestSequence:
  - id: "auth"
    description: "Authenticate with the service"
    raw: |
      POST /auth HTTP/1.1
      Host: api.dominos.com
      Content-Type: application/json
      
      {
        "clientId": "${CLIENT_ID}",
        "clientSecret": "${CLIENT_SECRET}"
      }
  - id: "place-order"
    description: "Submit the pizza order" 
    raw: |
      POST /orders HTTP/1.1
      Host: api.dominos.com
      Authorization: Bearer ${AUTH_TOKEN}
      
      {
        "orderItems": [{
          "size": "${SIZE}",
          "toppings": ${TOPPINGS_ARRAY}
        }]
      }
```

## Features

- **Registry**: Single source for discovering service capabilities
- **Raw Requests**: Pre-configured HTTP requests ready for execution
- **Variable Templates**: Clear system for request customization
- **Sequence Management**: Handles multi-step processes and dependencies
- **Future-Ready**: Design supports eventual migration to .well-known endpoints

## Architecture

1. **Registry Service**
   - Maintains service catalog
   - Provides capability discovery
   - Returns execution sequences

2. **Service Integration**
   - Services register their capabilities
   - Provide request sequences
   - Optional: Future support for .well-known endpoints

3. **Agent Integration**
   - Query available services
   - Get execution sequences
   - Execute raw requests

## Getting Started

[Coming Soon]
- How to register a service
- How to query capabilities
- How to execute sequences

## Future Enhancements
- Proof of concept demo
- Enhanced authentication patterns
- Request sequence validation
- Success/failure pattern matching
- `.well-known` endpoint support for direct service discovery

## Contributing

We welcome contributions! Areas we need help with:
- Protocol specification
- Reference implementations
- Service integrations

Contributions are welcome! Please follow these steps:

1. **Fork the Repository**: Create your own fork to work on.
3. **Commit Changes**: Write clear, concise commit messages.
4. **Open a Pull Request**: Describe the changes and link any relevant issues.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---
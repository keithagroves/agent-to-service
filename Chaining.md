## Capability Chaining

Capabilities can depend on other capabilities, allowing for complex workflows. Below is an example of a capability that chains multiple dependent capabilities.

### `WeatherAlertCapability` Definition

```yaml
a2s: 1.0.0
name: "WeatherAlertCapability"
description: "Monitors weather and sends alerts through multiple channels when severe conditions are detected"
charset: "utf-8"
domains:
  - "api.weather.com"
  - "api.twitter.com"
  - "api.telegram.org"
version: "1.0"
checksum: "<calculated_checksum>"
authors:
  - name: "Jane Smith"
    email: "jane.smith@example.com"
    organization: "Weather Systems Inc."
    github: "janesmith2"

dependencies:
  - capability: "GetWeatherAlert"
    version: "^1.0.0"
    checksum: "<dependency_checksum>"
  - capability: "PostSocialUpdate"
    version: "^2.0.0"
    checksum: "<dependency_checksum>"
  - capability: "SendTelegramMessage"
    version: "^1.2.0"
    checksum: "<dependency_checksum>"

execution:
  type: sequence
  variables:
    shared:
      location: "string"
      alert_severity: "string"
      alert_message: "string"
      notification_ids: "object"
  steps:
    - id: "checkWeather"
      type: "capability"
      use: "GetWeatherAlert"
      input_mapping:
        location: "${location}"
        check_severity: true
      output_mapping:
        alert_severity: "$.severity"
        alert_message: "$.description"
        alert_time: "$.timestamp"

    - id: "evaluateAlert"
      type: "condition"
      condition: "${alert_severity} in ['severe', 'extreme']"
      if_true:
        steps:
          - id: "sendAlerts"
            type: "parallel"
            steps:
              - id: "tweetAlert"
                type: "capability"
                use: "PostSocialUpdate"
                input_mapping:
                  platform: "twitter"
                  message: "WEATHER ALERT for ${location}: ${alert_message}"
                output_mapping:
                  notification_ids.twitter: "$.post_id"

              - id: "telegramAlert"
                type: "capability"
                use: "SendTelegramMessage"
                input_mapping:
                  chat_id: "${TELEGRAM_CHAT_ID}"
                  message: "🚨 ${alert_severity} ALERT 🚨\nLocation: ${location}\n${alert_message}\nTime: ${alert_time}"
                output_mapping:
                  notification_ids.telegram: "$.message_id"

output:
  alert_info:
    severity: "${alert_severity}"
    message: "${alert_message}"
    time: "${alert_time}"
  notifications: "${notification_ids}"
```
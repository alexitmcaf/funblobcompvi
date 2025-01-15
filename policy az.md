Azure 

enforce_mandatory_tags
```
{
"mode": "All",
"policyRule": {
      "if": {
        "anyOf": [
          {
            "field": "tags[Duration]",
            "exists": "false"
          },
          {
            "field": "tags[ENV]",
            "exists": "false"
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    },
```

enforce-valid-values-in-tag
```
{
"mode": "All",
"policyRule": {
      "if": {
        "anyOf": [
          {
            "field": "tags[Duration]",
            "notin": [
              "LT",
              "ST"
            ]
          },
          {
            "field": "tags[ENV]",
            "notin": [
              "DEV",
              "INT",
              "UAT",
              "PROD",
              "LAB"
            ]
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    },
```
add-created-on-date
```
{
"mode": "All",
"policyRule": {
      "if": {
        "anyOf": [
          {
            "field": "tags['CreatedOnDate']",
            "exists": "false"
          }
        ]
      },
      "then": {
        "effect": "append",
        "details": [
          {
            "field": "tags['CreatedOnDate']",
            "value": "[concat(substring(utcNow(), 0, 19), 'Z')]"
          }
        ]
      }
    },
```

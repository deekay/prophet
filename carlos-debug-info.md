# lookup_invoice requests - all timing out with no response

## Relay
```
wss://nwc.primal.net/09nuj40lmk4t8wl01sv2dfblvdxhyu
```

## Our pubkey
```
b899da2e89f3dedca98c72407584856593fc2b59b365e8e38ccc8916d3cb212f
```

## Request being made
```json
{
  "method": "lookup_invoice",
  "params": {
    "payment_hash": "29b03c03ebc819b29c25fbad4763c718403950fff68b700f705360762f4ad903"
  }
}
```

## Sample request event IDs (any of these can be investigated)

| Request Event ID | Subscription ID |
|------------------|-----------------|
| `170f660d49c7b099e1ad340389d919b780517508a647ea13479752fb8c7196a0` | sub:16 |
| `2329fab167026e387040c9088ba2c250da79ba67f226b462f82e8de7a1fb7bb5` | sub:18 |
| `0249563452dd1b06a22f2af91882b87f3659500b4f2775ce333330d30c5387c5` | sub:20 |

## Filter we're subscribing with
```json
{
  "kinds": [23195],
  "#e": ["<request_event_id>"],
  "#p": ["b899da2e89f3dedca98c72407584856593fc2b59b365e8e38ccc8916d3cb212f"],
  "since": 1768416254
}
```

## What happens
- We receive EOSE (subscription connects successfully)
- No kind 23195 response event is ever received
- Request times out after 30 seconds

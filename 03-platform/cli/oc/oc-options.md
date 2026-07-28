---
tags:
  - cli
  - oc
  - configuration
---

#### --request-timeout

Useful when automating.
```
%> oc project --request-timeout=2
Unable to connect to the server: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
```
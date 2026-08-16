Start a new rollout, view its status or history, rollback to a previous revision of your app.

Available commands:

- cancel:     Cancel the in-progress deployment
- history:    View rollout history
- latest:      Start a new rollout for a deployment config with the latest state from its triggers
- pause:     Mark the provided resource as paused
- restart:    Restart a resource
- resume:   Resume a paused resource
- retry:        Retry the latest failed rollout
- status:      Show the status of the rollout
- undo:       Undo a previous rollout

```
%> oc rollout status deployment/oras-client
Waiting for deployment "oras-client" rollout to finish: 0 of 1 updated replicas are available...
```

```
%> oc rollout restart deployment/database  
%> oc rollout status deployment cluster-logging-operator 
deployment "cluster-logging-operator" successfully rolled out
```
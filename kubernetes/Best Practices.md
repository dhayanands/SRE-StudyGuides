# Application HA

## rolling update for deployment

```yaml
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

## Setting Zones when in cloud

[reference](https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesiozone)
`topology.kubernetes.io/zone: "us-east-1c"`

## using PodDisruptionBudget with minAvailable

[reference](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)


# ControlPlane HA

- Run control place across zones in cloud

[reference]()
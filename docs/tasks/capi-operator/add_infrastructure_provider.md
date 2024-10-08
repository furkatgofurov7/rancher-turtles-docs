---
sidebar_position: 3
---

# Installing AWS Infrastructure Provider using CAPIProvider resource

This section describes how to install the AWS `InfrastructureProvider` via `CAPIProvider`, which is responsible for managing Cluster API AWS CRDs and the Cluster API AWS controller.

:::note
This section describes how to install the raw AWS `InfrastructureProvider`, which is responsible for managing the Cluster API AWS CRDs and the Cluster API AWS controller. The detailed configuration steps are described in the [official](https://cluster-api-operator.sigs.k8s.io/03_topics/03_basic-cluster-api-provider-installation/02_installing-capz#installing-azure-infrastructure-provider) CAPI Operator documentation.
:::

*Example:*

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-variables
  namespace: capa-system
type: Opaque
stringData:
  AWS_B64ENCODED_CREDENTIALS: ZZ99ii==
  ExternalResourceGC: "true"
---
apiVersion: turtles-capi.cattle.io/v1alpha1
kind: CAPIProvider
metadata:
  name: aws
  namespace: capa-system
spec:
  name: aws
  type: infrastructure # required
  version: v2.6.1
  configSecret:
    name: aws-variables # This will additionally populate the default set of feature gates for the provider inside the secret
  variables:
    EXP_MACHINE_POOL: "true"
    EXP_EXTERNAL_RESOURCE_GC: "true"
    CAPA_LOGLEVEL: "4"
  manager:
    syncPeriod: "5m"
```

### Deleting providers

To remove the installed providers and all related Kubernetes objects just delete the following CRs:

```bash
kubectl delete coreprovider cluster-api
kubectl delete infrastructureprovider aws
```

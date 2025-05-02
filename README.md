# argocd


### Manual step
I've not yet automated the additional of a label to the argocd namespace so for now I'm going to run
sudo kubectl label namespace argocd istio-injection=enabled --overwrite

### Secret management
Installation steps can be found here: https://github.com/bitnami-labs/sealed-secrets?tab=readme-ov-file#kubeseal

You shouldn't commit plain text secrets to this repo. Please use the below example to generate a sealed secret which can be commited to this repo

```bash
kubectl create secret generic <secret-name> --from-literal=<name>=<value> --namespace=cert-manager --dry-run=client -o yaml > <generated-secret-name>.yaml
kubeseal --format=yaml --namespace=<namespace> --controller-namespace=sealed-secrets --controller-name=sealed-secrets <generated-secret-name>.yaml > sealed-<generated-secret-name>.yaml
```

You can then commit sealed-<generated-secret-name>.yaml to the relevant config directory

### Random helpful commands
to change the namespace you can run
```yaml
kubectl config set-context --current --namespace=istio-system
```

### known issues

Sometimes the monitoring stack will throw the following error
```text
  Warning  OperationCompleted  7m22s  argocd-application-controller  Sync operation to 71326b7cf66f9cbf4aa66b44c6a8a2d02128898d failed: one or more objects failed to apply, reason: CustomResourceDefinition.apiextensions.k8s.io "prometheuses.monitoring.coreos.com" is invalid: metadata.annotations: Too long: may not be more than 262144 bytes,CustomResourceDefinition.apiextensions.k8s.io "scrapeconfigs.monitoring.coreos.com" is invalid: metadata.annotations: Too long: may not be more than 262144 bytes,CustomResourceDefinition.apiextensions.k8s.io "thanosrulers.monitoring.coreos.com" is invalid: metadata.annotations: Too long: may not be more than 262144 bytes,resource mapping not found for name: "monitoring-kube-prometheus-alertmanager" namespace: "monitoring" from "/dev/shm/2366278253": no matches for kind "Alertmanager" in version "monitoring.coreos.com/v1"
```

To fix this it's easiest to just apply the CRD's manually using
kubectl apply -f https://github.com/prometheus-operator/prometheus-operator/releases/download/v0.75.1/stripped-down-crds.yaml

# Overview
The purpose of this is to be a source of shared configuration across the whole cluster.
The need for this is because ConfigMaps are scoped to NameSpaces,
hence it is not possible to address shared cluster-wide configuration with ConfigMaps.

## Installation
This is a Helm chart.
There is a default values file, and a values folder with non-default values.

Currently there are 2 non-default values files that would be used as appropriate:
* aws.condel.yaml
```
> cd <cluster-config root folder>
> helm install cluster-config -f cluster-config/values/aws.condel.yaml
```
* developer.yaml
```
> cd <cluster-config root folder>
> helm install cluster-config -f cluster-config/values/developer.yaml
```

## Configuration

### Cluster Name
This defines a name for the cluster.
There is currently one application of needing this as shared configuration: Ingress Controllers.

#### Use by Ingress Controller
The matr-ingress-controller uses this cluster name to append to all ingress host names.
If an ingress host is a.b and the cluster name is c.d then the matr-ingress-controller will respond to a.b.c.d

The cluster-config configmap template (./templates/configmap.yaml):
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: cluster-config
  namespace: {{ .Values.global.namespace }}
  annotations: {{ include "global.annotations.standard" . | indent 4 }}
data:
  name: {{ .Values.config.name }}
```

The cluster-config default values file (./values.yaml):
```
global:
  namespace: cluster-config
config:
  name: kubernetes.local
```

A cluster-config custom values file (./values/developer.yaml):
```
config:
  name: dev.matr.com
```

The helm command to install cluster-config:
```
> cd workspace/cluster-config
> helm install --name cluster-config ./ -f ./values/developer.yaml
```

A matr-ingress-controller values file:
```
global:
  namespace: ingress-nginx
  cluster:
    config:
      namespace: cluster-config
      configmap: cluster-config
```

An ingress:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus
  namespace: monitoring
spec:
  rules:
  - host: prometheus.monitoring
    http:
      paths:
      - backend:
          serviceName: prometheus
          servicePort: 9090
```

The end result of the above example is that requests to "prometheus.monitoring.dev.matr.com" would be sent to the prometheus services

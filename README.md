# cluster-config

Currently this is nothing more than a configmap to define the cluster name.
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

# frek8strade

Small project to deploy multiple freqtrade bots from a common base on a kubernetes cluster.

How to check configuration files generated by kustomize:
``` 
kubectl kustomize ./
```

How to apply configuration to your cluster:
``` 
kubectl apply -k ./
```

From base/ingress.yaml, change the HTTP route "parentRefs" in accordance with your cluster Gateway API. Here "traefik" is been used.

``` 
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: freqtrade
spec:
  parentRefs:
  - name: traefik-gateway
    namespace: default
    sectionName: websecure
  hostnames:
  - freqtrade.example.com
  rules:
  - matches:
    - path:
        value: /
    backendRefs:
    - name: freqtrade
      port: 8080
```

To add a new bot : 
 - copy/paste the repository overlays/bot1
 - add a new strategy python file in the new repository
 - change strategy name accordingly in kustomization.yaml secretGenerator and configMapGenerator 
 - change namesuffix and commonLabels 

example : 
```
nameSuffix: -bot3

commonLabels:
  app: freqtrade-bot3

secretGenerator:
- name: freqtrade-secret
  behavior: merge
  literals:
  - FREQTRADE__STRATEGY=Strategy003

configMapGenerator:
- name: freqtrade-strategies
  behavior: replace
  files: 
  - Strategy003.py
```

From the root ./kustomization.yaml don't forget to add the new resources 

```
resources:
- overlays/namespace.yaml
- overlays/bot1
- overlays/bot2

- overlays/bot3
```

then 
```
kubectl apply -k ./
```
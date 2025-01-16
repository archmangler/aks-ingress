# A selection of unwise hacks and inadvisable workarounds


* Configuring the ingress controller to use IP address only, if you don't have a DNS entry:

```
#Assuming TLS
spec:
  tls:
  - hosts:
    - "192.168.1.100"  # Replace with the public IP address of the Ingress Controller
    secretName: my-app-tls
  rules:
  - host: "192.168.1.100"  # Same IP address in the host field
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

* Even worse, without TLS:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: siren  # The namespace where the Ingress will reside
spec:
  rules:
  - host: "192.168.1.100"  # Replace with the public IP address of the Ingress Controller
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service  # Service must exist in the 'siren' namespace
            port:
              number: 80

```

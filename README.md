# Cilium Network Policy.

This repo will help you to create a network policy in Kubernetes using cilium CNI. So, only two namespaces can communicate with each other. Cilium is an open-source software for providing, securing and observing network connectivity between container workloads.

Prerequisite:-
 - A Kubernetes cluster up and running.
 - Cilium is installed.
 - Application is deployed in different namespaces. (For this task I have created an application in test, default, frontend and backend namespaces.)

The scope of the network policy is per namespace. So, for this task, I have deployed application in test, default, frontend and backend namespaces using `deploy.yaml` file.

## Manifests and there explanation.

### Network-policy manifest for frontend namespace is (`front-np.yaml`): -

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: network-policy
  namespace: frontend
spec:
  endpointSelector: {}
  ingress:
    - fromEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: backend
  egress:
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: kube-system
            k8s-app: kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: UDP
          rules:
            dns:
              - matchPattern: "*"
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: backend
    - toEntities:
        - world
```

In the above manifest I have allowed ingress and egress traffic to the backend namespace and also allow egress traffic to the internet so that my application pod can communicate to the internet. If you don’t want your pod to get connected to the internet you can remove the `toEntities` section from the manifest.

I have also allowed communication from the frontend namespace to the kube-dns pod running in the kube-system namespace. Pods running in this Kubernetes cluster use the kube-dns service for resolving service names to IPs.

### Network-policy manifest for the backend namespace is (`back-np.yaml`) : -

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: network-policy
  namespace: backend
spec:
  endpointSelector: {}
  ingress:
    - fromEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: frontend
  egress:
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: kube-system
            k8s-app: kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: UDP
          rules:
            dns:
              - matchPattern: "*"
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: frontend
    - toEntities:
        - world
```

In the above-mentioned manifest I have allowed ingress and egress traffic to the frontend namespace. The Remaining part of the manifest is the same as the frontend manifest.

### Manifest to block ingress and egress traffic in a namespace (`block-np.yaml`) : -

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
 name: untitled-policy
 namespace: default
spec:
 endpointSelector: {}
 ingress:
   - {}
 egress:
   - {}
```
In the above given manifest I have blocked all the ingress and egress traffic in default namespace.

## Testing:-

To test the application SSH into the frontend pod and curl the pods in other namespace . You will only be able to curl the pod running in backend namespace and vice versa.


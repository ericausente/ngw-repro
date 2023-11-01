# NGINX Gateway Fabric Deployment Guide

Prerequisites

Before you begin, ensure you have met the prerequisites mentioned [here](https://github.com/nginxinc/nginx-gateway-fabric/blob/v1.0.0/docs/installation.md#prerequisites).


## Deploying NGINX Gateway Fabric from Manifests
Note: By default, NGINX Gateway Fabric (NGF) will be installed into the nginx-gateway Namespace. If you wish to run NGF in a different Namespace, you'll need to modify the installation manifests.

Clone the Repository:
```
git clone https://github.com/nginxinc/nginx-gateway-fabric.git --branch v1.0.0
cd nginx-gateway-fabric
```

Checkout the Latest Tag:
```
git fetch --tags
latestTag=$(git describe --tags `git rev-list --tags --max-count=1`)
git checkout $latestTag
```

Install the Gateway API Resources:
```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v0.8.1/standard-install.yaml
```

At this point, you should notice pods being created under the gateway-system namespace.
Sample Output: 
```
gateway-system      gateway-api-admission-lhh4w                              0/1     Completed           0          8s
gateway-system      gateway-api-admission-patch-nmbdx                        0/1     Completed           1            20s
```

Deploy the NGINX Gateway Fabric CRDs:
```
kubectl apply -f deploy/manifests/crds
```

Deploy the NGINX Gateway Fabric:
```
kubectl apply -f deploy/manifests/nginx-gateway.yaml
```


Now, you should notice pods being created in the nginx-gateway namespace.
Sample Output: 
```
nginx-gateway       nginx-gateway-645f779487-drpv2                           2/2     Running             0            8s
```

Confirm NGINX Gateway Fabric is Running:
```
kubectl get pods -n nginx-gateway
```

## Creating a LoadBalancer Service

For GCP or Azure:

```
kubectl apply -f deploy/manifests/service/loadbalancer.yaml
```

To check the public IP:
```
kubectl get svc nginx-gateway -n nginx-gateway
```

For AWS:
```
kubectl apply -f deploy/manifests/service/loadbalancer-aws-nlb.yaml
```

To get the DNS name:
```
kubectl get svc nginx-gateway -n nginx-gateway
```

For testing purposes, you can resolve the DNS name to get the IP address:
```
nslookup <dns-name>
```

# Example Configuration

Configure the Gateway:
```
vi examples/cafe-example/gateway.yaml
```

gateway Sample Yaml
```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    hostname: "*.kushikimi.xyz"                                 
```

Configure the Routes:
```
vi examples/cafe-example/cafe-routes.yaml
```

HTTPRoute SAMPLE YAML:
```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: coffee
spec:
  parentRefs:
  - name: gateway
    sectionName: http
  hostnames:
  - "ngw.kushikimi.xyz"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /coffee
    backendRefs:
    - name: coffee
      port: 80
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: tea
spec:
  parentRefs:
  - name: gateway
    sectionName: http
  hostnames:
  - "ngw.kushikimi.xyz"
  rules:
  - matches:
    - path:
        type: Exact
        value: /tea
    backendRefs:
    - name: tea
      port: 80
```


DNS Configuration:
- Log in to your GoDaddy account.
- Navigate to your domain's DNS management page.
- Look for the A record for ngw.kushikimi.xyz.
- Ensure that the IP address for this A record matches the external IP of your LoadBalancer service.

DNS Propagation:
- DNS changes can take anywhere from a few minutes to 48 hours to propagate worldwide, although most often it's much quicker.
- You can use online tools like DNS Checker to check the propagation status of your subdomain.
  
Once configured, you should be able to access:
- http://ngw.kushikimi.xyz/tea
- http://ngw.kushikimi.xyz/coffee

Uninstalling NGINX Gateway Fabric
```
kubectl delete -f deploy/manifests/nginx-gateway.yaml
kubectl delete -f deploy/manifests/crds
```

Uninstall the Gateway API Resources:
Warning: This command will delete all corresponding custom resources in your cluster across all namespaces! Ensure there are no custom resources you want to keep and no other Gateway API implementations running in the cluster.

```
kubectl delete -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v0.8.1/standard-install.yaml
```


## Deploy NGINX Gateway Fabric using Helm

To deploy NGINX Gateway Fabric using Helm, please follow the instructions on this [page](https://github.com/nginxinc/nginx-gateway-fabric/blob/v1.0.0/deploy/helm-chart/README.md)


### If you intent to execute the nginx -T command (to verify the generated configuration) inside the specific nginx container of the NGF pod
```
kubectl exec  my-release-nginx-gateway-fabric-7d845c777f-fxkqh -n nginx-gateway -c nginx -- nginx -T
```

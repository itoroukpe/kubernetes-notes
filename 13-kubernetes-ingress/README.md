
# Rondus Technologies == Kubernetes Ingress
# What is the Ingress in kubernetes?

The Ingress is a Kubernetes resource that lets you configure an HTTP load balancer for applications running on Kubernetes, represented by one or more [Services](https://kubernetes.io/docs/concepts/services-networking/service/). Such a load balancer is necessary to deliver those applications to clients outside of the Kubernetes cluster. It also provides SSL Termination and SSL Redirect for HTTPS.

The Ingress resource supports the following features:
* **Content-based routing**:
    * *Host-based routing*. For example, routing requests with the host header `foo.example.com` to one group of services and the host header `bar.example.com` to another group.
    * *Path-based routing*. For example, routing requests with the URI that starts with `/serviceA` to service A and requests with the URI that starts with `/serviceB` to service B.

See the [Ingress User Guide](http://kubernetes.io/docs/user-guide/ingress/) to learn more about the Ingress resource.

## What is the Ingress Controller?

The Ingress controller is an application that runs in a cluster and configures an HTTP load balancer according to Ingress resources. The load balancer can be a software load balancer running in the cluster or a hardware or cloud load balancer running externally. Different load balancers require different Ingress controller implementations. 

In the case of NGINX, the Ingress controller is deployed in a pod along with the load balancer.


# Installing the Ingress Controller In Amazom EKS using Helm Charts
# Install helm a package manager for kubernetes
```
 curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
 chmod 700 get_helm.sh
./get_helm.sh

```

## 1. Add helm charts for Nginx Ingress into a server where you have kubectl and helm

Getting the Chart Sources
This step is required if you’re installing the chart using its sources. Additionally, the step is also required for managing the custom resource definitions (CRDs), which the Ingress Controller requires by default, or for upgrading/deleting the CRDs.

Clone the Ingress Controller repo:
```
$ git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v3.1.0
```
## 2. Change your working directory to /deployments/helm-chart:
```
$ cd kubernetes-ingress/deployments/helm-chart
```
## Adding the Helm Repository
This step is required if you’re installing the chart via the helm repository.
```
$ helm repo add nginx-stable https://helm.nginx.com/stable
$ helm repo update
```
## Installing the Chart
Installing the CRDs
By default, the Ingress Controller requires a number of custom resource definitions (CRDs) installed in the cluster. The Helm client will install those CRDs. If the CRDs are not installed, the Ingress Controller pods will not become Ready.

If you do not use the custom resources that require those CRDs (which corresponds to controller.enableCustomResources set to false and controller.appprotect.enable set to false), the installation of the CRDs can be skipped by specifying --skip-crds for the helm install command.

Installing via Helm Repository
To install the chart with the release name my-release (my-release is the name that you choose):

For NGINX:
```
$ helm install my-release nginx-stable/nginx-ingress
```
##  UnDeploy nginx-ingress using helm
 # NB. This will delete nginx-ingress and all associated resources and services
```
  helm uninstall nginx
```
## 3. Verify nginx-ingress is running

```
 helm ls
 kubectl get all
 kubectl get svc
```

##  How Ingress Controller can be deployed

We include two options for deploying the Ingress controller:
 * *Deployment*. Use a Deployment if you plan to dynamically change the number of Ingress controller replicas.
 * *DaemonSet*. Use a DaemonSet for deploying the Ingress controller on every node or a subset of nodes.


##  Check that the Ingress Controller is Running

Check that the Ingress Controller is Running
Run the following command to make sure that the Ingress controller pods are running:
```
 kubectl get pods --namespace=ingress-nginx
```
##  Get Access to the Ingress Controller

 **If you created a daemonset**, ports 80 and 443 of the Ingress controller container are mapped to the same ports of the node where the container is running. To access the Ingress controller, use those ports and an IP address of any node of the cluster where the Ingress controller is running.

### Service with the Type LoadBalancer

A service with the type **LoadBalancer** will be created as well. Kubernetes will allocate and configure a cloud load balancer for load balancing the Ingress controller pods.

**For AWS, run:**
```
kubectl apply -f service/loadbalancer-aws-elb.yaml
```

To get the DNS name of the ELB, run:
```
 kubectl describe svc ingress-nginx --namespace=ingress-nginx
```

`OR`

```
kubectl get svc -n ingress-nginx 
```

You can resolve the DNS name into an IP address using `nslookup`:
```
nslookup <dns-name>
```

# Ingress Resource:

### Define path based or host based routing rules for your services.

### Single DNS Sample with host and servcie place holders
``` yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: <DomainNameOne>
    http:
      paths:
      # Default Backend (Root /)
      - backend:
          serviceName: <serviceName>
          servicePort: 80
``` 

### Multiple DNS Sample with hosts and servcies place holders
``` yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: <DomainNameOne>
    http:
      paths:
      - backend:
          serviceName: <serviceNameOne>
          servicePort: 80
  - host: <DomainNameTwo>
    http:
      paths:
      - backend:
          serviceName: <serviceNamTwo>
          servicePort: 80	
``` 		  

### Path Based Routing Example
``` yaml		  
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: springboot.example.com
    http:
      paths:
      # Default Path(/)
      - backend:
          serviceName: springboot
          servicePort: 80
      - path: /java-web-app
        backend:
          serviceName: javawebapp
          servicePort: 80	
``` 
`Make sure you have services created in K8's with type ClusterIP for your applications. Which your are defining in Ingress Resource`.

## Uninstall the Ingress Controller

 Delete the `ingress-nginx` namespace to uninstall the Ingress controller along with all the auxiliary resources that were created:
 ```
 $ kubectl delete namespace ingress-nginx
 ```

 **Note**: If RBAC is enabled on your cluster and you completed step 2, you will need to remove the ClusterRole and ClusterRoleBinding created in that step:

 ```
  kubectl delete clusterrole ingress-nginx
  kubectl delete clusterrolebinding ingress-nginx
 ```

## Ingress with Https Using Self Signed Certificates:

### Generate self signed certificates
```
 openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out mylandmark-ingress-tls.crt -keyout mylandmark-ingress-tls.key -subj "/CN=javawebapp.javawebapp.landmarkfintech.com/O=mylandmark-ingress-tls"

# Create secret for with your certificate .key & .crt file

 kubectl create secret tls mylandmark-ingress-tls --namespace default --key mylandmark-ingress-tls.key --cert mylandmark-ingress-tls.crt
```
### Mention tls/ssl(certificate) details in ingress
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  tls:
  - hosts:
    - app.landmarkfintech.com
    secretName: mylandmarktech-ingress-tls
  ingressClassName: nginx
  rules:
  - host: app.landmarkfintech.com
    http:
      paths:
      - backend:
          serviceName: springapp
          servicePort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  ingressClassName: nginx
  rules:
  - host: landmarkfintech.com
    secretName: mylandmarktech-ingress-tls
    http:
      paths:
      # Default Path(/)
      - backend:
          serviceName: springapp
          servicePort: 80
      - path: /app
        backend:
          serviceName: webapp
          servicePort: 80
      - path: /java-web-app
        backend:
          serviceName: javawebapp
          servicePort: 80
```




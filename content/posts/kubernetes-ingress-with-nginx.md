---
title: Kubernetes Ingress with NGINX
date: 2019-09-11 13:46:37
---
I've updated my [Git repository](https://github.com/picofish/k8s-vagrant) with an NGINX proxy role. The role installs NGINX on the Kubernetes cluster, but a dedicated VM can do as well. This NGINX will proxy requests to the Kubernetes Ingress in our cluster.

The proxy configuration is as simple as:

```nginx
upstream cluster {
  server 192.168.178.15:30000;
}

server {
  listen 80;
  listen [::]:80;

  server_name kubernetes;

  location / {
    proxy_set_header Host $host;
    proxy_pass http://cluster/;
    proxy_pass_request_headers on;
  }
}
```

The [Kubernetes Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/) is something that can manipulate traffic going to your cluster. Basically, the Ingress resource is the configuration of yet another proxy in front of Kubernetes, which can also expose services without using `kubectl port-forward`.

Your cluster needs something that's called Ingress controller, which is something that in our case will translate to yet another proxy. This ingress controller is in charge of sending the traffic to the correct K8s service. This is different than the proxy deployed with the role mentioned above; that one moves traffic from "outside" inside your cluster.

For this exercise I'm using [NGINX](https://kubernetes.github.io/ingress-nginx/) as a controller, the one maintained by the Kubernetes project.

To deploy the controller you need to run this:

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

The above command will deploy the controller and its dependencies. The controller, however, needs to be reached from outside the internal network of Kubernetes. To do that, I use the `NodePort` service type.

Run this:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      nodePort: 30000
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      nodePort: 30001
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```

After this command you should be able to see ports 30000 and 30001 listening on the master node. Run a `netstat -tulpn` to be sure.

Now, the Ansible role comes into play. The role will send traffic coming on port 80 to port 30000 in the cluster.

So in order to test an ingress, run the following:

+ Create a dummy web app:
```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

+ Expose this deployment through a service:
```shell
kubectl expose deployment nginx-deployment
```

+ Create the ingress:
```yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-deployment-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: nginx-deployment-ingress.dev
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-deployment
          servicePort: 80
```

Note how the ingress will connect to the service. I needed to create a host and add a backend, the Kubernetes service with its port.

So now, if your DNS can translate `nginx-deployment-ingress.dev` to the IP of the NGINX proxy you will be able to connect to the service, pods and therefore to the containers we deployed as a dummy web app.

PS: You can modify the contents of the two dummy web app pods to see actual load-balancing happening.

***
References:

+ [https://www.linode.com/docs/web-servers/nginx/use-nginx-reverse-proxy/](https://www.linode.com/docs/web-servers/nginx/use-nginx-reverse-proxy/)
+ [https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)
+ [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)
+ [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)
+ [https://kubernetes.io/docs/concepts/services-networking/service/#nodeport](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport)
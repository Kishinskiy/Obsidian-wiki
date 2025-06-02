Hello everyone! After facing some problems in my K3S cluster, I decided to change my default ingress and proxy installed in K3S. And so that you don’t face the same problem, I’ll teach you how to install the **Nginx Ingress Controller**, as a proxy in your K3S cluster. If you want to learn how to install a K3S cluster in an automated way using 100% free instances on Oracle Cloud (OCI), please check my GitHub project [here](https://github.com/alessonviana/OCI_Infrastructure).

# A little bit about K3S

K3s is a project by Rancher Labs company, which was released in 2019 and has become popular as a lightweight and easy-to-use alternative to Kubernetes. K3s is a certified Kubernetes distribution, which is designed to run in resource constrained environments such as IoT devices, network edge, and private clouds. It is designed to run with less hardware and software resources than standard Kubernetes while maintaining compatibility with the Kubernetes API.

K3s also includes a number of useful features to make it easier to deploy and manage Kubernetes in production environments, such as automated TLS deployment, built-in storage, and a simplified web dashboard.

The K3s project has been widely embraced by the developer community and enterprises looking for a scalable, easy-to-use container orchestration solution for their applications. You can get more information from K3S official [website](https://k3s.io/).

![[1*R6j4ZiEwA-mKDaTP_c1c_w.webp]]
# Why install Nginx as a proxy?

By default, your K3S cluster will use Traefik as _“Ingress”_ and _“Proxy”_. However, in my experiences using Traefik on K3S, I faced several problems, and after many reports of similar problems in the IT community, I decided to use Nginx Ingress Controller instead of Traefik. **Note:** Keep in your mind that if you want to install your K3S cluster without Traefik, you must pass a parameter before the cluster installation time.

 ```sh 
 Ex: $ export INSTALL_K3S_EXEC="server - no-deploy traefik"
```

You can consume more information in the official documentation of [K3S](https://docs.k3s.io/networking).

# How to install Nginx Ingress Controller on K3s

It is very likely that this version or installation method has changed by the time you are reading this article. So, I advise you to also check the official documentation [here](https://kubernetes.github.io/ingress-nginx/deploy/?ref=blog.thenets.org#bare-metal).

[NOTE] In my case, I'm running my K3S cluster in virtual machines, so this command bellows it's about bare metal way, if you use any cloud provider like AWS, GCP, or Azure, please take a look at the documentation mentioned above.

Install the Nginx Ingress Controller with the following command:

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/baremetal/deploy.yaml

Your Ingress Controller will not have an entry point. So let’s create a Load Balancer to expose the input ports:

```yaml
apiVersion: v1  
kind: Service  
metadata:  
  name: ingress-nginx-controller-loadbalancer  
  namespace: ingress-nginx  
spec:  
  selector:  
    app.kubernetes.io/component: controller  
    app.kubernetes.io/instance: ingress-nginx  
    app.kubernetes.io/name: ingress-nginx  
  ports:  
    - name: http  
      port: 80  
      protocol: TCP  
      targetPort: 80  
    - name: https  
      port: 443  
      protocol: TCP  
      targetPort: 443  
  type: LoadBalancer
  ```

# Creating a test file

In this example, we will create a deployment that will be exposed using the Nginx Ingress Controller.

**Note:** It is important you pay attention to the _“annotation”_ (line 41): **_nginx.ingress.kubernetes.io/ssl-redirect: “false”_**, by default SSL will be used and will generate a missing certificate error.

Another important point to note is the domain name used (line 44). In this example, I’m using the domain _alessonviana.tech_, but you should change it and use your own domain name and point it to the K3S instance node.

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: site  
spec:  
  replicas: 5  
  selector:  
    matchLabels:  
      name: site-nginx-frontend  
  template:  
    metadata:  
      labels:  
        name: site-nginx-frontend  
    spec:  
      containers:  
        - name: blog  
          image: alesson23/site:latest  
          imagePullPolicy: Always  
          ports:  
            - containerPort: 80  
---  
apiVersion: v1  
kind: Service  
metadata:  
  name: site-nginx-service  
spec:  
  ports:  
    - name: http  
      port: 80  
      protocol: TCP  
      targetPort: 80  
  selector:  
    name: site-nginx-frontend  
---  
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: site-nginx-ingress  
  annotations:  
    #app.kubernetes.io/ingress.class: "nginx"  
    nginx.ingress.kubernetes.io/ssl-redirect: "false"  
spec:  
  rules:  
  - host: alessonviana.tech  
    http:  
      paths:  
        - path: /  
          pathType: Prefix  
          backend:  
            service:  
              name: site-nginx-service  
              port:  
                number: 80
```

After creating your file, so you can deploy this example and test your Ingress, use the command:

$ kubectl -f <your_file>.yaml

After applying your manifest, you should be able to access your application through your domain and exposed port.

> Thanks for reading this far!! I hope this article has helped you in some way.  
> You can find this GitHub repository and DockerHub repository below.
> 
> I will be happy to receive your feedback and I will be available to answer your questions.

You can find me at the following links:  
**Portifolio:** [alessonviana.tech](http://alessonviana.tech/)

**E-mail:** alesson.viana@gmail.com  
**LinkedIn:** [https://www.linkedin.com/in/alesson-viana/](https://www.linkedin.com/in/alesson-viana/)
---
title:  "kubernetes: Ingress, Load balancer and Node port"
date:   2020-07-10T21:03:00+0800
tags: [kubernetes, backend]
categories: technique
---

In the kubernetes' world, `Ingress`, `Load balancer` and `Node port` are resources that expose services to public. However, it requires some research to decides how and what to use to fit your needs.

### 1. LoadBalancer
`LoadBalancer` is a **Service**, if you assign a deployment to a `LoadBalancer` type service, that effectly means you are assigning a **external**(outside your cluster) load balancer to it.

If anyone request the **external** load balancer, it will send those traffic to that deployment, and of course, load balance the traffic to each pod in a round robin fashion.

#### Bare metal (in-premise)
If you are deploying the cluster in-premise, you can try to use another way to expose your service since you don't have a load balancer. Or there is a solution for bare metal, checkout [MetalLB](https://github.com/metallb/metallb).

#### Cloud
If you are deploying the cluster in cloud, such as GCP, AWS, etc. You just have to write the service yaml file, and when you apply it to your cluster(`kubectl apply -f`), the cloud provider will automatically give you a load balancer (No, you pay for it). 

Notice that the provisioned load balancer is binding with certain service. For example, if you have 5 services that is `LoadBalancer` type. In GKE, that means you will have to provision 5 load balancers for each services. You will have to pay for those 5 load balancers. A clever way is to set a single reverse proxy (such as `nginx`) as `LoadBalancer`.


### 2. NodePort
`NodePort` is pretty easy to understand. Basically, it exposes a port of your machine (node), with a range 30000-32767 randomly. Every traffic to this port will be send into cluster, just like port-forwarding.

The drawback is obvious, if someone wants to access you service, you will also need to expose your IP address of your machine, the one directly running the app. It might not be a good idea, suggest use it only in development.

### 3. Ingress
Not like `LoadBalancer` and `NodePort`, `Ingress` is not a **service**.
It's type is ... `Ingress` (Okay..).
At first it is hard to grasp for me. The reason is that, to me, I see load balancer, reverse proxy, as a micro service, i.e., I would start a nginx as a service, let it routes the traffic to other services.

On the other hand, `Ingress` is not a service, but it is doing nginx stuff that I think of as a service. That is the point that makes me feel weird at first.

Nevertheless, it might makes the cluster simpler.
If you use `Ingress` correctly, you can avoid configuring the nginx routing stuff. 
You just need to write the routing rule in `Ingress`.

But, `Ingress` under the hood is just a bunch of rules, you need someone to really implement it for you. Who? it's nginx again... :)

There is a nginx ingress controller provide by official.
Follow the step, it will use the ingress yaml file to generate a `nginx.conf`.

So, it is nginx again. Turns out that `Ingress` is a high level routing rules data structure that aims to simplfy the routing logic. If you are familiar with nginx and needs some customization, I will suggest go with nginx as a `LoadBalancer`, and configure the routing by yourself. On the other hand, if your application routing logic is relatively simple, you might find `Ingress` a handy alternative to configure your cluster.

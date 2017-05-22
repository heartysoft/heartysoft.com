
+++
title = "Public ELBs with Private Kubernetes"
description = "How to expose a service running on provate Kubernetes through a public ELB."
tags = [ "blog", "devops", "kubernetes", "aws" ]
date = "2017-05-22 18:17:00.00"
slug = "public-elbs-with-private-kubernetes"
+++

Kubernetes services allow us to address a collection of pods through a stable endpoint. Depending on the service type, this endpoint may be addressable only from within the cluster, from the same network, or even from the public internet. When running on AWS, setting the service type for a service to LoadBalancer will create an AWS ELB, which can be used as the endpoint. Wih a Kubernetes cluster running on public subnets, this just works. However, if your Kubernetes cluster only runs on private subnets (which is an option when spinning up clusters with Kops), then this doesn't work. It creates the service as expecting, but it gets stuck in a pending state, where it fails to create the ELB. Doing a 

```
kubectl describe svc service-name -n namespace-name
``` 

shows the fact that it can't find a subnet to create the load balancer. The fix for this is simple (once you know how!). You just need to tag one or more AWS public subnets with some special tags, which then allows Kubernetes to create the load balancer in one of those subnets. The tags in question depend on the version of Kubernetes. I'm using 1.6, and as such, added the following tags with empty values:

```
kubernetes.io/cluster/my-cluster-name
kubernetes.io/role/elb
```

With these tags in place, I could create a service as follows:

```
kind: Service
apiVersion: v1
metadata:
  name: hellopub
spec:
  selector:
    app: hello-world
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

This created an ELB, and everything ran fine. 

If you're not on Kubernetes 1.6, then I think the following tags will work:

```
KubernetesCluster (value would be the name of your cluster)
kubernetes.io/role/elb
```

This of course means that you can't use the same subnet across clusters.


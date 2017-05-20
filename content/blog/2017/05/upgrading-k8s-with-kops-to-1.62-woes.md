
+++
title = "Upgrading Kubernetes with Kops to 1.6.2"
description = "Upgrading Kubernetes with Kops to 1.6.2 didn't go smoothly. Here's how to fix it."
tags = [ "blog", "devops", "kubernetes" ]
date = "2017-05-20 20:42:30.00"
slug = "upgrading-k8s-with-kops-to-1.6.2-woes"
+++

Kops 1.6.0 was released this week, and I went ahead and ran kop update on one of my 1.5.2 clusters. After the rolling upgrade, it appeared to go well, but the master nodes didn't seem to be ready. I tried various things, but the issues seemed to revolved around three components: kube-dns, weave-net, and dns-controller. 

Fixing kube-dns was simple. I just created an empty ConfigMap named kube-dns in the kube-system namespace:

```
kubectl create configmap -n kube-system kube-dns
```

If you're not running HA masters, it's possibly you might need to do this, and the rolling update would fail without it. In my case, I ran it after.

I also ran kubectl replace -f with the file from [link](https://github.com/kubernetes/kops/blob/5fe50e0764405e892ab297b9f9a22459b6c7c62b/upup/models/cloudup/resources/addons/networking.weave/k8s-1.6.yaml).

These two things fixed most issues. Kubernetes UI was reachable through the public address again (I could access it via kubectl proxy before, but it didn't seem to be reachable using public dns before the fixes. 

There was still one issue; the dns-controller replica set in kube-set was failing to start its solitary pod. It seemed like the replica set was missing some toleration, and as such could not find a suitable node to run on (the upgrade process tainted the potential nodes, so pods without a specific toleration could not run on them). This was quite difficult to track down. @justinsb from the kops slack channel was extremely helpful, and after a deep dive debugging session, we discovered that the toleration was indeed missing. There's an [issue open](https://github.com/kubernetes/kops/issues/2594) on this, and our best guess is that during the upgrade, the nodes did get tainted, and the correct toleration was applied. But then an HA master still running 1.5.2 came along and overwrote the toleration, resulting in the problem; tolerations were done by annotation in 1.5.2, and are a first class concept in 1.6. 

If this is the case, then running

```
kubectl get deployment dns-controller -oyaml -n kube-system
```
will have the appropriate toleration missing in the spec section (it may still be present in the annotations section). 

The fix was to simply edit the namespace:

```
kubectl edit namespace kube-system
```

and remove the line with the addons.k8s.io/dns-controller.addons.k8s.io toleration. As soon as that's done, protokube kicks in and applies the correct annotation, thus fixing the issue. dns-controller will start working again within a few seconds. 

It's worth noting that our applications continued running while all this upgrade malarkey was going on. 

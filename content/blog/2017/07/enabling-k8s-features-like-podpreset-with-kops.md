+++
title = "Enabling K8s Features like PodPreset with Kops"
description = "How to enable features like PodPreset in a Kubernetes cluster using Kops."
tags = [ "blog", "devops", "kubernetes", "aws" ]
date = "2017-06-05 23:14:30.00"
slug = "enabling-k8s-features-like-podpreset-with-kops"
+++

Kubernetes PodPresets are an interesting features. It allows us to define some common things about containers, and reuse them across different things (like deployments). For example, in my current project, we have some Kubernetes secrets relating to FTP connections. We use Airflow for orchestration, and as part of the workflow, sensors running in the Airflow container detect if particular files on FTP have changed. If they have, then a step in the workflow is triggered that runs a different pod in the cluster. Both the Airflow container, and the one that runs the second job need access to the FTP secrets. With PodPresets, we simply define them in a PodPreset, and apply them to the relevant pods with labels. Kubernetes, via the admission controller, adds in the secrets to the relevant containers when the pods are created. From then on, the behaviour is as if the sections specified in the PodPreset were actually part of the pod definitions themselves. You can see how this works [here](https://kubernetes.io/docs/tasks/inject-data-application/podpreset/).

Before our PodPresets work, two preconditions need to be met:

* You have enabled the api type settings.k8s.io/v1alpha1/podpreset
* You have enabled the admission controller PodPreset 

The first is enabled by default on Kubernetes 1.6.x (though the plan seems to be to disable v1alpha1 settings by default for 1.8.x). Admission controller settings however, are specified when starting the kube api server. I noticed there are three kube-apiserver pods in my cluster (in three AZs), and I even tried changing the yaml definition to add PodPreset to the admission control string. The command in the yaml looked like this:

```
/usr/local/bin/kube-apiserver --address=127.0.0.1 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,ResourceQuota ...
```

Attempting to modify the yaml got rejected saying that I'm only allowed to update certain fields. I started looking for the proper way to make such changes in a cluster set up with Kops. 

I tried running

```
kops edit cluster
```
This brought up an editor with the cluster config, only there was no kubeAPIServer segment. At this point, [Antoine Cotten helped out](https://stackoverflow.com/questions/44872442/enabling-kubernetes-podpresets-with-kops/44881754?noredirect=1). I needed to run

```
kops get cluster --full -o=yaml
```

This gave me the current kubeAPIServer section. I copied the whole section of the yaml, and added it to the cluster spec got by running

```
kops edit cluster
```

Once saved, I ran 

```
kops update --yes
kops rolling-update --instance-group master-eu-west-1a --yes
kops rolling-update --instance-group master-eu-west-1b --yes
kops rolling-update --instance-group master-eu-west-1c --yes
```

That got the setting rolled out. 

With all this done, I restarted my pod with the PodSpec applied, and it worked just as expected.




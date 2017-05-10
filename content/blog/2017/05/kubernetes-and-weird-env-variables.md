+++
title = "Kubernetes and Weird Environment Variables"
description = "A kubernetes feature meant some stuff started failing, and how we solved it."
tags = [ "blog", "devops", "kubernetes" ]
date = "2017-05-10 00:27:30.00"
slug = "kubernetes-and-weird-env-vars"
+++

We've started using Kubernetes, and came across an obvious thing that wasn't apparent until we realised what it was. We were setting up a Kafka Schema Registry, and a Spark cluster on Kubernetes. Things seemed to work, except we suddenly saw that they didn't. Digging down, we saw that the Schema Registry was failing during initialisation stating that SCHEMA_REGISTRY_PORT is deprecated. We couldn't figure out what was injecting the environment variable called SCHEMA_REGISTRY_PORT, and why it suddenly started getting injected, having worked fine before. After a bit more digging, it turned out that it only failed when we created our service named "schema-registry". Without the service, it started up fine, with the service, it failed. The environment variable injected also had a value like tcp://10.4.144.27:80. What could this possibly be?

Well, it turns out that if we create a service named "foo-bar", Kubernetes goes through the trouble if injecting an environment variable called FOO_BAR_PORT that has the location of the service as its value. With the cause uncovered, we simply changed the name of our service to "avro-schema-registry", and things ran fine. 

We also had a similar problem with our Spark cluster in kubernetes; we had named the master service "spark-master", and as a result, an environment variable called SPARK_MASTER_PORT got injected. That variable means something to Spark, and it caused the Spark processes to fail. Changing the service name to "spark" fixed the issue. 

It's obvious once you know what's going on, but it's quite easy to get caught out on this one. 
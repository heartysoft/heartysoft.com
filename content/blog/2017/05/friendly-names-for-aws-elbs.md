+++
title = "Friendly Names for AWS ELBS"
description = "How to use friendly names to access AWS ELBS."
tags = [ "blog", "devops", "aws" ]
date = "2017-05-14 17:54:05.00"
slug = "friendly-names-for-aws-elbs"
+++

Elastic Load Balancers (ELBs) in AWS allow us to access load balanced resources via single endpoints. While useful, a common problem is that they have hideous names. A dns name like internal-aaaaaabbbbcc359111e789ac0ac70354142-1111223232.eu-west-1.elb.amazonaws.com isn't the easiest thing in the world to remember, share, or use. We can use AWS Route53 to expose these load balancers with "friendly" names. 

Say, I have a domain, mysite.com registered via Route53 as a public Hosted Zone. You can use Route53 to register a new domain, or transfer an existing domain, or even have an existing domain point to AWS stuff. I won't go into the details of all this here. What I will show is how you can expose a load balancer DNS as a subdomain of your domain. i.e. with my domain already set up in Route53, I'll be able to use kibana.mysite.com, which will do what we need. 

In the Route53 console, select the relevant Hosted Zone, and click Create Record Set. For the parameters, use the following:

```
Name: mysubdomain
Type: A - IPv4 address
Alias: Yes
Alias Target: internal-aaaaaabbbbcc359111e789ac0ac70354142-1111223232.eu-west-1.elb.amazonaws.com
```
Hit create, wait a while, and voila. You should be able to access the target at mysubdomain.mydomain.com.

If you have an ip address instead of a DNS name, just use Alias=No, and add the ip address (or addresses) in the value field. 

This works for both public, and private ELBs obviously. Private ELBs resolve to private ip addresses, so you'll need to be on a network that understands that address in order to access it. For me, I could only access kibana from our office network. So, kibana.mydomain.com would only work in the office, or if I was vpn-ed into the office network.

This might seem quite basic, but dns, route53, etc. isn't something I'd delved into that much before, and googling around didn't seem to help. Hence the "for dummies" post :)

Of course, you'd want to do this via terraform, and what not. But the gist of the steps should be the same. 
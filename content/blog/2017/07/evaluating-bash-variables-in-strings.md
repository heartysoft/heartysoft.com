
+++
title = "Evaluating Bash Variables in Strings"
description = "How to evaluate variables where the name is a string in bash."
tags = [ "blog", "bash", "linux" ]
date = "2017-07-05 23:32:30.00"
slug = "evaluating-bash-variables-in-strings"
+++

This is one of the things I have to look up everytime I attempt it. Say we have the following variables:

```
export KUBE_CA_PEM_TEST="foo="
export MY_ENV="TEST"
```
I would like to get "foo=" by using MY_ENV as the suffix of a string. 

This can be achieved with the following two liner:

```
export my_var=KUBE_CA_PEM_$MY_ENV
export my_value=${!my_var}
```

The same can be done with the following one liner:

```
export my_value=$(eval "echo \$KUBE_CA_PEM_$MY_ENV")
```


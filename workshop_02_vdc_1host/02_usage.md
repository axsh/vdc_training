# Exercise: Start and terminate instances

## Goals

* Learn of to use Wakame-vdc to create and destroy instances on the fly

## Assignment

You should still be logged into the machine labeled `Wakame1`. If not, use your ssh client to log into it.

We are going to use the `mussel` client that we have installed in the last exercise. Run the following command to show all the instances currently on Wakame-vdc.

```
mussel instance index
```

This will show you that there are 0 instances. That is correct. We haven't started any instances yet.

We can also use `mussel` to show the network and machine image we have registered in the previous exercise.

```
mussel network index
mussel image index
```

Before we can start instances, we will have to register an ssh key pair. This key pair will later be used to log into our instances.

There should already be a key pair prepared in your home directory.

* Private key: `~/ssh_key_pair/demo_key.pem`
* Public key: `~/ssh_key_pair/demo_key.pem.pub`

```
mussel ssh_key_pair create --display-name "demo key" --public-key ~/ssh_key_pair/demo_key.pem.pub
```

This should give you output similar to this:

```
---
:id: ssh-hzfltyzd
:account_id: a-shpoolxx
:uuid: ssh-hzfltyzd
:finger_print: 10:8a:e4:3f:cb:83:ae:0e:75:90:0a:32:14:ed:c2:20
:public_key: |
  ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAgEA3A7VjdBHCvWWfQ0hyjaVKBD/TktSot9SJvp5vKzrrohwK4vITgyzLp2S+n4Ujw4h3C5rGh/QE5+YmGGzuHjksTApX5ZkcZtUr2m0Jo4kF2EY9CbvWDd3AXMu9JdaYLm9KquVYaecrSHo3CMOHJdn5IeTPpVMY8Q/X5AlWVcmG4yJexstzodAis/QzSey5WLNP0NalyxYEplkIyqeGGK4FxKNLdTdGZc/mo1gFvjaMaapr4Qi8zqcKzcyJ/eiL2rh2xjndeEU9o1JQX0UeS8S8V0PZaD/jeMBzRGeP3tuqGKn1KPrUW2ElveMD/RcuTQhuHSIy3FnOQ+xCn9xYqYAzOOgCgMlJXCHFgG1iuZhWCZeDxx1h8JWaMj+Ohalzk/vMxndbY4+SsEl2+okWHacmxEX1N9b8fDUv+H/cGK4Hs1D/7X5zh+Jg4wuTO8AyIqK77BuYysho8zqXvFz7qL/TUoNrk/UwcTOZYoC0bdKdNuAuw/hp3txQgx2tRVTY84I+ZfCxgVNthZ1IqKPjUPC82/sihxusvTMD4322k9RvqZaQJrzJWTt0ltAUIxCr592yN7OUN5AsRuUFHrg2IRDOBTWtkvivREbbAOhn52+ZQYXEwqUIRXFvdZX5VY6G+IbPom6vqKaz+IFeBMzKV3EdlHXs5xzyIFiKGF34530PCU= centos@joske
:description: ''
:created_at: 2016-09-02 10:04:25.000000000 Z
:updated_at: 2016-09-02 10:04:25.000000000 Z
:service_type: std
:display_name: demokey
:deleted_at: 
:labels: []
```

Make note of the `id` line. In this example it is `ssh-hzfltyzd` but it will be something else for you. We are going to need this later so write it down.

If you ever forgot, you can use the following command to check it.

```
mussel ssh_key_pair index
```

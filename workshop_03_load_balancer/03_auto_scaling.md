# Exercise: Autoscaling

## Goals

* Learn how to automatically spin up new instances and register them to load balancers at given times.

## Assignment

To achieve auto-scaling in Wakame-vdc, we are going to take a very simple approach. We are going to identify peek times and then use Cron to scale up and down at those times.

First of all, create a load balancer and assign an instance to it.

```
mussel load_balancer create \
  --balance-algorithm leastconn \
  --display-name "scaling_balancer" \
  --instance_port 80 \
  --instance_protocol http \
  --max_connection 1000 \
  --port 80 \
  --protocol http

mussel instance create \
  --cpu-cores 1 \
  --hypervisor openvz \
  --image-id wmi-lbnode1d64 \
  --memory-size 256 \
  --ssh-key-id ssh-hzfltyzd \
  --vifs /tmp/vifs.json \
  --display-name "lbnode1"

# Wait until states are running.
mussel load_balancer show lb-gbe0lluu | grep -e '^:state'
mussel instance show i-dy7rn5cx | grep -e '^:state'
```

Now we will assume that this environment is always up. There is always one web server instance running but on peek times we want to increase the amount of web server instances.

First let's make a script that automatically starts a bunch of instances and another script to automatically shut them down.

# Exercise: Autoscaling

## Goals

* Learn how to automatically spin up new instances and register them to load balancers at given times.

## Assignment

To achieve auto-scaling in Wakame-vdc, we are going to take a very simple approach. We are going to identify peek times and then use Cron to scale up and down at those times.

First of all, create a load balancer and assign an instance to it.

```
echo '{"eth0":{"network":"nw-demo1"}}' > /tmp/vifs.json

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

rm /tmp/vifs.json

# Wait until states are running.
mussel load_balancer show lb-gbe0lluu | grep -e '^:state'
mussel instance show i-dy7rn5cx | grep -e '^:state'
```

Next find out the instance's NIC uuid and register it to the load balancer.

```
mussel load_balancer register lb-gbe0lluu --vifs vif-87nuogob
```

Now we will assume that this environment is always up. There is always one web server instance running but on peek times we want to increase the amount of web server instances.

First let's make a script that automatically starts a bunch of instances and another script to automatically shut them down.

```
#!/bin/bash

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
UUID_FILE="$DIR/instance_uuids.txt"

LB_UUID="lb-gbe0lluu"

for i in 2 3 4; do
  output=$(mussel instance create \
    --cpu-cores 1 \
    --hypervisor openvz \
    --image-id wmi-lbnode1d64 \
    --memory-size 256 \
    --ssh-key-id ssh-hzfltyzd \
    --vifs /tmp/vifs.json \
    --display-name "lbnode$i")

  echo "$output" | grep -e '^:id:' | cut -d ' ' -f2 >> "$UUID_FILE"
done

for uuid in $(cat "$UUID_FILE"); do
  echo "Waiting for running: "$uuid""
  while [[ $(mussel instance show "$uuid" | grep -e '^:state') != ":state: running" ]]; do
    sleep 2
  done
  echo "Instance running: $uuid"

  echo "Registering instance to LB"
  vif=$(mussel instance show "$uuid" | grep ":vif_id:" | cut -d ' ' -f 3)
  mussel load_balancer register "$LB_UUID" --vifs "$vif"
done
```

Let's save this as `/home/centos/autoscale_script/scale_up.sh`.

Next let's also write a script that scales down again.

```
#!/bin/bash

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
UUID_FILE="$DIR/instance_uuids.txt"

LB_UUID="lb-gbe0lluu"

for uuid in $(cat "$UUID_FILE"); do
  vif=$(mussel instance show "$uuid" | grep ":vif_id:" | cut -d ' ' -f 3)
  mussel load_balancer unregister "$LB_UUID" --vifs "$vif"

  mussel instance destroy "$uuid"
done

rm "$UUID_FILE"
```

Let's save this as `/home/centos/autoscale_script/scale_down.sh`.

Now that we have scripts to scale up and down, the only thing left to do is set up a Cron job.

https://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-autotasks.html

Open the file `/etc/crontab` and add entries for the time to scale up and down.

Examples:

```
# Scale up at every X:10 (01:10, 02:10 ... 21:10 ...)
10 * * * * centos /home/centos/autoscale_script/scale_up.sh

# Scale down at every X:20 (01:20, 02:20 ... 21:20 ...)
20 * * * * centos /home/centos/autoscale_script/scale_down.sh


# Scale up every day at 17:00
0 17 * * * centos /home/centos/autoscale_script/scale_up.sh

# Scale down every day at 22:30
30 22 * * * centos /home/centos/autoscale_script/scale_down.sh
```

Every time you change this file, you will have to restart the Cron daemon.

```
sudo service crond restart
```

For more information about scheduling Cron tasks, just google around. It is a very commonly used tool and there is loads of documentation out there.

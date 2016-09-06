# Exercise: Register instances to a load balancer

## Goals

* Learn how to register instances with load balancers

## Assignment

In the previous exercise we should have gotten a load balancer up and running. The next step is to add instances to it.

To check if load balancing is working, we need an instance that has a simple web server running.

```
/opt/axsh/wakame-vdc/dcmgr/bin/vdc-manage

backupobject add \
  --storage-id=bkst-local \
  --uuid=bo-lbnode1d64 \
  --display-name='lbnode.x86_64.openvz.md.raw.tar.gz' \
  --object-key=lbnode.x86_64.openvz.md.raw.tar.gz \
  --size=2147483648 \
  --allocation-size=494090998 \
  --checksum=c3ecd2e0c0952ac629fecb9fd34f1880 \
  --container-format=tgz \
  --description='lbnode.x86_64.openvz.md.raw.tar.gz'

image add local bo-lbnode1d64 \
  --account-id a-shpoolxx \
  --uuid wmi-lbnode1d64 \
  --arch x86_64 \
  --description 'lbnode.x86_64.openvz.md.raw.tar.gz local' \
  --file-format raw \
  --root-device label:root \
  --service-type std \
  --is-public \
  --display-name 'lbnode1d64' \
  --is-cacheable
```

This image has a web server that returns the uuid of the instance. This will allow us to test load balancers.

First let's start two instances of this image. Let's call them instance A and instance B. Notice that this time we are setting the security groups directly in the `/tmp/vifs.json` file.

```
echo '{"eth0":{"network":"nw-demo1","security_groups":["sg-wavx6zxz"]}}' > /tmp/vifs.json

mussel instance create \
  --cpu-cores 1 \
  --hypervisor openvz \
  --image-id wmi-lbnode1d64 \
  --memory-size 256 \
  --ssh-key-id ssh-hzfltyzd \
  --vifs /tmp/vifs.json \
  --display-name "LBNode\ instance\ A"

mussel instance create \
  --cpu-cores 1 \
  --hypervisor openvz \
  --image-id wmi-lbnode1d64 \
  --memory-size 256 \
  --ssh-key-id ssh-hzfltyzd \
  --vifs /tmp/vifs.json \
  --display-name "LBNode\ instance\ B"

rm /tmp/vifs.json
```

Again make note of both of their UUID. In the example they had the following values but it'll be different for you.

* Instance A: i-4qm2geq9
* Instance B: i-8va6g9cr

To register them with a load balancer, we actually need to register their NICs rather than the instance themselves. In the case of an instance with multiple NICs, it is possible to register only one NIC. Therefore we need to find the UUID for the NICs belonging to Instance A and Instance B.

```
mussel instance show i-4qm2geq9 | grep ":vif_id:"
#Output: - :vif_id: vif-auvahnef

mussel instance show i-8va6g9cr | grep ":vif_id:"
#Output: - :vif_id: vif-urd0y302
```

This tells us:

* Instance A NIC: vif-auvahnef
* Instance B NIC: vif-urd0y302

Register them to the load balancer using the UUID from your environment. Remember that the example load balancer UUID was `lb-o5osuocx` but it will be something else in your environment.

```
mussel load_balancer register lb-o5osuocx --vifs vif-auvahnef
mussel load_balancer register lb-o5osuocx --vifs vif-urd0y302
```

Now when you surf to the load balancer's IP address, you will see that requests are being redirected to Instance A and Instance B in turn.

We're done. When we're ready to tear down our environment again, we can do it with these commands.

```
mussel load_balancer unregister lb-o5osuocx --vifs vif-auvahnef
mussel load_balancer unregister lb-o5osuocx --vifs vif-urd0y302

mussel load_balancer destroy lb-o5osuocx

mussel instance destroy i-4qm2geq9
mussel instance destroy i-8va6g9cr
```

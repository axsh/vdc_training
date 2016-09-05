# Exercise: Load balancing

## Goals

* Learn how to set up and use Wakame-vdc's load balancing feature

## Assignment

Wakame-vdc's load balancing feature is actually a special instance running [haproxy](http://www.haproxy.org). This instance will have two NICs that will each be connected to a different network. Those two networks are:

* The instances network

  This is the one we configured last in the previous workshop. All instances are connected to this. The load balancer needs to be connected to it too so it can pass traffic on to the other instances.


* The management network

  This is the network on which RabbitMQ, DCMGR and the other Wakame-vdc components are listening. The load balancer needs to have access to this network because Wakame-vdc will sends it commands over AMQP. For example if a new instance gets registered to this load balancer, Wakame-vdc will automatically apply the correct configuration this way.

![Management network Vs instances network](images/03_01_01_management_vs_instances_network.png)

![LB bridged network setup](images/03_01_02_LB_bridged_setup.png)

The machine image for this special instance was provided on the **Wakame1** VM and if you followed the instructions in [01_installation.md](01_installation.md) correctly, it should be located at `/var/lib/wakame-vdc/images/lb-centos6.6-stud.x86_64.openvz.md.raw.tar.gz`

Register it to the Wakame-vdc database using `vdc-manage`.

```
/opt/axsh/wakame-vdc/dcmgr/bin/vdc-manage
```

We have to register the load balancer in a similar way we registered machine images in the previous workshop. First register the backup object. (= hard drive image)

```
backupobject add \
  --storage-id=bkst-local \
  --uuid=bo-haproxy1d64 \
  --display-name='lb-centos6.6-stud.x86_64.kvm.md.raw.tar.gz' \
  --object-key=lb-centos6.6-stud.x86_64.kvm.md.raw.tar.gz \
  --size=2147483648 \
  --allocation-size=470420388 \
  --checksum=cafbe29e468be992e49b68285bb759db \
  --container-format=tgz \
  --description='lb-centos6.6-stud.x86_64.kvm.md.raw.tar.gz'
```

Next we add the machine image which again means that this backup object is bootable as an instance.

```
image add local bo-haproxy1d64 \
  --account-id a-shpoolxx \
  --uuid wmi-haproxy1d64 \
  --arch x86_64 \
  --description 'lb-centos6.6-stud.x86_64.kvm.md.raw.tar.gz local' \
  --file-format raw \
  --root-device label:root \
  --service-type lb \
  --is-public \
  --display-name 'haproxy1d64' \
  --is-cacheable\
```

Notice that the `service type` is set to `lb` this time. That will tell Wakame-vdc to treat this image as a load balancer.

Next open `/etc/wakame-vdc/dcmgr.conf` in a text editor.

Find the part that looks like this.

```
service_type("lb", "LbServiceType") {
  image_id 'wmi-demolb'
  instances_network 'nw-demo1'
  management_network 'nw-demo8'
  host_node_scheduler :LeastUsage
  storage_node_scheduler :LeastUsage
  network_scheduler :VifsRequestParam
  mac_address_scheduler :ByHostNodeGroup do
    default 'mr-demomacs'

    pair 'hng-shhost', 'mr-range1'
  end

  # Please specify the addresses that can be referenced from within an instance of the load balancer.
  # amqp_server_uri is saved to userdata in instance of the load balancer.
  amqp_server_uri 'amqp://example.com/'
}
```

This is where the configuration for the load balancer is set. Let's walk through it line by line.

```
  image_id 'wmi-demolb'
```

First we have this line. Like we mentioned before, the load balancer is really just another instance. That's why Wakame-vdc needs to know which machine image to use for it. This line tells Wakame-vdc to use the image with uuid `wmi-demolb` when starting a load balancer.

However, there currently is not image with `wmi-demolb` registered. Instead, we have just registered a machine image with uuid `wmi-haproxy1d64`. We have to tell Wakame-vdc to use this one.

Change the line to look like this:

```
  image_id 'wmi-haproxy1d64'
```

Next let's take a look at the following two lines.

```
  instances_network 'nw-demo1'
  management_network 'nw-demo8'
```

These tell Wakame-vdc what networks the load balancer should be connected to. A load balancer always has 2 NICs:

* Management NIC: This NIC is used to connect to RabbitMQ. Wakame-vdc will send AMQP messages to the load balancer to dynamically update the instances registered to it and other configuration.

* Instances NIC: This is the NIC that is used to pass the actual load balanced traffic to instances.

It is possible to set these to the same network but that will mean that all instances will also have access to RabbitMQ and the Wakame-vdc Web API. Only do this if there is no untrusted parties accessing the instances.

Let's add a second network called `nw-manage`

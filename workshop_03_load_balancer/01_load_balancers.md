# Exercise: Load balancing

## Goals

* Learn how to set up and use Wakame-vdc's load balancing feature

## Assignment

Wakame-vdc's load balancing feature is actually a special instance running [haproxy](http://www.haproxy.org).

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

This is where the configuration for the load balancer is set. In the previous step we have registered an image with uuid `wmi-haproxy1d64`. We have to take the following line:

```
  image_id 'wmi-demolb'
```

and change it into:

```
  image_id 'wmi-haproxy1d64'
```

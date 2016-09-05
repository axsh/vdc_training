# Exercise: Load balancing

## Goals

* Learn how to set up and use Wakame-vdc's load balancing feature

## Assignment

Wakame-vdc's load balancing feature is actually a special instance running [haproxy](http://www.haproxy.org).

The machine image for this special instance was provided on the **Wakame1** VM and if you followed the instructions in [01_installation.md](01_installation.md) correctly, it should be located at `/var/lib/wakame-vdc/images/lb-centos6.6-stud.x86_64.openvz.md.raw.tar.gz`

Register it to the Wakame-vdc database using `vdc-manage`.

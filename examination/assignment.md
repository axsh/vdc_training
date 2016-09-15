# Examination

You will be provided this environment.

![Assignment empty](images/e_00_assignment_empty.png)

#### Host 1

* **Eth0** is attached to the **instances** network with IP address `192.168.5.11`.
* **Eth1** is attached to the **management** network with IP address `172.16.5.11`.

#### Host 2

* **Eth0** is attached to the **instances** network with IP address `192.168.5.12`.
* **Eth1** is attached to the **management** network. No IP address set yet.

#### Host 3

* **Eth0** is attached to the **instances** network with IP address `192.168.5.13`.
* **Eth1** is attached to the **management** network. No IP address set yet.

## Assignment

The assignment is to create the following:

![Assignment image](images/e_01_assignment.png)

### Part 1: Install and configure Wakame-vdc

#### Host 1

Install and configure the following Wakame-vdc components.

* Dcmgr
* Collector
* RabbitMQ
* Mysql
* Mussel

#### Host 2

* Install and configure **HVA**

* Set up the bridges so both instances and load balancer can work here.
  - **Br0** with IP address `192.168.5.12` connected to instances network.
  - **Br1** with IP address `172.16.5.12` connected to management network.


#### Host 3

* Install and configure **HVA**

* Set up the bridges so both instances and load balancer can work here.
  - **Br0** with IP address `192.168.5.13` connected to instances network.
  - **Br1** with IP address `172.16.5.13` connected to management network.

#### Hints

* Remember to reboot after installing OpenVz (to load the OpenVz kernel)
* Make sure the default gateway is **192.168.5.1** (not 172.16.5.1)
* dcmgr is on host 1. That means vdc-manage is on host 1
* Run the `scale_up.sh` and `scale_down.sh` scripts manually first to make sure they work
* The instances network is **192.168.5.0/24**. It is **NOT** 192.168.4.0/24.
* The management network is **172.16.5.0/24**. It is **NOT** 172.16.0.0/24.
* Remember how you had to ssh/curl from Manual1 because of the firewall? The thing is you can't get network connectivity form the **host** to the instances. There is no HVA on the *host 1* machine so you can ssh/curl to instances from host 1
* The drawing is correct this time. Eth0 and Eth1 are **not** reversed.

### Part 2: Set up instances

The machine images are provided in `~/images` on Host 2 and Host 3. They are the same images used in the exercises.

The instances do **not** need to be the same as shown on the illustation above. They can be on any HVA.

You are allowed to install and use the GUI if you prefer it but it is not required.

#### Ex 1: Instance and security groups

* Create 1 instance. It can be either lbnode or ubuntu.
* Use security groups to open tcp port 22.

#### Ex 2: Load Balancing

* Create a load balancer with 2 lbnode instances registered. Use tcp port 80 just like the exercises.

#### Ex 3: Auto scaling

- Every day at 17:00 create 3 more instances and register them to the load balancer from Ex 2.
- Every day at 22:00, shut those 3 instances down again.

# Examination

## Assignment

![Assignment image](e_01_assignment.png)

#### Part 1: Install and configure Wakame-vdc

3 hosts will be provided. Install the following in each host.

#### Host 1

* Dcmgr
* Collector
* RabbitMQ
* Mysql
* Mussel

Eth0 is attached to the management network.

#### Host 2

* HVA

Eth0 is connected to instances network
Eth1 is connected to management network

You will need to set up the bridges so both instances and load balancer can work here.

#### Host 3

* HVA

Eth0 is connected to instances network
Eth1 is connected to management network

You will need to set up the bridges so both instances and load balancer can work here.

#### Part 2: Set up instances

The machine images are provided in `~/images` on Host 2 and Host 3. They are the same images used in the exercises.

The instances do **not** need to be the same as shown on the illustation above. They can be on any HVA.

##### Ex 1: Instance and security groups

* Create 1 instance. It can be either lbnode or ubuntu.
* Use security groups to open tcp port 22 and 80.

##### Ex 2: Load Balancing

* Create a load balancer with 2 lbnode instances registered. Use tcp port 80 just like the exercises.

##### Ex 3: Auto scaling

- Every day at 17:00 create 3 more instances and register them to the load balancer from Ex 2.
- Every day at 22:00, shut those 3 instances down again.

# Module 3 - Data Models

In this module, I plan to create a data model, which I will leverage to perform numerous operations.

The activities which I want to address are:  

- Create a data model, abstracted from device configuration(s)
- Develop tools which allow the data model to be vendor agnostic (IOS, Junos, EOS and NXOS )
- Build a data model which can be used to deploy the intent across the network
- Build a data model which can be used to validate against device configuration(s)  

A successful abstraction of the data model will only be deemed successful if the following is met:  
- Ability to successfully replace an existing device with a device from another vendor. For example, replace hostnameX, vendorY with hostnameX, vendorZ
- Ability to add another device to the topology, with the requisite information required for the data model and little need for the operator to understand the underlying syntax of each vendor (within reason)

## Pre-requisites

In order for you to get the best value out of this toolkit, DNS is employed for the resolution of hostnames. Compliance to using DNS names will give you the best experience.

If you run this on a Vagrant Ubuntu host, you will have access to the /etc/hosts file at which point you can add DNS entries for the devices if they don't exist.

Below is an example of my /etc/hosts file:

10.0.0.11 lab-arista-01.lab.dfjt.local  
10.0.0.12 lab-iosv-01.lab.dfjt.local  
10.0.0.13 lab-iosv-02.lab.dfjt.local  
10.0.0.14 lab-nxos-01.lab.dfjt.local  
10.0.0.19 lab-junos-01.lab.dfjt.local  
10.0.0.1 dfjt-r001.lab.dfjt.local  

## Assumptions

The following assumptions are made when using this playbook:  
- There is no existing BGP routing process on these routers, other than the ones specified in the data model
- Management access is already provisioned to the device(s) and has been tested
- Interface loopback 0 is free for use and will be used in the solution


## Playbooks

At present, there are three playbooks which are used for the solution. They are explained in order below:  

### Playbook One - create-data-model.yml

This playbook is used to translate the data model file `fabric-model.yml` into a format which is easier to deploy device configurations from. A focus was made on minimising as much duplicate information as possible when asking the operator to populate this file.  

The destination of this file is in `datamodel/node-model.yml`. This file is overwritten each time it's run, but for your convenience I've uploaded a sample to this repository.  

The playbook uses a Jinja2 template to appropriately take the initial values and render a host friendly data model. I've used the node ID value, plus some Jinja2 techniques to dynamically allocate BGP ASNs and Loopback IP addressing.  

For example, if a node ID is 10, the Jinja2 template will perform the following:  
`bgp_base_asn: 65000` + `node_id: 10` = `node_asn 65010`  

Similarly, if the same node ID of 10 is used, the Jinja2 template will perform the following:  

`loopback_subnet: '192.168.40.0/24'` + `id: 10` = `loopback_ip: 192.168.40.10/32`   

This allows you to simply change the subnet and Base BGP ASN if you would like to deploy the same network at another by simply changing the `bgp_base_asn` and `loopback_subnet` values.

#### Abstraction vs Data Input Decisions

I made the decision to not dynamically populate the routing peer AS numbers so this allows the operator to interconnect to other routing protocols outside the full data model.  

I also decided to not ask the operator to populate the loopback interface number or the name. This is handled statically inside the Jinja2 template. Each vendor names their loopback interfaces differently.  

### Playbook Two - data-model-compare.yml

This playbook performs a few operations.

Firstly, it generates configurations from the data model file `datmodel/node-model.yml` and leverages some vendor specific information in the `group_vars/` directory using Ansible roles and Jinja2 templates and output them to the `configs/compiled/<hostname>` folder.  

There are four roles at present in this playbook and overall solution:

#### Roles

`base`- This role uses the data model and the applicable `{{os}}` Jinja2 template to create a file (`00-base.conf`) containing the hostname, domain-name and timezone across all operating systems.  

`common`- This role uses the data model and the applicable `{{os}}` Jinja2 template to create a file (`05-common.conf`) containing the NTP server(s), DNS server(s) and syslog servers across all operating systems.  

`interfaces`- This role uses the data model and the applicable `{{os}}` Jinja2 template to create a file (`10-interfaces.conf`) containing the interfaces across all operating systems. Some basic standards have been enforced in the data model, namely the interface description.  

`routing` - This role uses the data model and the applicable `{{os}}` Jinja2 template to create a file (`15-interfaces.conf`) containing the routing across all operating systems. BGP has been elected as the routing protocol of choice, however the structure and naming convention would allow one to elect any other routing protocol.  
Each BGP instance is configured with the following standards:
- Loopback0 is the router-id for each device and the example 'customer route', which is subsequently advertised throughout the BGP fabric by using a route-map or export-map  
- Every BGP neighbor session has a password set. In my example, it's 'lab' and it's across all neighbours  
- Every BGP neighbor must contain a description      

I chose BGP as I plan in future to use NAPALM Validate to perform testing to verify the detailed operation of the BGP neighbourships in a subsequent unit testing module of the course.  

The playbook then assembles all the four files generated above into one large file (`assembled.conf`), which is used for the final play of the playbook.

The final play connects to each device and compares the file `assembled.conf` with the configuration on the device and reports the differences into a file called `config-diff`.  
The _napalm_install_config_ module is used to perform this comparision. At this point, you can cease the usage of this project if you only need to report on changes needed to achieve the data model. However, if you want to deploy the changes, the second playbook can deploy these.

### Playbook Three - data-model-deploy.yml

This playbook will take the output of the second playbook file `configs/compiled/<hostname>/config-diff`, use this as the config file and install it onto the applicable device.  
For the sake of auditing purposes, all differences are reported to a file named `configs/deployed/<hostname>-deployed-config-diff`

## Known Issues

Due to the vagaries of various operating systems, I've had to make some trade offs with the solution. Without investing time and resources into a highly complex set of conditional programming, I've decided rather to document these known issues which I've deemed minor enough to not be a functional problem. These are:  

### BGP Authentication - Encrypted Passwords

Depending on the vendor, each password has a different method of encryption type. I have deployed the relevant encryption in the Jinja2 templates so that the `data-model-compare.yml` will not report false positives on these authentication passwords. For your convenience, they are listed below:

| Vendor  | Password Encryption Type|
| ------- |:-----------------------:|
| IOS     | Type 7                  |
| NXOS    | Type 3                  |
| Junos   | AES-256?^               |
| EOS     | Type 0                  |
^ Unsure, tested on version junosv-firefly 12.1X44-D20.3.

There is two ways to get the encrypted passwords. The first and easiest way is to configure a dummy BGP neighbor with your desired clear-text password. Then, show the output and use the encrypted password as the value in your host_vars file.

The second way is to modify the Jinja2 `roles/routing/templates/{{os}}/routing.j2` template to deploy your BGP password in clear-text. If you have password encryption turned on, it will then display the encrypted password for use in your host_vars file.


### IOS - `interfaces` role

The version of IOS I was testing with did not show the `no shutdown` statement in the running configuration with SVIs and Loopback interfaces. I had three options:   

 1) Add the `no shutdown` statement into the interface template, and accept that SVIs and loopbacks will always report that they require changes.  
 2) Add a condition where if the interface name starts with 'Et'(short for 'Ethernet'), add a `no shutdown` statement. In all other instances, do not include the `no shutdown` command. This means any shutdown SVIs or loopbacks will not be enabled by the playbooks.  
 3) Write some crazy multiple if/elif statements to properly deal with this.  
 
 I've already spent an inordinate amount of time on this assignment, so for my purposes I've gone with Option 2. I would like to revisit nested if statements or another graceful way for Option 3 in future.

### IOS - Virtual Devices

Please note there is an existing NAPALM issue against [IOS Virtual devices](https://github.com/napalm-automation/napalm-ansible/issues/145).  
I've linked the GitHub issue so that you can track when this will be resolved. 

As a result, the device `dfjt-r001.lab.dfjt.local - 10.0.0.1` is a physical IOS router that I've used to test my playbooks on.

### ansible.cfg file

I was having issues with my playbooks timing out on appliances. I'm unsure whether there is due to the fact that my lab environment is unable to respond in a timely matter, or that the code is highly inefficient. Either way, there are some entries in the `ansible.cfg` which I configured to get around these.  

`timeout = 180`  
`[persistent_connection]`    
`command_timeout = 180`  

### Data Model - Non point-to-point links

The fabric section of the data model is ideally suited to standard point-to-point links. You will notice in my data model that I have an example where one router `dfjt-r001.lab.dfjt.local` is connected to two routers `lab-iosv-01.lab.dfjt.local` and `lab.iosv-02.lab.dfjt.local` using the same IP address.   
As a result, the playbooks above keep thinking that 'changes' are needed each time I run it. This is a false positive and something to be aware of.  

## Learnings

Reflecting on the learning that I have made between module two and module three, I have been able to upskill my playbooks in the following areas:

- Checking for directories, cleaning up directories and creation of non-existent directories. This allows the playbooks to be more dynamic and graceful.
- Implemented usage of Jinja2 templates, to provide the appropriate level of abstraction for the end user.
- Implemented usage of Ansible roles, which cut the volume of code in the main playbook. In addition, this allows me to reuse these roles in future playbooks, or extend the code to include other roles.
- Trialling the implementation of tags. In the playbooks I've written, I need to consider them again before using them as a reliable function.

It also wasn't all skittles and rainbows. I learned some other valuable lessons which were equally important as well as challenging.

- Jinja2 is useful, but coming from a Python background it's somewhat limited. I was wary of not making crazy templates to cover corner cases. I'm sure the more Jinja2 I use, the more graceful and elegant my solutions will be.
- [Ivan](https://github.com/ipspace/) wasn't mucking around when he said you will throw out your first data model. I threw out a few, tweaked some more and at times wasn't getting anywhere fast. An example of this was I started with the subnet mask as a seperate value but soon realised other platforms used CIDR notation for IP addressing on interfaces. I changed my data model to suit and used the ipaddr module in Jinja2 to properly deal with this.
- Initially, my compare and deploy playbook were one big playbook. Thinking about the tradeoff of accidently deploying changes rather than just checking them,  I decided to split these out. Whilst this may be slower, it's certainly more prudent to give users the option of previewing what will happen.

## Summary

I'm mostly satisfied with the solution, however there are two areas which could do with future improvements:  
- Adding import/export/route-maps to every BGP neighbor would be a great improvement.
- Extensively flesh out the `common` and `base` templates which the desired standards for each vendor.

All in all, I addressed the activites which I wanted to achieve and the solution does meet the levels of abstraction which I had aimed for.



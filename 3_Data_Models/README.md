# Module 3 - Data Models

In this module, I plan to create a data model, which I will leverage to perform numerous operations.

The activities which I want to address are:  

- Create a data model, abstracted from device configuration(s)
- Develop tools which allow the data model to be vendor agnostic (IOS, Junos, EOS and NXOS )
- Build a data model which can be used to deploy the intent across the network
- Build a data model which can be used to validate against device configuration(s)  

A successful abstraction of the data model will only be deemed successful if the following is met:  
- Ability to successfully replace an existing device with a device from another vendor. For example, replace hostnameX, vendorY with hostnameX, vendorZ
- Ability to add another device to the topology, with the requisite information required for the data model and no need for the operator to understand the underlying syntax of each vendor (within reason)

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


## Playbooks

At present, there are two playbooks which are used for the solution. They are explained in order below:  

### Playbook One - data-model-compare.yml

This playbook performs a few operations.

Firstly, it generates configurations from the data model in the respective `host_vars/hostname.yml` and the `group_vars/all.yml` using Ansible roles and Jinja2 templates and output them to the `configs/compiled/<hostname>` folder.  

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

### Playbook Two - data-model-deploy.yml

This playbook will take the output of the first playbook file `configs/compiled/<hostname>/config-diff`, use this as the config file and install it onto the applicable device.  
For the sake of auditing purposes, all differences are reported to a file named `configs/deployed/<hostname>-deployed-config-diff`

## Known Issues

Due to the vagaries of various operating systems, you may see that 'changes' are needed to be performed or deployed when running both  playbooks. Without investing time and resources into a highly complex set of conditional programming, I've decided rather to document these known issues which I've deemed minor enough to not be a functional problem. These are:  

### BGP Authentication - Encrypted Passwords

Depending on the vendor, each password has a different method of encryption type. I have deployed the relevant encryption in the Jinja2 templates so that the `data-model-compare.yml` will not report false positives on these authentication passwords. For your convenience, they are listed below:

| Vendor  | Password Encryption Type|
| ------- |:-----------------------:|
| IOS     | Type 7                  |
| NXOS    | Type 3                  |
| Junos   | AES-256?^               |
| EOS     | Type 0                  |
^ Unsure, tested on version junosv-firefly 12.1X44-D20.3.

There is two ways to get the encrypted passwords. The first and easiest way is to configure a dummy BGP neighbor with your desired clear-text password. Then, use that value in your host_vars file.

The second way is to modify the Jinja2 `roles/routing/templates/{{os}}/routing.j2` template to deploy your BGP password in clear-text. If you have password encryption turned on, it will then display the encrypted password for use in your host_vars file.


### IOS - `interfaces` role

The version of IOS I was testing with did not show the `no shutdown` statement in the running configuration with SVIs and Loopback interfaces. I had three options:   

 1) Eliminate the `no shutdown` statement entirely from the Jinja2 interface template. Run the risk of interfaces in a shutdown state not being enabled when provisioned. 
 2) Add the `no shutdown` statement into the interface template, and accept that SVIs and loopbacks will always report that they require changes.
 3) Write some crazy multiple if/elif statements to properly deal with this.
 
 I've already spent an inordinate amount of time on this assignment, so for my purposes I've gone with option 2.


## Caveats - IOS Virtual Devices

The playbook `data-model-compare.yml` has been tested against all operating systems. Please note there is an existing NAPALM issue against [IOS Virtual devices](https://github.com/napalm-automation/napalm-ansible/issues/145).  
I've linked the GitHub issue so that you can track when this will be resolved. 

As a result, the device `dfjt-r001.lab.dfjt.local - 10.0.0.1` is a physical IOS router that I've used to test my playbooks on.

### Learnings

Reflecting on the learning that I have made between module two and module three, I have been able to upskill my playbooks in the following areas:

- Checking for directories, cleaning up directories and creation of non-existent directories. This allows the playbooks to be more dynamic and graceful.
- Implemented usage of Jinja2 templates, to provide the appropriate level of abstraction for the end user.
- Implemented usage of Ansible roles, which cut the volume of code in the main playbook. In addition, this allows me to reuse these roles in future playbooks, or extend the code to include other roles.
- Trialling the implementation of tags. In the playbooks I've written, I need to consider them again before using them as a reliable function.

Finally, it wasn't all skittles and rainbows. I learned some other valuable lessons which were equally important as well as challenging.

- Jinja2 is useful, but coming from a Python background it's somewhat limited. I was wary of not making crazy templates to cover corner cases. I'm sure the more Jinja2 I use, the more graceful and elegant my solutions will be.
- [Ivan](https://github.com/ipspace/) wasn't mucking around when he said you will throw out your first data model. I threw out a few, tweaked some more and at times wasn't getting anywhere fast. An example of this was I started with the subnet mask as a seperate value but soon realised other platforms used CIDR notation for IP addressing on interfaces. I changed my data model to suit and used the ipaddr module in Jinja2 to properly deal with this.
- Initially, my compare and deploy playbook were one big playbook. Thinking about the tradeoff of accidently deploying changes rather than just checking them,  I decided to split these out. Whilst this may be slower, it's certainly more prudent to give users the option of previewing what will happen.

### Summary

I'm not entirely happy with the solution, as there is some data duplication in my data model, which I would optimise if I was using in production. Also, adding import/export/route-maps to every BGP neighbor would be a great improvement. I would also assign each host an identifier which I could use to prepopulate the BGP ASN and the Loopback IP addressing.

All in all, I addressed the activites which I wanted to achieve and the solution does meet the levels of abstraction which I had aimed for.



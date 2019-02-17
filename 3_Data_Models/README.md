# Module 3 - Data Models

In this module, I plan to create a data model, which I will leverage to perform numerous operations.

The activities which I want to address are:  

- Create a data model, abstracted from device configuration(s)
- Develop tools which allow the data model to be vendor agnostic
- Build a data model which can be used to deploy the intent across the network
- Build a data model which can be used to validate against device configuration(s)

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

## Caveats

The playbook `data-model-compare.yml` has been tested against all operating systems. Please note there is an existing NAPALM issue against [IOS Virtual devices](https://github.com/napalm-automation/napalm-ansible/issues/145). I've linked the GitHub issue so that you can track when this will be resolved. 

As a result, the device `dfjt-r001.lab.dfjt.local - 10.0.0.1` is a physical IOS router that I've used to test my playbooks on.




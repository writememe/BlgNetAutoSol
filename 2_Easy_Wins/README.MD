# Module 2 - Easy Wins
In this module, I've decided to build a toolkit of playbooks for various tasks which are usually involved when first auditing or interacting with a network.

The activities which I wanted to address were:
- Collection of running and startup configurations from device(s), using NAPALM
- Collection of extensive facts about device(s) for offline operations in both JSON and YAML format, using NAPALM
- Collection of basic facts about device(s) for a summary report, using NAPALM
- A multi-vendor compliance tool which can be used to in combination with the group_vars and host_vars files to check compliance against standards.

This toolkit allows you to gather information about the network for common network consulting tasks such as:
- Serial number collation for maintenance renewals
- OS Version search or compliance
- Model version search
- Configuration retrieval for offline analysis

## Pre-requisites
In order for you to get the best value out of this toolkit, DNS is employed for the resolution of hostnames. Whilst there may be some trepidation to this (I certainly rolled my eyes at the thought), compliance to using DNS names will give you the best experience.

If you run this on a Vagrant Ubuntu host, you will have access to the /etc/hosts file at which point you can add DNS entries for the devices if they don't exist.

Below is an example of my /etc/hosts file:

10.0.0.11 lab-arista-01.lab.dfjt.local  
10.0.0.12 lab-iosv-01.lab.dfjt.local  
10.0.0.13 lab-iosv-02.lab.dfjt.local  
10.0.0.14 lab-nxos-01.lab.dfjt.local  
10.0.0.19 lab-junos-01.lab.dfjt.local  

## Playbooks
There are six playbooks in this project, which are broken into three functions and they are described below:

### Fact Gathering
`gather-facts.yml` - Used to gather NAPALM facts about all devices, store them in both JSON and YAML format in the `outputs/facts/` directory using the hostname to name the file. The output of this playbook is typically for programmers to perform more operations with the data.

`collect-basic-facts.yml` - This is the first playbook used to collect basic fact data about device(s) and store them into a text file in the `outputs/facts/info`directory using the hostname to name the file. The output is meant to be more human readable, and something you could give to a non-programmer for their persual.  
`assemble-basic-report.yml` - This is the second playbook, to be used after `collect-basic-facts.yml` has been executed. It retrieves all the text files in the `outputs/facts/info`directory and assembles them into the file `reports/Basic-Facts-Summary-Report.txt` file.

Both these playbooks were split intentionally because there may be times when you don't need all 70 odd devices in one file when you are only after one device.

### Configuration Retrieval
`collect-running-config.yml` - This playbook collects all the running configurations from devices(s) and stores them into a text file based on their hostname `outputs/configs/running`.  
`collect-startup-config.yml` - This playbook collects all the startup configurations from devices(s) and stores them into a text file based on their hostname `outputs/configs/startup`.  

### Compliance

`multi-vendor-compliance.yml` - This playbook is the most complex of the toolkit. I've taken ideas from the IPSpace Sample Compliance Checks, however I've extended functionality to have multi-vendor support. It uses the `ansible_network_os` variable to execute the correct compliance checks against the correct OS system tests in the `tests/` directory. If you enter `tree tests/` from the `ansible` directory, you will get a better idea on how this works.

#### Available Tests

At present, there are four tests written for the playbook to execute against each vendor:
- Check OS Version
- Check Serial Number
- Check Syslog Server
- Check NTP Server

In terms of the tests themselves, they use the ios_command, nxos_command, eos_command and junos_command modules as I wasn't getting far with NAPALM. I believe that it is definitely related to my lack of experience. In the interests of having a working product versus a perfect non-working product, I chose the latter.  

#### Report Results

This playbook performs various compliance checks and reports against non-compliance. If the output file is empty, this means your devices are fully compliant against your data model. The compliance report can found at `reports/Non-Compliance.log`

#### Variable Usage

In order to get the best value out of the compliance check, I would recommend adhering to these guidelines:    
_group_vars/all.yml_ - Define all global standards here. NTP, DNS, Syslog and SNMP are examples of values which should be in here  
_group_vars/XXos.yml_ - i.e ios.yml Define all standards unique to that OS or vendor. OS versions, vendor specific features are examples of values which should be in here  
_host_vars/hostname_ - Define all values which are unique to that device. Serial numbers should ideally be the only device in here, although possibly an ancient router running an approved legacy OS could be in here as an 'override' to the OS standards  

## TODO

Below is a list of things which I would like to complete, time permitting:
- Extending the multi-vendor tests to include DNS, IP domain-names, SNMP, login banners
- Read the YAML and JSON files and produce a text file report with all those facts
- Possibly rewrite my tests using NAPALM, time permitting
- Add Aruba vendor support for the compliance checks playbooks
- Add tags to my tests so that I can just run one or more checks across all devices when using the compliance playbook

## Summary

Going through this process, it took me significant time to work out what was the best way to achieve these outcomes. Putting to use what I have learned so far has been rewarding. I imagine if I was to review this project at the conclusion of the course, there would be areas which I could improve as I learn more skills.

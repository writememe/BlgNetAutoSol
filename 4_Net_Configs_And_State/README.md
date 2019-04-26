# Module 4 - Changing Network Configurations and State #

In this module, I plan to extend the current functionality delivered in [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models) by validating deployments.

The activities I wanted to address are:  
 - Create validation tests to validate the deployment, using NAPALM Validate  
 - Dynamically create and execute validation tests based off the data model
 - Produce detailed individual compliance reports for validation on each host
 - Produce summarised invidivual compliance reports for validation on each host
 - Produce a collated summarised compliance report for all hosts 

The success criteria of addressing this module is:
 - Validate that the FDQN and hostname is configured correctly
 - Validate that interfaces are up and have correct descriptions
 - Validate that IP address on interfaces are configured correctly
 - Validate that BGP neighbors are up, have correct ASNs and correct BGP neighbor descriptions
 - Any changes to the original data model `fabric-model.yml` can be cascaded down to extra or less testing. For example, if a routing link is removed, this should no longer be validated. If an IP address is changed, this should be able to be validated without having to manually intervene or write validation tests.

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


The following assumptions are made when using these playbooks:  
- There is no existing BGP routing process on these routers, other than the ones specified in the data model
- Management access is already provisioned to the device(s) and has been tested
- Interface loopback 0 is free for use and will be used in the solution

## Project Structure

As this is an extension on the existing functionality [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models). 

Below is a diagram which depicts the workflows and dependencies of each playbook:  

![Project Workflow](https://github.com/writememe/BlgNetAutoSol/blob/master/4_Net_Configs_And_State/Module%204%20-%20Change%20Net%20Configs%20and%20State.png)


Starting from the top-left, the description of the appropriate components are described below.  

### fabric-model.yml - Original Data Model

This YAML file contains the customer/operator data model. All intent should be populated into this file, following the example in the repository. From this file, all subsequent tasks are completed.

### create-data-model.yml - Data Model Transformation Playbook

This playbook is used to translate the data model file `fabric-model.yml` into a format which is easier to deploy and validate device configurations from. A focus was made on minimising as much duplicate information as possible when asking the operator to populate this file.  

The destination of this file is in `datamodel/node-model.yml`. For more information, please refer to the relevant section in [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models).

### roles/datamode/templates/per-node.j2 - Data Model Transformation Jinja2 Template

This Jinja2 template is called from the `create-data-model.yml` playbook and performs the transformation of the `fabric-model.yml` to the `datamodel\node-model.yml`.  

### datamodel/node-model.yml - Node-friendly Data Model

This YAML file contains the node-friendly data model. As shown in the picture above, this data model is used by three playbooks to perform various functions.

### data-model-compare.yml and data-model-deploy.yml - Compare and Deploy Playbooks

These playbooks perform comparision and deployment functions for the solution. They utilise the `common`, `base`, `interface` and `routing` roles where applicable. More information on these playbooks can be found in [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models).  

### data-model-validate.yml - Validation and Compliance Reporting Playbook

This playbook is the main new playbook produced for this module. At a high level, it performs three main roles:  

1/ Read the `datamodel/node-model.yml` node friendly data model and generate individual validation files.  
2/ Read the individual validation files created in Step 1 and use the `napalm_validate` module against these validation files.  
3/ Report compliance or non-compliance in three forms; a debug file, an individual compliance report and a collated summary report. 

### roles/validate/templates/per-node-validate.j2 - Validate File Generate Jinja2 Template

This Jinja2 template is called from the `data-model-validate.yml` playbook and performs the transformation of the  `datamodel\node-model.yml` into individual host validation files.

For example, a host named `acme` would have a validation file generated in the location `validate/files/acme/automated-validation.yml`.

### validate/files/{{inventory_hostname}}/automated-validation.yml - Indvidual Host Validation File

The following validation checks are generated from the Jinja2 template:  

#### get_facts

- Check hostname
- Check FQDN

#### get_interfaces

- Check interface is up
- Check interface is enabled
- Check interface description

#### get_interfaces_ip

- Check IP address is correct
- Check prefix length is correct

#### get_bgp_neighbors

- Check BGP peer is present
- Check BGP peer is up
- Check BGP peer is enable
- Check BGP peer local AS
- Check BGP peer remote AS
- Check BGP peer description
- Check BGP router-id

### reports/All-Host-Validation-Report.txt

This file contains a collation of all host compliance reports in a single report. This report is intentionally high-level and quite binary. The reasoning is that generally everything should pass compliance so there is no point hindering the operator with lines and lines of superflous information.

Each host either has passed or failed all it's compliance checks.

Then, additional high-level information indicates which aspect of the validation had failed. For a detailed analysis, refer to the `reports/debug/`directory for the exact reason for non-compliance.

## Known Issues

At the time of writing the known issues in [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models#known-issues) still apply. Other than that, the following known issues were encountered:

### NAPALM Validate - nxos - get_interfaces_ip

At the time of writing, napalm=2.4.0 did not correctly process the `get_interfaces_ip` function for nxos device. Below is a link to the issue raised, which seems to be resolved in the `develop` branch of this version:  

[https://github.com/napalm-automation/napalm/issues/964](https://github.com/napalm-automation/napalm/issues/964)

It's anticipated that this will be resolved in the next release. There is a workaround in the above link.

### NAPALM Validate - nxos - get_bgp_neighbors 

At the time of writing, the underlying `get_bgp_neighbors` function does not process the BGP description configuration correctly. It's currently hard coded to "". To workaround this, the [Validate File Generate Jinja2 Template](https://github.com/writememe/BlgNetAutoSol/tree/master/4_Net_Configs_And_State#rolesdatamodetemplatesper-nodej2---data-model-transformation-jinja2-template) has the following in it:

```
{% for peer in node.peers %}
        {{ peer.ip }}:
{%      if node.os == 'nxos' %}
{%      elif node.os != 'nxos' %}
          description: {{ peer.description }}
```
This sets the validation rules to not check the BGP peer description for nxos devices.

### NAPALM Validate - junos - interfaces

NAPALM validate verifies Junos devices based on `<interface_name>.0`. For example, interface `ge-0/0/1 unit 0` is verified using `ge-0/0/1.0`. 

To workaround this, the [Validate File Generate Jinja2 Template](https://github.com/writememe/BlgNetAutoSol/tree/master/4_Net_Configs_And_State#rolesdatamodetemplatesper-nodej2---data-model-transformation-jinja2-template) has the following in it:

```
- get_interfaces:
{% for interface in node.interfaces %}
{%  if node.os == 'junos' %}
    {{ interface.name }}.0:
{%  elif node.os != 'junos' %}
    {{ interface.name }}:
```
And this as well:
```
- get_interfaces_ip:
{% for interface in node.interfaces %}
{%   if node.os == 'junos' %}
    {{ interface.name }}.0:
{%   elif node.os != 'junos' %}
    {{ interface.name }}:
```
This appends the .0 to the end of the interface name to ensure that the NAPALM Validation works.


## Learnings

Reflecting on the learning that I have made between module three and module four, I have been able to upskill my playbooks in the following areas:

- Conditional Jinja2 templating, to get around inconsistencies in NAPALM module support between vendors.
- Multi-function playbook, which generates validation data and then reads that data to perform another function. Previously, I would have had this in two seperate playbooks.
- Increased of the run_once option, to speed up my playbook without losing functionality.
- Consider and appropriately workflow my project, so it's as efficent as possible.

I learned some other lessons from this module as well:

- Initially, I was trying to manually generate my validation files to use in NAPALM validate. While this worked, it certainly would not be scalable.
- Open source software is great. Kudos to those who support and contribute to projects like NAPALM. I've used paid software with inferior support or response in comparison to this project.
- I struggled with where I should source my data to generate the validation files. I did get an Ansible role `roles/datamodel-validate/templates/all-node-validate.j2` to process the `fabric-model.yml` to generate one big validation file and spent a lot of time on this Jinja2 template. However, I couldn't work out how to split the file, based on {{ inventory_hostname }} which is the value of nodes dictionary. If anyone could work that out or show me how, that would be great. As a result, I reverted to the solution in this Module which still meets the requirements.

## Summary

I'm mostly satisfied with this solution, if time permitted in future I would look to add `state` to the routing block in `fabric-model.yml` and disable/enable BGP neighbors and validate appropriately.

It was already obvious but you can't really deploy configurations without validating it! It's added more robustness to the overall solution and closes the loop of automating solutions.

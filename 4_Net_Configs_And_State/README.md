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

As this is an extension on the existing functionality [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models). Below is a diagram which depicts the workflows and dependencies of each playbook:  

![Project Workflow](https://github.com/writememe/BlgNetAutoSol/blob/module-4-dev-4/4_Net_Configs_And_State/Module%204%20-%20Change%20Net%20Configs%20and%20State.png)

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

Then, additional high-level information indicates which aspect of the validation had failed. For a detailed analysis, refer to the `reports/debug/` for the exact reason for non-compliance..




This assignment in under construction. Refer to [TODO.md](https://github.com/writememe/BlgNetAutoSol/blob/master/4_Net_Configs_And_State/TODO.md) for progress.

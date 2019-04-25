# Module 4 - Change Network Configurations and State

This TODO file is a brain dump of things that I need to work on during the development of my assignment.

## Ideas

Some ideas for validating network configurations and state:

 - [ ] Create a list of validation tests from the `fabric-model.yml` model
 - [x] Use NAPALM facts to retrieve the validation data for now, to use for validation
 - [x] Perform all validation tests using Ansible roles
 - [ ] Write a playbook to break the data on the device, so that you can see test failures
 - [ ] Update the `fabric-model.yml` model to deal with additional field such as state= enabled or disabled  
 - [ ] Work out whether all vendors provide a configuration replace option.


## Tasks

Some tasks which need working on:

- [x] Gather facts for LLDP
- [x] Gather facts for BGP
- [x] Gather facts for interfaces
- [x] Create LLDP validation files for all devices
- [x] Create BGP validation files for all devices
- [x] Create interface validation files for all devices
- [x] Work on node-model to validate-datmodel.yml Jinja2 template

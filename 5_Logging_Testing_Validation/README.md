# Module 5 - Logging, Testing and Validation #

[![Build Status](https://travis-ci.org/writememe/BlgNetAutoSol.svg?branch=master)](https://travis-ci.org/writememe/BlgNetAutoSol)

In this module, I plan to extend the current functionality delivered in [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models) and [Module 4 - Changing Network Configurations and State](https://github.com/writememe/BlgNetAutoSol/blob/master/4_Net_Configs_And_State) by adding logging and continuous integration (CI) testing.

The activities I wanted to address are:  
- Log all configuration changes made to devices
- Create a CI pipeline to ensure all Ansible playbooks are tested before being merged into the `master` branch.

The success criteria of addressing this module is:
- An ability to conditionally log all configuration changes, at the operators request
- Create a CI pipeline which automatically testss annd enforces correct YAML and Ansible best practices

## Logging configuration changes ##

In this module, there is only one playbook which deploys configuration changes which is the `data-model-deploy.yml` playbook.

The playbook has been configured with a `debug` variable interpersed throughout the playbook. To activate the logging of the configuration changes, use the following command:

`ansible-playbook data-model-deploy -e "debug=yes"`

By activating the `debug` variable, the playbook will log all configuration changes in the `debugs/` directory. The formatting of the log files within the directory are as follows:  

*hostname-yy-mm-dd-hh-mm-ss.log*  

For example, a hostname of 'lab-router' which had configuration changes deployed on the 3th of May, 2019 at 19:38:00 would generate the log file of:  

*lab-router-2019-05-30-19-38-00.log*

This allows the operator to repeatedly execute configuration changes over time, yet have a clear audit log of all changes executed with the playbook.

## CI Pipeline ##

This module has a CI pipeline using [Travis CI](https://travis-ci.org/). Given that I have already performed validation testing using NAPALM Validate in [Module 4](https://github.com/writememe/BlgNetAutoSol/blob/master/4_Net_Configs_And_State), this CI pipeline will focus on ensuring all future changes to the project are automatically built and tested, when a pull request is created.

The pipeline has two stages which are described in order below:

### Stage One - yamllint ###

This first stage uses [yamllint](https://github.com/adrienverge/yamllint) to check for syntax validity, key repitition, line length, trailing spaces and identation of the playbooks, being YAML files.

The command which is executed is:  
`yamllint 5_Logging_Testing_Validation/ansible/*data-model*.yml` 

This command ensures that the following playbooks are checked:   
`create-data-model.yml`    
`data-model-compare.yml`  
`data-model-deploy.yml` 
`data-model-validate.yml`  

Most importantly, it doesn't check `fabric-model.yml`. This is intentional as I have deliberately excepted this from yamllint. The main reason is I intentionally want my `fabric-model.yml` file to be readable, rather than cosmetically correct as per yamllint enforcement.

### Stage Two - ansible-lint ###

The second stage uses [ansible-lint](https://github.com/ansible/ansible-lint) to check playbooks for practices and behaviour that could potentially be improved.

The command which is executed is:  
`yamllint 5_Logging_Testing_Validation/ansible/*data-model*.yml`  

This command ensures that the following playbooks are checked:  
`create-data-model.yml`    
`data-model-compare.yml`  
`data-model-deploy.yml` 
`data-model-validate.yml`  

Finally, it doesn't check `fabric-model.yml`, given it's not an Ansible playbook.


This assignment is under construction.

Completed

- Integrated Travis CI pipeline, with two build stages (yamllint and ansible-lint)
- Custom `ansible.cfg` file added for the CI pipeline, so that napalm-ansible modules will pass the ansible-lint requirements.
- Conditional debug output for the `data-model-deploy.yml` playbook so that a running set of logs can be kept if needed when making changes to devices.


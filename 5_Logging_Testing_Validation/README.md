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

`ansible-playbook data-model-deploy.yml -e "debug=yes"`

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

## Known Issues ##

At the time of writing, all the known issues encountered in [Module 3 - Data Models](https://github.com/writememe/BlgNetAutoSol/tree/master/3_Data_Models) and [Module 4 - Changing Network Configurations and State](https://github.com/writememe/BlgNetAutoSol/blob/master/4_Net_Configs_And_State), still apply. The following known issues were encountered in this module.

### Placement of .travis.yml file ###

Due to the way Travis CI works, your `.travis.yml` file must be in the root of your repository directory. Initially, I had it at the root of the `5_Logging_Testing_Validation/ansible/` directory. To migitate this, I have placed at the root of my repository, but have ensured both my `yamllint` and `ansible-lint` stages refer to the specific playbook directory.

### napalm-ansible modules failing ansible-lint ###

When Travis CI, builds the test environment, ansible-lint fails every playbook with napalm-ansible modules because it can't find the  napalm-ansible library or the action plugins. In order to get around this, I have created a custom `ansible.cfg` directory in the root of the repository to give the Travis CI environment the location for these variables. Below is the excerpt of the file:  
```
# Due to the vagaries of the CI pipeline, this ansible config file is present and is purely used for Travis CI testing
[defaults]
library = /home/travis/virtualenv/python3.6.3/lib/python3.6/site-packages/napalm_ansible/modules
action_plugins = /home/travis/virtualenv/python3.6.3/lib/python3.6/site-packages/napalm_ansible/plugins/action
````
You will notice that the library and action_plugins refer the virtualenv created under the user travis which is used by Travis CI.

## Learnings

Using the `--extra-vars` option in Ansible playbooks is a graceful way to toggle debugging on and off on playbooks. Using the date command to ascertain a timestamp for use to appending to a log file name is a nice way of creating an audit trail on debug logs.  

Building a CI pipeline took some significant time to integrate and understand. Initially I got my CI pipeline to pass with only one stage testing a single file using yamllint. Then, I moved to adding a single file in a second stage using ansible-lint. My next approach was to yamllint and ansible-lint all my playbooks locally, then resolve the issues with those playbooks. At this point, I thought that testing all my playbooks using yamllint and ansible-lint would work after that.  

That's when I encountered the napalm-ansible modules issue which took me a significant amount of time to identify and resolve the problem, given that they were working locally and not in Travis CI. 

The best thing about the CI pipeline is that all tests are binary. There is no room for partial success. The worst thing about the CI pipeline is that all tests are binary. These is no rooom for partial failure! If you look through my [build history](https://travis-ci.org/writememe/BlgNetAutoSol/builds), you will gain an insight into how long it took.

## Summary

Adding conditional logging and CI to my overall solution has added another level of reliability and rigour to the solution. Learning how to use and interact with Travis CI has been a worthwhile process. When attempt to scale a git project over multiple developers, CI providers a way to automating testing of the code prior to being pushed to production. It forces you and your colleagues to write better code and have greater confidence when performing proactive testing.


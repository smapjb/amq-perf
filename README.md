# Playbook run performance battery against Red Hat AMQ

This playbook assumes that you have a amq broker installed and running that you want to test against.

The role packages -> default -> main.yml installs the dependencies required to run Maven and the amq performance battery https://github.com/joshdreagan/amqp-perf-test. 

Adjust this to fit your environment. E.g. you may already have a different java installed, you may not need to subscribe the system to get yum access.

## Instructions

I have tried to make this as friendly as possible for use within a locked down environment. One assumption that I have made is that you have access to the RHEL ecosystem - ie you have access to standard rhel yum channels either via satellite or access.redhat.com.

All other dependencies can be overriden to pull from local proxies or self hosted servers. For example in `site.yml` you can override the Maven proxy to point to your corporate Artifactory or Nexus, and you can pull the Josh Dreagon zip archive from whereever you have downloaded it.

### Package install

To run the playbook simply replace this with your inventory. Notice the --limit this confines the playbook to the group client_hosts this is because the playbook has hosts: all ready for execution in Tower.

`ansible-playbook -i inventory  site.yml --limit client_hosts`

You will see that site.yml is very succinct simply override the variables in that file to fit your environment.

The site playbook will install the package dependencies and will install Maven, download Josh Dreagons amqp-perf-test and run a `maven clean install` ready for the battery run.

Note that this playbook works for rhel 7 and rhel 8. On rhel 7 it uses software collections to install Maven. You will need access to the software collections channel.

### Battery

The battery role and playbook is a simplified/generalised version of Josh West's battery playbooks https://github.com/shuawest/amq-perf-battery.git

The idea here is that you override the variables in the battery.yml playbook to define a test run, and which broker to run against.

`ansible-playbook -i inventory  battery.yml --limit client_hosts`

Note that in an Ansible role the default/main.yml has the least order of variable precedence and so acts as a kind of interface definition. To that end you can 'always' look in that file to see what variables you can override and change.

I have included a notes.txt in this repository that has the set of tests that were run in Josh Wests amq-perf-battery so that you can see the diffent scenarios that could be used as input to the battery.yml playbook.

### Inventory

Note that I have used --limit on the playbook run command line. In the case of this project it will expect a inventory like.

```
[client_hosts]
host1
host2

```

Note that writing the playbooks with `hosts: all` and defining groups in this way decouples the inventory from the playbook runs which will be useful when/if we start using Tower/AWX

## TODO

I need to collect the results from the test servers and bring them local

Update the notes with the correct variable names to override.

Maybe think about clean up - I have the luxury of throwing away my infra (cattle)
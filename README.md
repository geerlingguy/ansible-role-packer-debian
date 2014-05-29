# Ansible Role: Packer Debian/Ubuntu Configuration for Vagrant VirtualBox

This role configures Debian/Ubuntu (either minimal or full install) in preparation for it to be packaged as part of a .box file for Vagrant/VirtualBox deployment using [Packer](http://www.packer.io/).

The role may be made more flexible in the future, so it could work with other Linux flavors and/or other Packer builders besides VirtualBox, but I'm currently only focused on VirtualBox, since the main use case right now is developer VMs.

## Requirements

Prior to running this role via Packer, you need to make sure Ansible is installed via a shell provisioner, and that preliminary VM configuration (like adding a vagrant user to the appropriate group and the sudoers file) is complete, generally by using a Kickstart installation file (e.g. `ks.cfg`) with Packer. An example array of provisioners for your Packer .json template would be something like:

    "provisioners": [
      {
        "type": "shell",
        "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E bash '{{.Path}}'",
        "script": "scripts/ansible.sh"
      },
      {
        "type": "ansible-local",
        "playbook_file": "ansible/main.yml",
        "role_paths": [
          "/Users/jgeerling/Dropbox/VMs/roles/geerlingguy.packer-debian",
        ]
      }
    ],

The files should contain, at a minimum:

**scripts/ansible.sh**:

    #!/bin/bash -eux
    # Install Ansible repository and Ansible.
    apt-add-repository ppa:rquillo/ansible
    apt-get update
    apt-get install ansible

**ansible/main.yml**:

    ---
    - hosts: all
      sudo: yes
      gather_facts: yes
      roles:
        - geerlingguy.packer-debian

You might also want to add another shell provisioner to run cleanup, erasing free space using `dd`, but this is not required (it will just save a little disk space in the Packer-produced .box file).

If you'd like to add additional roles, make sure you add them to the `role_paths` array in the template .json file, and then you can include them in `main.yml` as you normally would. The Ansible configuration will be run over a local connection from within the Linux environment, so all relevant files need to be copied over to the VM; configuratin for this is in the template .json file. Read more: [Ansible Local Provisioner](http://www.packer.io/docs/provisioners/ansible-local.html).

## Role Variables

None.

## Dependencies

None.

## Example Playbook

    - hosts: all
      roles:
        - { role: geerlingguy.packer-debian }

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).

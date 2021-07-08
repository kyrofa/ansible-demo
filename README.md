# Demo playbook

**Author:** Kyle Fazzari <kyle@miriamtech.com>

This playbook was written as an Ansible demo. It's meant to be a learning tool, a starting point for building out one's configuration, nothing more.

There are two roles here:

- **common**: Common configuration, things that every instance in the inventory should have
- **postgresql**: Configuration necessary to install and configure a single postgresql database (and associated user)

## Pre-requisites

This demo has two instances in the inventory:

- **production.example.com**: A demo production instance
- **staging.example.com**: A demo staging instance

### Replace hosts with real ones

Of course these domain names are completely bogus. Replace them with IP addresses or domain names that actually represent the infrastructure you have on hand. Note that you can remove staging entirely if you want, it won't break anything.

### Rename `host_vars/*` as necessary

Note that the `host_vars/*` files are discovered based on how they're named in the inventory, so after you have renamed the hosts in the inventory, make sure you also rename (or remove) those files in the same way.

### Ensure key-based SSH authentication to hosts

This playbook assumes that it can SSH into each host using the 'ubuntu' user using one of your SSH keys. Make sure the 'ubuntu' user exists and has your key in `~/.ssh/authorized_keys`.

You can tell this playbook to SSH using a different user by modifying the `host_vars/*` files, but good luck, they're encrypted, and you'll never guess the password unless you keep reading. I have to keep you engaged somehow.

## How to run

Now that you have everything setup properly, run:

    $ ansible-playbook --ask-vault-pass -i inventory playbook.yml

It will prompt you for a password, which is 'password' (no quotes). It will then set each instance up according to which roles are assigned to the groups in the inventory. By default, this will apply both the `common` and `postgresql` roles to both staging and production.

To break that command down:

- **--ask-vault-pass**: Host variables oftentimes include sensitive stuff. In this case, it includes database passwords. As a result, it's a best practice to either keep them outside of your configuration altogether, or encrypt them. We're using the latter method here: each file within `host_vars/` is symmetrically encrypted, which means you need to decrypt them to use them. We can provide that password to ansible in a few ways, but using this option we're saying "just prompt me for it."
- **-i inventory**: By default ansible will use `/etc/ansible/hosts` for your inventory, which is stupid because it tends to be pretty closely related to the playbooks, so I generally ship them alongside the playbooks/roles. As a result, we need to tell it where to look.
- **playbook.yml**: The playbook to execute, which serves the purpose of tying inventory groups to roles.

Anyway, once that command finishes, you can try running it again, and you'll notice that ansible didn't do anything. Ansible tries pretty hard to be idempotent.

SSH into one or both of those machines, and you'll notice that UFW is enabled, allowing SSH:

    $ sudo ufw status

You'll also notice PostgreSQL is installed, and has a database and user just for you:

    $ psql -h localhost example example_user

For the password, see the corresponding vault.

## What is this vault thing?

You can edit each `host_vars/` file using `ansible-vault`:

    $ ansible-vault edit host_vars/production.example.com

It'll prompt you for the password (which is, again, 'password' with no quotes), and open it up in your editor. Once you save, it updates the encrypted file.
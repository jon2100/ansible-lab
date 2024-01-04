# For lab and not production usage

## Environment Setup
5 VMs
ESXi Hypervisor 6.7 2u

 1. 8ball - Bastion RHEL 9 host (2vcpu, 4GB RAM, 40GB thin disk)
 2. aaphub - Ansible Automation Platform Hub RHEL 8 (4vcpu, 8GB Ram, 30GB Thin disk1, 60GB Thin disk2 )
 3. ctrlaap - Ansible Automation Platform Controller RHEL 8 (4vcpu, 8GB RAM,2 30GB thin disk)
 4. pdbapp - Ansible Automation Platform DB *PostgreSQL* RHEL 8 (4vcpu, 8GB RAM, 30GB thin disk1, 200GB thin disk2)
 5. eeaap - Anislbe Automation Platofrom Execution Environment RHEL 8 (4vcpu, 8GB RAM, 50GB thin disk1)

### Setup passwordless login from Bastion to hosts

** Note this can be done with an Ansible playbook, but this is taking it from a fresh build, so Ansible is not setup yet.

Create SSH key pair if one does not exist; this is going to be done as root as the setup will be run as root
- #ssh-keygen (This creates a keypair in ~/.ssh/)
  **Caution: if this is an existing host, it may have id_rsa and id_rsa.pub
- #ssh-copy-id root@host.domain.com

All nodes need to be registered with Red Hat.
This was performed at build time but can be done from the CLI.
  **Ensure the proper entitlements for Bastion and hosts.

Option 1: Online
This can be set in the access.redhat.com
- Subscriptions
  - Systems
    - Subscription (tab)
      - Attach Subscription(button)
      - Find the subscription and select
      - Attach the subscription button at the bottom of the page

Option 2: CLI
From the command line
- ssh into each node
  - $ sudo subscription-manager register -u (username used to login to access.redhat.com)
    ** For some old accounts, you may need to use your RHN login
  - $ sudo subscription-manager refresh
  - $ sudo subscription-manager list --consumed
    ** This should list the AAP entitlements

- On the Bastion, enable the repos and install the AAP installer
  -

### Install the Installer
From the Bastion, run the following as root or sudo
- subscription-manager repos --enable ansible-automation-platform-(version-platform)-rpms
- dnf install -y sudo dnf install --enablerepo=ansible-automation-platform-(version-platform)-rpms ansible-automation-platform-installer

### Install AAP
1. Edit the inventory file located /opt/ansible-automation-platform/installer/inventory sudo or root will be needed.
  - Add correct hosts under the following stanzas
    - [automationcontroller]
    - [execution_nodes]
    - [database]
  - Add correct values for the following settings
    - admin_password='password'
    - pg_host='db host'
    - pg_password=''
    - registry_username=''
    - registry_password=''

    ** For the registry, this information is the user account able to pull packages from registry.redhat.io; for most, it will be the same account used to login to access.redhat.com.
    ** Recommended not to store passwords in code repositories; follow the steps here: https://www.ansible.com/blog/securing-tower-installer-passwords

2. Run the installer
   - from /opt/ansible-automation-platform/installer
     - sudo./setup.sh -e ansible-user=root
     or
     - as root ./setup.sh

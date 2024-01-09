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

### Useful tools to have installed
Bastion:
- bind-utils (nslookup)
- git (git utility)
- net-utils (network tools)
- vim
- openldap-clients (ldapsearch)
- tmux
- bash-completion-noarch
- wget
- sos (for RH support)
- yum-utils
- telnet (port connection check)
- httpd-utils (htpass)
- mtr (network path troubleshooting)

AAP Hosts:
- bind-utils (nslookup)
- vim
- openldap-clients (ldapsearch)
- bash-completion-noarch
- sos (for RH support)
- mtr (network path troubleshooting)

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
    ** Recommended not to store passwords in code repositories; follow the steps here:
    - https://www.ansible.com/blog/securing-tower-installer-passwords

2. Run the installer
   - from /opt/ansible-automation-platform/installer
     - sudo./setup.sh -e ansible-user=root
    ** if needing to add LDAP options
     - sudo /setup.sh -e ansible-user=root -e @ldapextras.yml
     or
     - as root ./setup.sh
    ** if needing to add LDAP options
     - as root ./setup.sh -e @ldapextras.yml

### If setting up LDAP on the HUB
Things to note the following must be set in the inventory file
   - 1automationhub_authentication_backend
   - automationhub_ldap_server_uri
   - automationhub_ldap_bind_dn
   - automationhub_ldap_bind_password
   - automationhub_ldap_user_search_base_dn
   - automationhub_ldap_group_search_base_dn

Other values that can be set in the ldapexteras.yml
  - auth_ldap_user_search_scope= 'SUBTREE'
  - auth_ldap_user_search_filter= '(uid=%(user)s)`\'
  - auth_ldap_group_search_scope= 'SUBTREE'
  - auth_ldap_group_search_filter= '(objectClass=Group)'
  - auth_ldap_group_type_class= 'django_auth_ldap.config:GroupOfNamesType'


If the connection is to AD the following will need to be set for the values in the ldapextras.yml
  - AUTH_LDAP_USER_Search_FILTER: '(sAMAccountName=%(user)s)`
  - AUTH_LDAP_GROUP_TYPE_CLASS: 'django_auth_ldap.config:MemberDNGroupType'
  - AUTH_LDAP_GROUP_TYPE_PARAMS: {"member_attr": "member", "name_attr": "cn"}

For Debug add this to the ldapextras.yml and rerun setup
 - GALAXY_LDAP_LOGGING: True

For Nested Groups
- auth_ldap_group_search_filter = '(objectClass=memberOf:1.2.840.113556.1.4.1941)'

### Example: ldapextras.yml
```
#ldapextras.yml
---
ldap_extra_settings:
  AUTH_LDAP_USER_SEARCH_SCOPE: 'SUBTREE'
  AUTH_LDAP_USER_SEARCH_FILTER: '(sAMAccountName=%(user)s)'
  AUTH_LDAP_GROUP_SEARCH_SCOPE: 'SUBTREE'
  AUTH_LDAP_GROUP_SEARCH_FILTER: '(objectClass=groupOfNames)'
  AUTH_LDAP_GROUP_TYPE_CLASS: "django_auth_ldap.config:MemberDNGroupType"
  AUTH_LDAP_GROUP_TYPE_PARAMS: {"member_attr": "member", "name_attr": "cn"}
  AUTH_LDAP_USER_ATTR_MAP: {"first_name": "givenName", "last_name": "sn", "email": "mail"}
  AUTH_LDAP_USER_FLAGS_BY_GROUP: {"is_superuser": "CN=myadmin,OU=ansibleGroups,OU=Groups,DC=domain,DC=.com",}
  AUTH_LDAP_MIRROR_GROUPS: True
  GALAXY_LDAP_LOGGING: True
```
Reference Links:
- https://www.ansible.com/blog/getting-started-ldap-authentication-in-ansible-tower
- https://access.redhat.com/solutions/3109871?band=se&seSessionId=01e1f7bd-d4a6-480f-920e-f26b1c082b38&seSource=Recommendation&seResourceOriginID=f4674dca-93d3-45cd-88be-7fc1a347d63d
- https://access.redhat.com/solutions/6983916
- https://access.redhat.com/solutions/6977153
- https://access.redhat.com/solutions/6978061
- https://docs.ansible.com/automation-controller/latest/html/administration/ldap_auth.html
- https://django-auth-ldap.readthedocs.io/en/latest/example.
- https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4
- https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4/html/red_hat_ansible_automation_platform_installation_guide/assembly-platform-install-scenario#ref-ldap-config-on-pah_platform-install-scenario
- https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4/html/automation_controller_administration_guide/controller-ldap-authentication.html

### Enable TLS
This lab has a self-signed AD cert so it will need to be imported into both the hub and controller hosts. If it's a default RHEL 8 build, then the cert can go to ```/etc/pki/ca-trust/source/anchor/ad-crt.cer ```

Then change the setting for ```automationhub_ldap_server_uri``` need to be set to ```ldaps``` and ```636``` for the port (change the port if it's not default) and rerun setup

You can set this value ```# custom_ca_cert=/path/to/ca.crt``` and the cert will be imported automatically.
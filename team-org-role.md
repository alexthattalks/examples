## Team/Org/Role Creation

### Prerequisites and Assumptions

The redhat.tower collection is needed for these examples. This is available here:

https://console.redhat.com/ansible/automation-hub/repo/published/ansible/tower/

In our testing, the collection was uncompressed and pushed into SCM at the following location:

- aap_test/collections/ansible_collections/redhat/tower/
- aap_test is the base directory for the playbook

Tested in Ansible Tower 3.8.6 (AAP 1.2)

### Step 1 - create secret file and secret password entry in vault

- Secret file contains the URL, username, and password for the Ansible Tower, so the playbook can authenticate
- Secret file is encrypted and added to SCM
- Secret file password is saved as a Vault credential
- Secret file is referenced in playbook under vars_files
- Secret file credential is added to Job Template to access values at runtime 

#### Secret file contents
```
secret:
  aap_url: example.url
  aap_user: admin
  aap_pass: password123
```
#### Calling Secret file in playbook
```
vars_files:
   - aap_secret.yml
```
### Step 2 - define environment variables

Sets up playbook for API authentication to the Ansible Tower
```
environment:
  TOWER_HOST: "{{ secret.aap_url }}"
  TOWER_USERNAME: "{{ secret.aap_user }}"
  TOWER_PASSWORD: "{{ secret.aap_pass }}"
```
### Step 3 - determine what this playbook should do

This code block can be used to create a Team
```
- name: Create team {{ aap_group }}
  tower_team:
    name: "{{ aap_group }}"
    organization: Default
    state: present
```
This code block can be used to create an Organization 
``` 
- name: Create organization for {{ aap_group }}
  tower_organization:
    name: "{{ aap_group }}"
    state: present
```
This code can be used to add permissions to a Team. This must be run once for each type of permission.
```
- name: Add execute Role to {{ aap_group }}
  tower_role:
    role: execute
    organization: "{{ aap_group }}"
    team: "{{ aap_group }}"
    state: present
```
This code can be used to create a smart inventory to filter based on the group’s server names
```
- name: Create smart inventory for {{ aap_group }}
  tower_inventory:
    kind: smart
    host_filter: <-- specific syntax filter for group's hosts
    organization: Default
    name: "{{ aap_group }}"
    variables: <-- dictionary
    state: present
```
This code can be used to add a team to a smart inventory to give them access to the hosts and the inventory
 ```
 - name: Add use Role to {{ aap_group }} Inventory
   tower_role:
     role: use
     team: "{{ aap_group }}"
     inventory: "{{ aap_group }}"
     state: present
 ```  
### Step 4 - load into tower and setup survey

This example code relies on the aap_group variable which gets filled in via a survey in Ansible Tower, which inputs the AD Group name to perform the tasks.

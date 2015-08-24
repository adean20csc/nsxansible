# NSX for vSphere Ansible Modules

In this repository you will find a number of Ansible modules, written in python, that you can use
to create, update and delete objects in NSX for vSphere.

## requirements

This modules requires that you have a copy of the NSX for vSphere RAML File (RAML based documentation) available.
The latest version of the NSX-v RAML file can be found and downloaded here: http://github.com/yfauser/nsxraml.

Also you need to have the python ```nsxramlclient`` installed. You can install it using pip:
```sh
sudo pip install nsxramlclient
```
More details on this python client for the NSX-v API can be found here: http://github.com/yfauser/nsxramlclient. Also check here if you have problems installing it through pip.

## How to use these modules

All modules are executed on the host that has ```nsxramclient`` installed, and has access to a copy of the NSX RAML File.
This will likely be localhost.
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: False
```
Every module needs an array called ```nsxmanager_spec``` with following parameters supplied:
- The location of the NSX-v RAML file describing the NSX-v API
- The NSX-v Manager to send calls to either as a hostname or IP Address
- The NSX-v Manager username
- The NSX-v Manager password for the user

These parameters are usually placed in a common variables file:

`answerfile.yml`
```yaml
nsxmanager_spec: 
  raml_file: '/raml/nsxvraml/nsxvapiv614.raml'
  host: 'nsxmanager.invalid.org'
  user: 'admin'
  password: 'vmware'
```
`test_logicalswitch.yml`
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: logicalSwitch Operation
    logicalSwitch:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      transportzone: "TZ"
      name: "TestLS"
      controlplanemode: "UNICAST_MODE"
      description: "My Great Logical Switch"
    register: create_logical_switch

  - debug: var=create_logical_switch
```
As you can see in the above example ```nsxmanager_spec``` is taken out of the file ```answerfile.yml```.

## Module specific parameters

Every module has specific parameters that are explained in the following sections

###  Module `logicalSwitch`
##### Create, update and delete logical Switches
- state: 
present or absent, defaults to present
- name: 
Mandatory: Name of the logical switch. Updating the name creates a new switch as it is the unique
identifier
- transportzone: 
Mandatory: Name of the transportzone in which the logical switch is created
- controlplanemode:
Mandatory: Control plane mode for the logical switch. The value can be 'UNICAST_MODE',
'HYBRID_MODE' or 'MULTICAST_MODE'. Default is 'UNICAST_MODE'.
- description:
Optional: Free text description for the logical switch

Example:
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: logicalSwitch Operation
    logicalSwitch:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: absent
      transportzone: "TZ"
      name: "TestLS"
      controlplanemode: "UNICAST_MODE"
      description: "My Great Logical Switch"
    register: create_logical_switch

  #- debug: var=create_logical_switch
```

###  Module `nsxVcRegistration`
##### Registers NSX Manager to VC or changes the registration

- vcenter: 
Hostname or IP Address of the vCenter server to which NSX Manager should be registered to
- vcusername:
Username on VC that should be used to register NSX Manager
- vcpassword:
Password of the VC User used to register NSX Manager
- vccertthumbprint: 
Certificate Thumbprint of vCenter Service

All above values are mandatory

Example:
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: NSX Manager VC Registration
    nsxVcRegistration:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      vcenter: 'testvc'
      vcusername: 'root'
      vcpassword: 'vmware'
      vccertthumbprint: 'D8:E1:1F:C1:AD:F7:BA:08:34:0B:20:63:CE:2B:42:C3:CC:90:20:AE'
    register: register_to_vc

  #  - debug: var=register_to_vc
```

### Module `nsxSsoRegistration`
##### Registers NSX Manager to SSO or changes and deletes the SSO Registration

- state:
present or absent, defaults to present
- sso_lookupservice_url:
SSO Lookup Service url in the format https://ip_or_hostname:7444/lookupservice/sdk
- sso_admin_username: 
Username to register to SSO, usually administrator@vsphere.local
- sso_admin_password:
Password of the SSO User used to register
- sso_certthumbprint: 
Certificate Thumbprint of SSO Service

All above values are mandatory

Example:
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: NSX Manager SSO Registration
    nsxSsoRegistration:
      state: present
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      sso_lookupservice_url: 'https://172.17.100.60:7444/lookupservice/sdk'
      sso_admin_username: 'administrator@vsphere.local'
      sso_admin_password: 'vmware'
      sso_certthumbprint: '33:80:F0:58:DB:D4:59:A2:46:14:83:14:2C:48:C3:29:70:25:BE:31'
    register: register_to_sso

#  - debug: var=register_to_sso
```

### Module `nsxIpPool`
##### Create, update and delete IP Pool in NSX Manager

- state:
present or absent, defaults to present
- name: 
Mandatory: Name of the IP Pool. Updating the name creates a new IP Pool as it is the unique
identifier
- start_ip:
Mandatory: Start IP Address in the Pool
- end_ip:
Mandatory: End IP Address in the Pool
- prefix_length:
Mandatory: Prefix length (Subnet 'Mask' bits) of the IP Pool IP Addresses
- gateway:
Optional: Default Gateway in the IP Pool
- dns_server_1:
Optional: First DNS Server in the Pool
- dns_server_2:
Optional: Second DNS Server in the Pool

Example:
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: Controller IP Pool creation
    nsxIpPool:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      name: 'ansible_controller_ip_pool'
      start_ip: '172.17.100.65'
      end_ip: '172.17.100.67'
      prefix_length: '24'
      gateway: '172.17.100.1'
      dns_server_1: '172.17.100.11'
      dns_server_2: '172.17.100.12'
    register: create_ip_pool

  #- debug: var=create_ip_pool
```

## License

The MIT License (MIT)

Copyright (c) 2015 nsxansible

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.



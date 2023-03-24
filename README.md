# DNSDiag Automated DNS Diagnostics

DNSDiag is a tool for automated DNS diagnostics. It is designed to be used by
network operators and administrators to test the health of their DNS infrastructure,
especially in the context of Anycast and global DNS deployments.

This automation will raise an instance of "Docker on CentOS7" for each region specified
in the variable `vultr_regions` in the file `inventory/group_vars/all/vultr.yml`. It will
then deploy an instance of Babak Farrokhi's containerised [dnsdiag](https://dnsdiag.org/)
toolset, executes a test as specified by the variable `dnsdiag_operation`, against a qname
defined by the variable `dnsdiag_qname` with an rdtype defined by the variable `dnsdiag_rdtype`,
optionally with a specific server defined by the variable `dnsdiag_resolver`, otherwise it will
use the first configured resolver within `/etc/resolv.conf`. Finally, the automation
then destroys the instances created previously after dumping the results to stdout. 

There is plans to integrate with AWX, and to build a Django web interface to display and 
interact with dnsdiag. 

## Requirements

* Ansible 2.14
* Python 3.11
* Vultr API Key
    * This must be connected to an account with a valid payment method.
* Terraform
    * A project path is required to be specified in the file `inventory/group_vars/all/vultr.yml`
    * This must be write-accessible by Ansible, as Ansible will place templated terraform files here. 
* Vultr & Terraform Ansible Collections ( `ansible-galaxy install -r collections/requirements.yml` )

## Usage

1. Clone this repository
2. Configure the file located at `inventory/group_vars/all/vultr.yml`:

```yaml
vultr_api_key: "your_vultr_api_key"
vultr_regions: 
  - lhr
  - mel
terraform_project_path: "./.terraform"
vultr_plan: "vhp-1c-1gb-amd" # 1 vCPU, 1 GB RAM, 25 GB NVMe SSD, AMD, $6/mo / $0.009/hr
vultr_app_id: 17 # Docker on CentOS 7 x64
vultr_ssh_key_id: "<UUID-OF-SSH-KEY-SAVED-ON-VULTR>"
```
3. Run the playbook:

```bash
ansible-playbook -i inventory/ operate-dnsdiag-instances.yml -e dnsdiag_operation=<ping/traceroute> -e dnsdiag_qname=example.com -e dnsdiag_rdtype=A -e dnsdiag_resolver=9.9.9.9 (optional)
```

If you do not want to destroy the instances following the completion of tests (for example, if you 
want to test further without waiting for the systems to reinstantiate), you can create
the Vultr instances by running the following playbook:

```bash
ansible-playbook -i inventory/ create-dnsdiag-instances.yml 
```

You can then run the tests by running the following command:

```bash
ansible-playbook -i inventory/ run-dnsdiag-tests.yml -e dnsdiag_operation=<ping/traceroute> -e dnsdiag_qname=example.com -e dnsdiag_rdtype=A -e dnsdiag_resolver=9.9.9.9 (optional)
```

Finally, you can destroy the instances by running the following playbook:

```bash
ansible-playbook -i inventory/ destroy-dnsdiag-instances.yml
```
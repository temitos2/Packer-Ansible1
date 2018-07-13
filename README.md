

# Create  `AWS AMI` with `Packer`

## Linux `Ubuntu` Machine

### Install Packer

```bash
curl -O https://releases.hashicorp.com/packer/0.11.0/packer_0.11.0_linux_amd64.zip
unzip packer*.zip ; rm -f packer*.zip
chmod +x packer
mv packer /usr/bin/packer.io
```
---

### create Packer File

```bash
touch packer.json
```
```json
{
  "variables": {
    "aws_access_key": "",
    "aws_secret_key": "",
    "aws_region":     "us-east-1",
    "aws_instance_type": "t2.micro",
    "aws_source_ami": "ami-a4dc46db"
  },
  "builders": [
    {
      "type": "amazon-ebs",
      "access_key": "{{user `aws_access_key`}}",
      "secret_key": "{{user `aws_secret_key`}}",
      "region": "{{user `aws_region`}}",
      "source_ami": "{{user `aws_source_ami`}}",
      "instance_type": "{{user `aws_instance_type`}}",
      "ssh_username": "ubuntu",
      "ami_name": "Linux-{{isotime | clean_ami_name}}",
      "tags": {
        "role": "Ubuntu"
      },
      "run_tags": {
        "role": "http"
      }
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "script": "ansible.sh"
    },
    {
      "type": "ansible-local",
      "playbook_file": "playbook.yml"
    }
  ]
}
```

---

### shell provisioner file
```bash
touch ansible.sh
```
```bash
#!/bin/bash -e

sudo apt-get install software-properties-common -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update -y
sudo apt-get install -y ansible
```
---

### ansible provisioner file

```bash
touch playbook.yml
```
```yml
- hosts: all
  sudo: yes
  tasks:

    - name: Installing python3.5
      apt: name=python3.5 state=present
    - name: Installing Apache
      apt: name=apache2 state=present

    - name: Starting nginx on boot
      service: name=apache2 enabled=yes state=started
```

### Run Packer

```bash
packer validate packer.json
```

```bash
packer.io build \
    -var 'aws_access_key=' \
    -var 'aws_secret_key=' \
    -var 'aws_region=us-east-1' \
    -var 'aws_source_ami=ami-a4dc46db' \
    -var 'aws_instance_type=t2.micro' \
    packer.json
```

---

## Windows Machine

### create Packer File

```bash
touch windows.json
```
```json
{
  "variables": {
    "aws_access_key": "",
    "aws_secret_key": "",
    "aws_region":     "us-east-1",
    "aws_instance_type": "t2.micro",
    "aws_source_ami": "ami-b8f3b5c7"
  },
  "builders": [
    {
      "type": "amazon-ebs",
      "access_key": "{{user `aws_access_key`}}",
      "secret_key": "{{user `aws_secret_key`}}",
      "region": "{{user `aws_region`}}",
      "source_ami": "{{user `aws_source_ami`}}",
      "instance_type": "{{user `aws_instance_type`}}",
      "user_data_file":"./ec2-userdata.ps1",
      "communicator": "winrm",
      "winrm_username": "Administrator",
      "winrm_use_ssl": true,
      "winrm_insecure": true,
      "ami_name": "Windows2012-{{isotime | clean_ami_name}}",
      "tags": {
        "role": "windows"
      },
      "run_tags": {
        "role": "windows"
      }
    }
  ]
}
```

---

### Run Packer

```bash
packer.io build \
    -var 'aws_access_key=' \
    -var 'aws_secret_key=' \
    -var 'aws_region=us-east-1' \
    -var 'aws_source_ami=ami-b8f3b5c7' \
    -var 'aws_instance_type=t2.micro' \
    windows.json
```

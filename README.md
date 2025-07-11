
# MY CAPSTONE PROJECT-2
My source files for my Capstone Project are located in the eks-cluster directory on the main branch.
I set up my EKS cluster on Amazon Web Services using Terraform for infrastructure as code.

my main.tf

```sh

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
provider "aws" {
  region = "eu-west-1"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.1"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
}

module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  version         = "~> 19.0"

  cluster_name    = "my-eks-cluster"
  cluster_version = "1.28"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  eks_managed_node_groups = {
    default = {
      instance_types = ["t3.medium"]
      desired_size   = 2
      min_size       = 1
      max_size       = 3
    }
  }
}

```
I initialized the Terraform configuration with terraform init and applied it using terraform apply
```sh
.............
.............
module.eks.module.eks_managed_node_group["default"].aws_eks_node_group.this[0]: Still creating... [2m40s elapsed]
module.eks.module.eks_managed_node_group["default"].aws_eks_node_group.this[0]: Still creating... [2m50s elapsed]
module.eks.module.eks_managed_node_group["default"].aws_eks_node_group.this[0]: Creation complete after 2m59s [id=my-eks-cluster:default-20250711131627437600000011]
Apply complete! Resources: 53 added, 0 changed, 0 destroyed.
```
```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/Capstone-Project-2/eks-cluster 
% aws eks --region eu-west-1 update-kubeconfig --name my-eks-cluster
kubectl get nodes
Added new context arn:aws:eks:eu-west-1:524196012679:cluster/my-eks-cluster to /Users/sgworker/.kube/config
E0711 15:29:24.685234   98135 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://7C36DD3DC5FAC52890F16785DD3ADC2B.gr7.eu-west-1.eks.amazonaws.com/api?timeout=32s\": dial tcp 10.0.2.119:443: i/o timeout"
E0711 15:29:54.688460   98135 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://7C36DD3DC5FAC52890F16785DD3ADC2B.gr7.eu-west-1.eks.amazonaws.com/api?timeout=32s\": dial tcp 10.0.2.119:443: i/o timeout"
```

I wasn't able to connect to the cluster using kubectl, so I had to run the following command:
```sh
aws eks update-cluster-config \
  --region eu-west-1 \
  --name my-eks-cluster \
  --resources-vpc-config endpointPublicAccess=true,endpointPrivateAccess=true
```
This enabled public access to the EKS cluster's API endpoint.



# 15 - Configuration Management with Ansible my work
#### This project is for the Devops Bootcamp module "15-Configuration Management with Ansible" 

I built an automated deployment setup using Ansible for provisioning infrastructure and deploying a Java Gradle application.
This project taught me how to use Ansible for full infrastructure automation, how to push application artifacts to a Nexus repository, and how to deploy applications onto a Kubernetes cluster.
I also gained hands-on experience with provisioning EC2 instances on AWS and setting up local Kubernetes environments with Docker Desktop.
I pushed a Java Gradle application to a Nexus repository. Then, I used Ansible to provision two EC2 instances on AWS. On one instance, I installed Java and Jenkins (as a regular application, not used for automation).
Using Docker Desktop, I set up a local Kubernetes cluster. I then deployed the application to the cluster — again fully automated with Ansible. The entire process from infrastructure provisioning to deployment was handled through Ansible without using Jenkins pipelines.



## 📄 Included PDF Resources



## Evidence / Proof

Here are my notes, work, solutions, and test results for the module **"Configuration Management with Ansible"**:  
👉 [PDF Link to Module Notes & Work](./15-Configuration_Management_with_Ansible.pdf)


All of my notes, work, solutions, and test results can be found in the PDF 11-Kubernetes_on_AWS-EKS.pdf. 
My complete documentation, including all notes and tests from the bootcamp, is available in this repository: https://github.com/Saban39/my_devops-bootcamp-pdf-notes-and-solutions.git



## My notes, work, solutions, and test results for Module "Configuration Management with Ansible"



![Bildschirmfoto 2025-06-27 um 10 06 55](https://github.com/user-attachments/assets/a59c9375-9b49-46ed-8bdc-cd20e2941919)

![Bildschirmfoto 2025-06-27 um 10 06 17](https://github.com/user-attachments/assets/0530aea3-5dd4-4ba5-9a3f-331dae349e12)





<details>
<summary>Solution 1: Build & Deploy Java Artifact </summary>
 <br>
> Use repository: https://gitlab.com/devops-bootcamp3/java-gradle-app

> EXERCISE 1: Build & Deploy Java Artifact
You want to help developers automate deploying a Java application on a remote server directly from their local environment. So you create an Ansible project that builds the java application in the Java-gradle project. Then deploys the built jar artifact to a remote Ubuntu server.

- Developers will execute the Ansible script by specifying their first name as the Linux user which will start the application on a remote server. If the Linux User for that name doesn't exist yet on the remote server, Ansible playbook will create it.

- Also consider that the application may already be running from the previous jar file deployment, so make sure to stop the application and remove the old jar file from the remote server first, before copying and deploying the new one, also using Ansible.

My own GitHub repository that I used: 

![Bildschirmfoto 2025-06-27 um 10 06 17](https://github.com/user-attachments/assets/0530aea3-5dd4-4ba5-9a3f-331dae349e12)

![Bildschirmfoto 2025-06-27 um 10 06 55](https://github.com/user-attachments/assets/a59c9375-9b49-46ed-8bdc-cd20e2941919)



Step 1: In the first step, I created an EC2 instance using the SSH key named ansible_ssh_key.

![Bildschirmfoto 2025-06-24 um 16 33 57](https://github.com/user-attachments/assets/fcc2fa87-a810-4009-b6ff-eacb2c8a921e)

After that, I configured the project in Visual Studio Code.

![Bildschirmfoto 2025-06-24 um 16 33 39](https://github.com/user-attachments/assets/4d3c1598-f7e0-492f-8824-5f1b886dd561)

!!!! fyi: I have used openjdk-17-jdk

my hosts file:

```sh
[web_server]
18.159.51.126

[web_server:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=ansible-ssh-key.pem
linux_user=sgworker
project_dir="/Users/sgworker/Desktop/ansible_exercises/ansible-exercises"
jar_name=build-tools-exercises-1.0-SNAPSHOT.jar

[localhost]
localhost ansible_connection=local


[localhost:vars]
project_dir=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises
jar_name=build-tools-exercises-1.0-SNAPSHOT.jar
```

my ansible.cfg
```sh
[defaults]
host_key_checking = False
inventory = hosts

```
my playbook file build_and_deploy.yaml
```sh
- name: Create Linux user
  hosts: web_server
  gather_facts: true
  become: true
  tasks:
    - name: Create linux user
      user:
        name: "{{ linux_user }}"
        group: adm

- name: Make sure Java is installed
  hosts: web_server
  become: true
  tasks:
    - name: Update apt repo cache
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600

    - name: Add the AdoptOpenJDK APT repository
      apt_repository:
        repo: 'ppa:openjdk-r/ppa'
        state: present

    - name: Install OpenJDK 17
      apt:
        name: openjdk-17-jdk
        state: present
        update_cache: yes

    - name: Set JAVA_HOME environment variable
      lineinfile:
        path: /etc/environment
        line: 'JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64'
        create: yes
        state: present

- name: Build application
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Build jar
      command:
        chdir: "{{ project_dir }}"
        cmd: gradle clean build

- name: Stop the currently running Java application and remove old jar file
  hosts: web_server
  become: true
  tasks:
    - name: Find existing jar file
      find:
        paths: "/home/{{ linux_user }}"
        patterns: "*.jar"
        file_type: file
      register: find_result

    - name: Kill any running Java process
      shell: "ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs -r kill"
      ignore_errors: yes

    - name: Remove existing jar files
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ find_result.files }}"
      when: find_result.files | length > 0

- name: Deploy Java application
  hosts: web_server
  become: true
  tasks:
    - name: Copy jar file to remote server
      copy:
        src: "{{ project_dir }}/build/libs/{{ jar_name }}"
        dest: "/home/{{ linux_user }}/"
        owner: "{{ linux_user }}"
        group: adm
        mode: '0755'

    - name: Start the application as sgworker (without become_user)
      become: true
      shell: "sudo -u {{ linux_user }} nohup java -jar {{ jar_name }} > /home/{{ linux_user }}/app.log 2>&1 &"
      args:
        chdir: "/home/{{ linux_user }}"
      async: 1000
      poll: 0
      register: result

    - name: Debug async result
      debug:
        msg: "{{ result }}"

    - name: Check if Java app is running
      shell: "ps aux | grep java | grep -v grep"
      register: app_status

    - name: Debug app status
      debug:
        msg: "{{ app_status.stdout_lines }}"

```
My ansible output: 
```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook build_and_deploy.yaml
[WARNING]: Found both group and host with same name: localhost

PLAY [Create Linux user] ***************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 18.159.51.126 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [18.159.51.126]

TASK [Create linux user] ***************************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126]

PLAY [Make sure Java is installed] *****************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126]

TASK [Update apt repo cache] ***********************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126]

TASK [Add the AdoptOpenJDK APT repository] *********************************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

TASK [Install OpenJDK 17] **************************************************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

TASK [Set JAVA_HOME environment variable] **********************************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

PLAY [Build application] ***************************************************************************************************************************************************************************************************************************************

TASK [Build jar] ***********************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
changed: [localhost]

PLAY [Stop the currently running Java application and remove old jar file] *************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126]

TASK [Find existing jar file] **********************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126]

TASK [Kill any running Java process] ***************************************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

TASK [Remove existing jar files] *******************************************************************************************************************************************************************************************************************************
changed: [18.159.51.126] => (item={'path': '/home/sgworker/build-tools-exercises-1.0-SNAPSHOT.jar', 'mode': '0755', 'isdir': False, 'ischr': False, 'isblk': False, 'isreg': True, 'isfifo': False, 'islnk': False, 'issock': False, 'uid': 1001, 'gid': 4, 'size': 21951687, 'inode': 262475, 'dev': 51713, 'nlink': 1, 'atime': 1750772653.7992837, 'mtime': 1750772651.4183536, 'ctime': 1750772652.290328, 'gr_name': 'adm', 'pw_name': 'sgworker', 'wusr': True, 'rusr': True, 'xusr': True, 'wgrp': False, 'rgrp': True, 'xgrp': True, 'woth': False, 'roth': True, 'xoth': True, 'isuid': False, 'isgid': False})

PLAY [Deploy Java application] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126]

TASK [Copy jar file to remote server] **************************************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

TASK [Start the application as sgworker (without become_user)] *************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

TASK [Debug async result] **************************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126] => {
    "msg": {
        "ansible_job_id": "j397924602853.8949",
        "changed": true,
        "failed": 0,
        "finished": 0,
        "results_file": "/root/.ansible_async/j397924602853.8949",
        "started": 1
    }
}

TASK [Check if Java app is running] ****************************************************************************************************************************************************************************************************************************
changed: [18.159.51.126]

TASK [Debug app status] ****************************************************************************************************************************************************************************************************************************************
ok: [18.159.51.126] => {
    "msg": [
        "root        8961  0.0  0.7  17552  7040 ?        S    13:48   0:00 sudo -u sgworker nohup java -jar build-tools-exercises-1.0-SNAPSHOT.jar",
        "sgworker    8962 73.4  5.8 2324908 57292 ?       Sl   13:48   0:00 java -jar build-tools-exercises-1.0-SNAPSHOT.jar"
    ]
}

PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
18.159.51.126              : ok=17   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

</details>



<details>
<summary>Solution 2:  Push Java Artifact to Nexus </summary>
 <br>
 
> EXERCISE 2: Push Java Artifact to Nexus

Developers like the convenience of running the application directly from their local dev environment. But after they test the application and see that everything works, they want to push the successful artifact to Nexus repository. So you write a play book that allows them to specify the jar file and pushes it to the team's Nexus repository.

First, I installed my Nexus repository on a test-based VM at IONOS. In the second step, I updated the hosts file and executed my playbook.

![Bildschirmfoto 2025-06-24 um 18 02 50](https://github.com/user-attachments/assets/0a579435-89e2-4565-9fe6-d56038d90614)


my extended hosts file:

```sh
[web_server]
18.159.51.126

[web_server:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=ansible-ssh-key.pem
linux_user=sgworker
project_dir="/Users/sgworker/Desktop/ansible_exercises/ansible-exercises"
jar_name=build-tools-exercises-1.0-SNAPSHOT.jar

[localhost]
localhost ansible_connection=local


[localhost:vars]
project_dir=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises
jar_name=build-tools-exercises-1.0-SNAPSHOT.jar
nexus_url=http://85.215.45.206:8081
nexus_user=sg
nexus_password=*************
artifact_name=sg_devops_java_app
artifact_version=1.0-SNAPSHOT
jar_file_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/build/libs/build-tools-exercises-1.0-SNAPSHOT.jar
```
and my playbook pust-to-nexus.yaml 

```sh
- name: Push to Nexus repo
  hosts: localhost
  gather_facts: False
  tasks:
  - name: Push jar artifact to Nexus repo
    # This protects password from being displayed in task output. Comment out if you want to see the output for debugging
    #no_log: True
    
    uri:
      # Notes on Nexus upload artifact URL:
      # 1 - You can add group name in the url ".../com/my/group/{{ artifact_name }}..."
      # 2 - The file name (my-app-1.0-SNAPSHOT.jar) must match the url path of (.../com/my-app/1.0-SNAPSHOT/my-app-1.0-SNAPSHOT.jar), otherwise it won't work
      # 3 - You can only upload file with SNAPSHOT in the version into the maven-snapshots repo, so naming matters
      url: "{{ nexus_url }}/repository/maven-snapshots/com/my/{{ artifact_name }}/{{ artifact_version }}/{{ artifact_name }}-{{ artifact_version }}.jar"
      
      method: PUT
      src: "{{ jar_file_path }}"
      user: "{{ nexus_user }}"
      password: "{{ nexus_password }}"
      force_basic_auth: yes
      
      # With default "raw" body_format request form is too large, and causes 500 server error on Nexus (Form is larger than max length 200000), So we are setting it to 'json'
      body_format: json
      
      status_code:
      - 201
```

 my playbook output:

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook push-to-nexus.yaml
[WARNING]: Found both group and host with same name: localhost

PLAY [Push to Nexus repo] *********************************************************************************************************************************************************************************************************************************

TASK [Push jar artifact to Nexus repo] ********************************************************************************************************************************************************************************************************************
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
```
![Bildschirmfoto 2025-06-24 um 18 02 27](https://github.com/user-attachments/assets/5097e666-2558-4e1a-bbb1-bfa082705238)

![Bildschirmfoto 2025-06-24 um 18 07 50](https://github.com/user-attachments/assets/8e5691e7-a94c-4191-8bb4-79b3ec435e18)


</details>



<details>
<summary>Solution 3: Install Jenkins on EC2 </summary>
 <br>

> EXERCISE 3: Install Jenkins on EC2

- Your team wants to automate creating Jenkins instances dynamically when needed. So your task is to write an Ansible code that creates a new EC2 server and installs and runs Jenkins on it. It also installs nodejs, npm and docker to be available for Jenkins builds.

- Now your team can use this project to spin up a new Jenkins server with 1 Ansible command.

Step1: In the first step, I created the EC2 Ansible server.

fyi: I added the extra-vars to my hosts file.

```sh
[web_server]
18.159.51.126

[web_server:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=ansible-ssh-key.pem
linux_user=sgworker
project_dir="/Users/sgworker/Desktop/ansible_exercises/ansible-exercises"
jar_name=build-tools-exercises-1.0-SNAPSHOT.jar

[localhost]
localhost ansible_connection=local


[localhost:vars]
project_dir=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises
jar_name=build-tools-exercises-1.0-SNAPSHOT.jar
nexus_url=http://85.215.45.206:8081
nexus_user=sg
nexus_password=*********
artifact_name=sg_devops_java_app
artifact_version=1.0-SNAPSHOT
jar_file_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/build/libs/build-tools-exercises-1.0-SNAPSHOT.jar
aws_region=eu-central-1
subnet_id=subnet-0e9565afb5dd37baf
ami_id=ami-02003f9f0fde924ea
key_name=ansible-ssh-key
ssh_user=ubuntu
ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem
subnet_id_db=subnet-09f493c21d5062bd9
subnet_id_web=subnet-0d621ddaf7eb3d890
docker_user=sg1905
docker_pass=***********
```
an then executed the playbook: 3-provision-jenkins-ec2.yaml

```sh
ansible-playbook 3-provision-jenkins-ec2.yaml
[WARNING]: Found both group and host with same name: localhost

PLAY [Provision Jenkins server] ***************************************************************************************************************************************************************************************************************************

TASK [get vpc_information] ********************************************************************************************************************************************************************************************************************************
[WARNING]: packaging.version Python module not installed, unable to check AWS SDK versions
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Get EC2 instances with Name tag 'jenkins-server'] ***************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "instances": [
            {
                "ami_launch_index": 0,
                "architecture": "x86_64",
                "block_device_mappings": [
                    {
                        "device_name": "/dev/xvda",
                        "ebs": {
                            "attach_time": "2025-06-24T16:42:35+00:00",
                            "delete_on_termination": true,
                            "status": "attached",
                            "volume_id": "vol-0789ae7d274768f0c"
                        }
                    }
                ],
                "boot_mode": "uefi-preferred",
                "capacity_reservation_specification": {
                    "capacity_reservation_preference": "open"
                },
                "client_token": "0dd55ff1e89f4e789daddd2f38710533",
                "cpu_options": {
                    "core_count": 2,
                    "threads_per_core": 1
                },
                "current_instance_boot_mode": "legacy-bios",
                "ebs_optimized": false,
                "ena_support": true,
                "enclave_options": {
                    "enabled": false
                },
                "hibernation_options": {
                    "configured": false
                },
                "hypervisor": "xen",
                "image_id": "ami-092ff8e60e2d51e19",
                "instance_id": "i-0de72f9cd985b560a",
                "instance_type": "t2.medium",
                "key_name": "ansible-ssh-key",
                "launch_time": "2025-06-24T16:42:35+00:00",
                "maintenance_options": {
                    "auto_recovery": "default",
                    "reboot_migration": "default"
                },
                "metadata_options": {
                    "http_endpoint": "enabled",
                    "http_protocol_ipv6": "disabled",
                    "http_put_response_hop_limit": 2,
                    "http_tokens": "required",
                    "instance_metadata_tags": "disabled",
                    "state": "applied"
                },
                "monitoring": {
                    "state": "disabled"
                },
                "network_interfaces": [
                    {
                        "association": {
                            "ip_owner_id": "amazon",
                            "public_dns_name": "ec2-3-69-146-43.eu-central-1.compute.amazonaws.com",
                            "public_ip": "3.69.146.43"
                        },
                        "attachment": {
                            "attach_time": "2025-06-24T16:42:35+00:00",
                            "attachment_id": "eni-attach-03338efacd2ee4fb4",
                            "delete_on_termination": true,
                            "device_index": 0,
                            "network_card_index": 0,
                            "status": "attached"
                        },
                        "description": "",
                        "groups": [
                            {
                                "group_id": "sg-080da33bdf384bb77",
                                "group_name": "default"
                            }
                        ],
                        "interface_type": "interface",
                        "ipv6_addresses": [],
                        "mac_address": "0a:49:31:c1:ee:0d",
                        "network_interface_id": "eni-0f2196a31512c33ed",
                        "operator": {
                            "managed": false
                        },
                        "owner_id": "524196012679",
                        "private_dns_name": "ip-172-31-12-253.eu-central-1.compute.internal",
                        "private_ip_address": "172.31.12.253",
                        "private_ip_addresses": [
                            {
                                "association": {
                                    "ip_owner_id": "amazon",
                                    "public_dns_name": "ec2-3-69-146-43.eu-central-1.compute.amazonaws.com",
                                    "public_ip": "3.69.146.43"
                                },
                                "primary": true,
                                "private_dns_name": "ip-172-31-12-253.eu-central-1.compute.internal",
                                "private_ip_address": "172.31.12.253"
                            }
                        ],
                        "source_dest_check": true,
                        "status": "in-use",
                        "subnet_id": "subnet-0e9565afb5dd37baf",
                        "vpc_id": "vpc-0aef6e3692d08e1df"
                    }
                ],
                "network_performance_options": {
                    "bandwidth_weighting": "default"
                },
                "operator": {
                    "managed": false
                },
                "placement": {
                    "availability_zone": "eu-central-1c",
                    "group_name": "",
                    "tenancy": "default"
                },
                "platform_details": "Linux/UNIX",
                "private_dns_name": "ip-172-31-12-253.eu-central-1.compute.internal",
                "private_dns_name_options": {
                    "enable_resource_name_dns_a_record": false,
                    "enable_resource_name_dns_aaaa_record": false,
                    "hostname_type": "ip-name"
                },
                "private_ip_address": "172.31.12.253",
                "product_codes": [],
                "public_dns_name": "ec2-3-69-146-43.eu-central-1.compute.amazonaws.com",
                "public_ip_address": "3.69.146.43",
                "root_device_name": "/dev/xvda",
                "root_device_type": "ebs",
                "security_groups": [
                    {
                        "group_id": "sg-080da33bdf384bb77",
                        "group_name": "default"
                    }
                ],
                "source_dest_check": true,
                "state": {
                    "code": 16,
                    "name": "running"
                },
                "state_transition_reason": "",
                "subnet_id": "subnet-0e9565afb5dd37baf",
                "tags": {
                    "Name": "jenkins-server",
                    "server": "Jenkins"
                },
                "usage_operation": "RunInstances",
                "usage_operation_update_time": "2025-06-24T16:42:35+00:00",
                "virtualization_type": "hvm",
                "vpc_id": "vpc-0aef6e3692d08e1df"
            }
        ],
        "warnings": [
            "packaging.version Python module not installed, unable to check AWS SDK versions"
        ]
    }
}

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "ansible_facts": {
            "discovered_interpreter_python": "/Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12"
        },
        "changed": false,
        "failed": false,
        "vpcs": [
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "10.0.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-0ba4e5c2ec8dd8e5b",
                        "cidr_block": "10.0.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": false,
                "enable_dns_support": true,
                "id": "vpc-06bb2dee1b5bfd46e",
                "instance_tenancy": "default",
                "is_default": false,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {},
                "vpc_id": "vpc-06bb2dee1b5bfd46e"
            },
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "172.31.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-03d6fba2db0daa6f7",
                        "cidr_block": "172.31.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": true,
                "enable_dns_support": true,
                "id": "vpc-0aef6e3692d08e1df",
                "instance_tenancy": "default",
                "is_default": true,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {},
                "vpc_id": "vpc-0aef6e3692d08e1df"
            },
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "192.168.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-07f0b62edc8d9c48b",
                        "cidr_block": "192.168.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": true,
                "enable_dns_support": true,
                "id": "vpc-0df57cf76051bb1e8",
                "instance_tenancy": "default",
                "is_default": false,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {
                    "Name": "eksctl-my-cluster-cluster/VPC",
                    "alpha.eksctl.io/cluster-name": "my-cluster",
                    "alpha.eksctl.io/cluster-oidc-enabled": "false",
                    "alpha.eksctl.io/eksctl-version": "0.167.0",
                    "aws:cloudformation:logical-id": "VPC",
                    "aws:cloudformation:stack-id": "arn:aws:cloudformation:eu-central-1:524196012679:stack/eksctl-my-cluster-cluster/ed6632d0-2a55-11f0-9ac9-02020b4a74e9",
                    "aws:cloudformation:stack-name": "eksctl-my-cluster-cluster",
                    "eksctl.cluster.k8s.io/v1alpha1/cluster-name": "my-cluster"
                },
                "vpc_id": "vpc-0df57cf76051bb1e8"
            }
        ],
        "warnings": [
            "packaging.version Python module not installed, unable to check AWS SDK versions",
            "Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information."
        ]
    }
}

TASK [Ensure hosts file exists] ***************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [update hosts file] **********************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "backup": "",
        "changed": true,
        "diff": [
            {
                "after": "",
                "after_header": "hosts-jenkins-server (content)",
                "before": "",
                "before_header": "hosts-jenkins-server (content)"
            },
            {
                "after_header": "hosts-jenkins-server (file attributes)",
                "before_header": "hosts-jenkins-server (file attributes)"
            }
        ],
        "failed": false,
        "msg": "line added"
    }
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

hosts-jenkins-server file was automatically created. I used it in the next step as inventory file and executed the 3-install-jenkins-ec2.yaml playbook.
!!!! It wasn't possible to start the Jenkins. It was compiled with Java 17 but the java was in playbook was setto java 11. I set it to java 17.

```sh
# Play Prepare Jenkins Server - with all needed tools, Jenkins, Docker, Nodejs & npm
- name: Prepare server for Jenkins
  hosts: "{{ hostvars['localhost']['ec2_result'].instances[0].public_ip_address }}"
  become: yes
  tasks:
  - name: Install Java 17
    yum:
        name: java-17-amazon-corretto
        update_cache: yes
        state: present
```

```sh

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook -i hosts-jenkins-server 3-install-jenkins-ec2.yaml --extra-vars "aws_region=eu-central-1"

PLAY [Get server ip] **************************************************************************************************************************************************************************************************************************************

TASK [Get public_ip address of the ec2 instance] **********************************************************************************************************************************************************************************************************
ok: [localhost]

PLAY [Prepare server for Jenkins] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 3.69.146.43 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [3.69.146.43]

TASK [Install Java] ***************************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [Install Jenkins Repository] *************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [Import RPM key] *************************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [Install /etc/yum.repos.d/jenkins.repo] **************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [Install Docker] *************************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [Check that nvm installed] ***************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Download installer] *********************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [shell] **********************************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [install node] ***************************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43] => {
    "msg": {
        "changed": true,
        "cmd": "source /root/.nvm/nvm.sh && nvm install 8.0.0 && node --version",
        "delta": "0:00:02.875594",
        "end": "2025-06-24 17:03:50.144150",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-24 17:03:47.268556",
        "stderr": "Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...\n\r                                                                           0.5%\r###################                                                       27.3%\r#####################################################################     96.8%\r######################################################################## 100.0%\nComputing checksum with sha256sum\nChecksums matched!",
        "stderr_lines": [
            "Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...",
            "",
            "                                                                           0.5%",
            "###################                                                       27.3%",
            "#####################################################################     96.8%",
            "######################################################################## 100.0%",
            "Computing checksum with sha256sum",
            "Checksums matched!"
        ],
        "stdout": "Downloading and installing node v8.0.0...\nNow using node v8.0.0 (npm v5.0.0)\nCreating default alias: \u001b[0;32mdefault\u001b[0m \u001b[0;90m->\u001b[0m \u001b[0;32m8.0.0\u001b[0m (\u001b[0;90m->\u001b[0m \u001b[0;32mv8.0.0\u001b[0m)\nv8.0.0",
        "stdout_lines": [
            "Downloading and installing node v8.0.0...",
            "Now using node v8.0.0 (npm v5.0.0)",
            "Creating default alias: \u001b[0;32mdefault\u001b[0m \u001b[0;90m->\u001b[0m \u001b[0;32m8.0.0\u001b[0m (\u001b[0;90m->\u001b[0m \u001b[0;32mv8.0.0\u001b[0m)",
            "v8.0.0"
        ]
    }
}

PLAY [Start Jenkins] **************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Start Jenkins server] *******************************************************************************************************************************************************************************************************************************
fatal: [3.69.146.43]: FAILED! => {"changed": false, "msg": "Unable to start service jenkins: Job for jenkins.service failed because the control process exited with error code.\nSee \"systemctl status jenkins.service\" and \"journalctl -xeu jenkins.service\" for details.\n"}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
3.69.146.43                : ok=12   changed=8    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
localhost  

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ssh ec2_user@3.69.146.43 -i ansible-ssh-key.pem 
ec2_user@3.69.146.43: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ssh ec2-user@3.69.146.43 -i ansible-ssh-key.pem
X11 forwarding request failed on channel 0
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Tue Jun 24 17:03:53 2025 from 94.114.29.188
[ec2-user@ip-172-31-12-253 ~]$ 
[ec2-user@ip-172-31-12-253 ~]$ 
[ec2-user@ip-172-31-12-253 ~]$ 
[ec2-user@ip-172-31-12-253 ~]$ 
[ec2-user@ip-172-31-12-253 ~]$ systemctl status jenkins.service
× jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/usr/lib/systemd/system/jenkins.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2025-06-24 17:03:57 UTC; 1min 46s ago
    Process: 30618 ExecStart=/usr/bin/jenkins (code=exited, status=1/FAILURE)
   Main PID: 30618 (code=exited, status=1/FAILURE)
        CPU: 538ms

Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Scheduled restart job, restart counter is at 5.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Stopped jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Start request repeated too quickly.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.
[ec2-user@ip-172-31-12-253 ~]$ sudo journalctl -u jenkins.service --no-pager
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Starting jenkins.service - Jenkins Continuous Integration Server...
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30449]: Running with Java 11 from /usr/lib/jvm/java-11-amazon-corretto.x86_64, which is older than the minimum required version (Java 17).
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30449]: Supported Java versions are: [17, 21]
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30449]: See https://jenkins.io/redirect/java-support/ for more information.
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Main process exited, code=exited, status=1/FAILURE
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Scheduled restart job, restart counter is at 1.
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Stopped jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:54 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Starting jenkins.service - Jenkins Continuous Integration Server...
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30504]: Running with Java 11 from /usr/lib/jvm/java-11-amazon-corretto.x86_64, which is older than the minimum required version (Java 17).
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30504]: Supported Java versions are: [17, 21]
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30504]: See https://jenkins.io/redirect/java-support/ for more information.
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Main process exited, code=exited, status=1/FAILURE
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Scheduled restart job, restart counter is at 2.
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Stopped jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:55 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Starting jenkins.service - Jenkins Continuous Integration Server...
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30542]: Running with Java 11 from /usr/lib/jvm/java-11-amazon-corretto.x86_64, which is older than the minimum required version (Java 17).
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30542]: Supported Java versions are: [17, 21]
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30542]: See https://jenkins.io/redirect/java-support/ for more information.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Main process exited, code=exited, status=1/FAILURE
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Scheduled restart job, restart counter is at 3.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Stopped jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Starting jenkins.service - Jenkins Continuous Integration Server...
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30580]: Running with Java 11 from /usr/lib/jvm/java-11-amazon-corretto.x86_64, which is older than the minimum required version (Java 17).
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30580]: Supported Java versions are: [17, 21]
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30580]: See https://jenkins.io/redirect/java-support/ for more information.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Main process exited, code=exited, status=1/FAILURE
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:56 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Scheduled restart job, restart counter is at 4.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Stopped jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Starting jenkins.service - Jenkins Continuous Integration Server...
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30618]: Running with Java 11 from /usr/lib/jvm/java-11-amazon-corretto.x86_64, which is older than the minimum required version (Java 17).
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30618]: Supported Java versions are: [17, 21]
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal jenkins[30618]: See https://jenkins.io/redirect/java-support/ for more information.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Main process exited, code=exited, status=1/FAILURE
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Scheduled restart job, restart counter is at 5.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Stopped jenkins.service - Jenkins Continuous Integration Server.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Start request repeated too quickly.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: jenkins.service: Failed with result 'exit-code'.
Jun 24 17:03:57 ip-172-31-12-253.eu-central-1.compute.internal systemd[1]: Failed to start jenkins.service - Jenkins Continuous Integration Server.



% ssh ec2-user@3.69.146.43 -i ansible-ssh-key.pem                                                           
X11 forwarding request failed on channel 0
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Tue Jun 24 17:10:31 2025 from 94.114.29.188
[ec2-user@ip-172-31-12-253 ~]$ java -version 
openjdk version "17.0.15" 2025-04-15 LTS
OpenJDK Runtime Environment Corretto-17.0.15.6.1 (build 17.0.15+6-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.15.6.1 (build 17.0.15+6-LTS, mixed mode, sharing)


% ssh ec2-user@3.69.146.43 -i ansible-ssh-key.pem
X11 forwarding request failed on channel 0
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Tue Jun 24 17:10:49 2025 from 94.114.29.188
^C
-bash-5.2$ exit
logout
Connection to 3.69.146.43 closed.
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook -i hosts-jenkins-server 3-install-jenkins-ec2.yaml --extra-vars "aws_region=eu-central-1"

PLAY [Get server ip] **************************************************************************************************************************************************************************************************************************************

TASK [Get public_ip address of the ec2 instance] **********************************************************************************************************************************************************************************************************
ok: [localhost]

PLAY [Prepare server for Jenkins] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 3.69.146.43 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [3.69.146.43]

TASK [Install Java 17] ************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Install Jenkins Repository] *************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Import RPM key] *************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Install /etc/yum.repos.d/jenkins.repo] **************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Install Docker] *************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Check that nvm installed] ***************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Download installer] *********************************************************************************************************************************************************************************************************************************
skipping: [3.69.146.43]

TASK [shell] **********************************************************************************************************************************************************************************************************************************************
skipping: [3.69.146.43]

TASK [install node] ***************************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43] => {
    "msg": {
        "changed": true,
        "cmd": "source /root/.nvm/nvm.sh && nvm install 8.0.0 && node --version",
        "delta": "0:00:01.209492",
        "end": "2025-06-24 17:11:46.385623",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-24 17:11:45.176131",
        "stderr": "v8.0.0 is already installed.",
        "stderr_lines": [
            "v8.0.0 is already installed."
        ],
        "stdout": "Now using node v8.0.0 (npm v5.0.0)\nv8.0.0",
        "stdout_lines": [
            "Now using node v8.0.0 (npm v5.0.0)",
            "v8.0.0"
        ]
    }
}

PLAY [Start Jenkins] **************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [Start Jenkins server] *******************************************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [Wait 10 seconds to check the Jenkins port] **********************************************************************************************************************************************************************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [3.69.146.43]

TASK [Check that application started with netstat] ********************************************************************************************************************************************************************************************************
changed: [3.69.146.43]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43] => {
    "msg": {
        "changed": true,
        "cmd": [
            "netstat",
            "-plnt"
        ],
        "delta": "0:00:00.017754",
        "end": "2025-06-24 17:12:11.142686",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-24 17:12:11.124932",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "Active Internet connections (only servers)\nProto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    \ntcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2239/sshd: /usr/sbi \ntcp6       0      0 :::8080                 :::*                    LISTEN      33532/java          \ntcp6       0      0 :::22                   :::*                    LISTEN      2239/sshd: /usr/sbi ",
        "stdout_lines": [
            "Active Internet connections (only servers)",
            "Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    ",
            "tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2239/sshd: /usr/sbi ",
            "tcp6       0      0 :::8080                 :::*                    LISTEN      33532/java          ",
            "tcp6       0      0 :::22                   :::*                    LISTEN      2239/sshd: /usr/sbi "
        ]
    }
}

TASK [Print out Jenkins admin password] *******************************************************************************************************************************************************************************************************************
ok: [3.69.146.43]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.69.146.43] => {
    "msg": "MTM4OTIwMWE5ZjgwNDdiNTk5ZTE2YzllYzRiNzkxYzkK"
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
3.69.146.43                : ok=16   changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% 


```
![Bildschirmfoto 2025-06-24 um 18 42 13](https://github.com/user-attachments/assets/21bcfb0b-75e2-42f8-9a24-7269f36ac325)

![Bildschirmfoto 2025-06-24 um 18 43 39](https://github.com/user-attachments/assets/c0bbe6fa-be74-4e6d-bf30-4e9cf41623a8)
![Bildschirmfoto 2025-06-24 um 18 43 49](https://github.com/user-attachments/assets/bc450100-b0b6-44f7-b4e5-e19fa5e887ee)
![Bildschirmfoto 2025-06-24 um 18 48 56](https://github.com/user-attachments/assets/836fc694-6875-4753-bfd4-18e385615602)
![Bildschirmfoto 2025-06-24 um 19 00 36](https://github.com/user-attachments/assets/b38d28d7-9656-4e97-b0a5-14b58e613e25)
![Bildschirmfoto 2025-06-24 um 19 05 39](https://github.com/user-attachments/assets/fcd45380-d57e-4002-b8c8-787bbb53758c)
![Bildschirmfoto 2025-06-24 um 19 13 14](https://github.com/user-attachments/assets/572dbf2e-2f91-4c42-b21d-14da5099bdd6)

</details>





<details>
<summary>Solution 4: Install Jenkins on Ubuntu </summary>
 <br>

> EXERCISE 4: Install Jenkins on Ubuntu

- Your company has infrastructure on multiple platforms. So in addition to creating the Jenkins instance dynamically on an EC2 server, you want to support creating it on an Ubuntu server too. Your task it to re-write your playbook (using include_tasks or conditionals) to support both flavors of the OS.

In the first step i created the Ubuntu EC2 server and executed the following playbook: ansible-playbook 3-provision-jenkins-ec2.yaml
with these extra-vars: "ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-02003f9f0fde924ea ssh_user=ubuntu"

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-02003f9f0fde924ea ssh_user=ubuntu"
[WARNING]: Found both group and host with same name: localhost

PLAY [Provision Jenkins server] ***************************************************************************************************************************************************************************************************************************

TASK [get vpc_information] ********************************************************************************************************************************************************************************************************************************
[WARNING]: packaging.version Python module not installed, unable to check AWS SDK versions
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Get EC2 instances with Name tag 'jenkins-server'] ***************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "instances": [],
        "warnings": [
            "packaging.version Python module not installed, unable to check AWS SDK versions"
        ]
    }
}

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "ansible_facts": {
            "discovered_interpreter_python": "/Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12"
        },
        "changed": false,
        "failed": false,
        "vpcs": [
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "10.0.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-0ba4e5c2ec8dd8e5b",
                        "cidr_block": "10.0.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": false,
                "enable_dns_support": true,
                "id": "vpc-06bb2dee1b5bfd46e",
                "instance_tenancy": "default",
                "is_default": false,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {},
                "vpc_id": "vpc-06bb2dee1b5bfd46e"
            },
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "172.31.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-03d6fba2db0daa6f7",
                        "cidr_block": "172.31.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": true,
                "enable_dns_support": true,
                "id": "vpc-0aef6e3692d08e1df",
                "instance_tenancy": "default",
                "is_default": true,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {},
                "vpc_id": "vpc-0aef6e3692d08e1df"
            },
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "192.168.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-07f0b62edc8d9c48b",
                        "cidr_block": "192.168.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": true,
                "enable_dns_support": true,
                "id": "vpc-0df57cf76051bb1e8",
                "instance_tenancy": "default",
                "is_default": false,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {
                    "Name": "eksctl-my-cluster-cluster/VPC",
                    "alpha.eksctl.io/cluster-name": "my-cluster",
                    "alpha.eksctl.io/cluster-oidc-enabled": "false",
                    "alpha.eksctl.io/eksctl-version": "0.167.0",
                    "aws:cloudformation:logical-id": "VPC",
                    "aws:cloudformation:stack-id": "arn:aws:cloudformation:eu-central-1:524196012679:stack/eksctl-my-cluster-cluster/ed6632d0-2a55-11f0-9ac9-02020b4a74e9",
                    "aws:cloudformation:stack-name": "eksctl-my-cluster-cluster",
                    "eksctl.cluster.k8s.io/v1alpha1/cluster-name": "my-cluster"
                },
                "vpc_id": "vpc-0df57cf76051bb1e8"
            }
        ],
        "warnings": [
            "packaging.version Python module not installed, unable to check AWS SDK versions",
            "Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information."
        ]
    }
}

TASK [Start an instance with a public IP address] *********************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: The network parameter has been deprecated, please use network_interfaces and/or network_interfaces_ids instead. This feature will be removed from amazon.aws in a release after 2026-12-01. Deprecation warnings can be disabled by
 setting deprecation_warnings=False in ansible.cfg.
changed: [localhost]

TASK [pause] **********************************************************************************************************************************************************************************************************************************************
Pausing for 60 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [localhost]

TASK [Ensure hosts file exists] ***************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [update hosts file] **********************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "backup": "",
        "changed": true,
        "diff": [
            {
                "after": "",
                "after_header": "hosts-jenkins-server (content)",
                "before": "",
                "before_header": "hosts-jenkins-server (content)"
            },
            {
                "after_header": "hosts-jenkins-server (file attributes)",
                "before_header": "hosts-jenkins-server (file attributes)"
            }
        ],
        "failed": false,
        "msg": "line added"
    }
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

then i installed Jenkins on the Ubuntu EC2 instance using playbook ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml.

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=ubuntu aws_region=eu-central-1"

PLAY [Get server ip] **************************************************************************************************************************************************************************************************************************************

TASK [Get public_ip address of the ec2 instance] **********************************************************************************************************************************************************************************************************
ok: [localhost]

PLAY [Prepare server for Jenkins] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 35.159.141.25 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [35.159.141.25]

TASK [Include task for amazon-linux server] ***************************************************************************************************************************************************************************************************************
skipping: [35.159.141.25]

TASK [Include task for ubuntu server] *********************************************************************************************************************************************************************************************************************
included: /Users/sgworker/Desktop/ansible_exercises/ansible-exercises/4-host-ubuntu.yaml for 35.159.141.25

TASK [Update apt repo cache] ******************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Add the AdoptOpenJDK APT repository] ****************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Install OpenJDK 17] *********************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Set JAVA_HOME environment variable] *****************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Add GPG keys] ***************************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Install Jenkins from debian package repository] *****************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Update apt repo cache] ******************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Install Jenkins] ************************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Install Docker] *************************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Start Docker] ***************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Check that nvm installed] ***************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Download installer] *********************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [shell] **********************************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [install node] ***************************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25] => {
    "msg": {
        "changed": true,
        "cmd": "source /root/.nvm/nvm.sh && nvm install 8.0.0 && node --version",
        "delta": "0:00:03.015284",
        "end": "2025-06-25 08:35:10.449818",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-25 08:35:07.434534",
        "stderr": "Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...\n#=#=#                                                                          \r\r############################                                              39.0%\r#################################################################         91.0%\r######################################################################## 100.0%\nComputing checksum with sha256sum\nChecksums matched!",
        "stderr_lines": [
            "Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...",
            "#=#=#                                                                          ",
            "",
            "############################                                              39.0%",
            "#################################################################         91.0%",
            "######################################################################## 100.0%",
            "Computing checksum with sha256sum",
            "Checksums matched!"
        ],
        "stdout": "Downloading and installing node v8.0.0...\nNow using node v8.0.0 (npm v5.0.0)\nCreating default alias: \u001b[0;32mdefault\u001b[0m \u001b[0;90m->\u001b[0m \u001b[0;32m8.0.0\u001b[0m (\u001b[0;90m->\u001b[0m \u001b[0;32mv8.0.0\u001b[0m)\nv8.0.0",
        "stdout_lines": [
            "Downloading and installing node v8.0.0...",
            "Now using node v8.0.0 (npm v5.0.0)",
            "Creating default alias: \u001b[0;32mdefault\u001b[0m \u001b[0;90m->\u001b[0m \u001b[0;32m8.0.0\u001b[0m (\u001b[0;90m->\u001b[0m \u001b[0;32mv8.0.0\u001b[0m)",
            "v8.0.0"
        ]
    }
}

PLAY [Start Jenkins] **************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Start Jenkins server] *******************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Wait 10 seconds to check the Jenkins port] **********************************************************************************************************************************************************************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [35.159.141.25]

TASK [Check that application started with netstat] ********************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25] => {
    "msg": {
        "changed": true,
        "cmd": [
            "netstat",
            "-plnt"
        ],
        "delta": "0:00:00.009532",
        "end": "2025-06-25 08:35:24.194307",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-25 08:35:24.184775",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "Active Internet connections (only servers)\nProto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    \ntcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      1194/sshd: ubuntu@p \ntcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      326/systemd-resolve \ntcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      326/systemd-resolve \ntcp        0      0 127.0.0.1:42205         0.0.0.0:*               LISTEN      6426/containerd     \ntcp6       0      0 :::8080                 :::*                    LISTEN      6058/java           \ntcp6       0      0 :::22                   :::*                    LISTEN      1/init              \ntcp6       0      0 ::1:6010                :::*                    LISTEN      1194/sshd: ubuntu@p ",
        "stdout_lines": [
            "Active Internet connections (only servers)",
            "Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    ",
            "tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      1194/sshd: ubuntu@p ",
            "tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      326/systemd-resolve ",
            "tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      326/systemd-resolve ",
            "tcp        0      0 127.0.0.1:42205         0.0.0.0:*               LISTEN      6426/containerd     ",
            "tcp6       0      0 :::8080                 :::*                    LISTEN      6058/java           ",
            "tcp6       0      0 :::22                   :::*                    LISTEN      1/init              ",
            "tcp6       0      0 ::1:6010                :::*                    LISTEN      1194/sshd: ubuntu@p "
        ]
    }
}

TASK [Print out Jenkins admin password] *******************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25] => {
    "msg": "NjNlMzQ4YTQzYzgzNDY4YmE2Y2VmMjhkOWJmMDg4YzQK"
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
35.159.141.25              : ok=24   changed=12   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



```

I then repeated both steps for Amazon Linux with the following extra-vars: ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-092ff8e60e2d51e19 ssh_user=ec2-user; see the screenshots.

```sh

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem aws_region=eu-central-1 key_name=ansible-ssh-key subnet_id=subnet-0e9565afb5dd37baf ami_id=ami-092ff8e60e2d51e19 ssh_user=ec2-user"
[WARNING]: Found both group and host with same name: localhost

PLAY [Provision Jenkins server] ***************************************************************************************************************************************************************************************************************************

TASK [get vpc_information] ********************************************************************************************************************************************************************************************************************************
[WARNING]: packaging.version Python module not installed, unable to check AWS SDK versions
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Get EC2 instances with Name tag 'jenkins-server'] ***************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "instances": [
            {
                "ami_launch_index": 0,
                "architecture": "x86_64",
                "block_device_mappings": [
                    {
                        "device_name": "/dev/sda1",
                        "ebs": {
                            "attach_time": "2025-06-25T08:30:22+00:00",
                            "delete_on_termination": true,
                            "status": "attached",
                            "volume_id": "vol-0e2ba90402461a239"
                        }
                    }
                ],
                "boot_mode": "uefi-preferred",
                "capacity_reservation_specification": {
                    "capacity_reservation_preference": "open"
                },
                "client_token": "8063cc79ae39410197ce442029e8cfda",
                "cpu_options": {
                    "core_count": 2,
                    "threads_per_core": 1
                },
                "current_instance_boot_mode": "legacy-bios",
                "ebs_optimized": false,
                "ena_support": true,
                "enclave_options": {
                    "enabled": false
                },
                "hibernation_options": {
                    "configured": false
                },
                "hypervisor": "xen",
                "image_id": "ami-02003f9f0fde924ea",
                "instance_id": "i-0d0891d9ed855f49c",
                "instance_type": "t2.medium",
                "key_name": "ansible-ssh-key",
                "launch_time": "2025-06-25T08:30:22+00:00",
                "maintenance_options": {
                    "auto_recovery": "default",
                    "reboot_migration": "default"
                },
                "metadata_options": {
                    "http_endpoint": "enabled",
                    "http_protocol_ipv6": "disabled",
                    "http_put_response_hop_limit": 2,
                    "http_tokens": "required",
                    "instance_metadata_tags": "disabled",
                    "state": "applied"
                },
                "monitoring": {
                    "state": "disabled"
                },
                "network_interfaces": [
                    {
                        "association": {
                            "ip_owner_id": "amazon",
                            "public_dns_name": "ec2-35-159-141-25.eu-central-1.compute.amazonaws.com",
                            "public_ip": "35.159.141.25"
                        },
                        "attachment": {
                            "attach_time": "2025-06-25T08:30:22+00:00",
                            "attachment_id": "eni-attach-07e1fa1fd32160cb8",
                            "delete_on_termination": true,
                            "device_index": 0,
                            "network_card_index": 0,
                            "status": "attached"
                        },
                        "description": "",
                        "groups": [
                            {
                                "group_id": "sg-080da33bdf384bb77",
                                "group_name": "default"
                            }
                        ],
                        "interface_type": "interface",
                        "ipv6_addresses": [],
                        "mac_address": "0a:a7:79:3d:63:ff",
                        "network_interface_id": "eni-0314c12d4e7ecd707",
                        "operator": {
                            "managed": false
                        },
                        "owner_id": "524196012679",
                        "private_dns_name": "ip-172-31-3-34.eu-central-1.compute.internal",
                        "private_ip_address": "172.31.3.34",
                        "private_ip_addresses": [
                            {
                                "association": {
                                    "ip_owner_id": "amazon",
                                    "public_dns_name": "ec2-35-159-141-25.eu-central-1.compute.amazonaws.com",
                                    "public_ip": "35.159.141.25"
                                },
                                "primary": true,
                                "private_dns_name": "ip-172-31-3-34.eu-central-1.compute.internal",
                                "private_ip_address": "172.31.3.34"
                            }
                        ],
                        "source_dest_check": true,
                        "status": "in-use",
                        "subnet_id": "subnet-0e9565afb5dd37baf",
                        "vpc_id": "vpc-0aef6e3692d08e1df"
                    }
                ],
                "network_performance_options": {
                    "bandwidth_weighting": "default"
                },
                "operator": {
                    "managed": false
                },
                "placement": {
                    "availability_zone": "eu-central-1c",
                    "group_name": "",
                    "tenancy": "default"
                },
                "platform_details": "Linux/UNIX",
                "private_dns_name": "ip-172-31-3-34.eu-central-1.compute.internal",
                "private_dns_name_options": {
                    "enable_resource_name_dns_a_record": false,
                    "enable_resource_name_dns_aaaa_record": false,
                    "hostname_type": "ip-name"
                },
                "private_ip_address": "172.31.3.34",
                "product_codes": [],
                "public_dns_name": "ec2-35-159-141-25.eu-central-1.compute.amazonaws.com",
                "public_ip_address": "35.159.141.25",
                "root_device_name": "/dev/sda1",
                "root_device_type": "ebs",
                "security_groups": [
                    {
                        "group_id": "sg-080da33bdf384bb77",
                        "group_name": "default"
                    }
                ],
                "source_dest_check": true,
                "state": {
                    "code": 16,
                    "name": "running"
                },
                "state_transition_reason": "",
                "subnet_id": "subnet-0e9565afb5dd37baf",
                "tags": {
                    "Name": "jenkins-server",
                    "server": "Jenkins"
                },
                "usage_operation": "RunInstances",
                "usage_operation_update_time": "2025-06-25T08:30:22+00:00",
                "virtualization_type": "hvm",
                "vpc_id": "vpc-0aef6e3692d08e1df"
            }
        ],
        "warnings": [
            "packaging.version Python module not installed, unable to check AWS SDK versions"
        ]
    }
}

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "ansible_facts": {
            "discovered_interpreter_python": "/Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12"
        },
        "changed": false,
        "failed": false,
        "vpcs": [
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "10.0.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-0ba4e5c2ec8dd8e5b",
                        "cidr_block": "10.0.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": false,
                "enable_dns_support": true,
                "id": "vpc-06bb2dee1b5bfd46e",
                "instance_tenancy": "default",
                "is_default": false,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {},
                "vpc_id": "vpc-06bb2dee1b5bfd46e"
            },
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "172.31.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-03d6fba2db0daa6f7",
                        "cidr_block": "172.31.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": true,
                "enable_dns_support": true,
                "id": "vpc-0aef6e3692d08e1df",
                "instance_tenancy": "default",
                "is_default": true,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {},
                "vpc_id": "vpc-0aef6e3692d08e1df"
            },
            {
                "block_public_access_states": {
                    "internet_gateway_block_mode": "off"
                },
                "cidr_block": "192.168.0.0/16",
                "cidr_block_association_set": [
                    {
                        "association_id": "vpc-cidr-assoc-07f0b62edc8d9c48b",
                        "cidr_block": "192.168.0.0/16",
                        "cidr_block_state": {
                            "state": "associated"
                        }
                    }
                ],
                "dhcp_options_id": "dopt-0b9c2076bc08be01b",
                "enable_dns_hostnames": true,
                "enable_dns_support": true,
                "id": "vpc-0df57cf76051bb1e8",
                "instance_tenancy": "default",
                "is_default": false,
                "owner_id": "524196012679",
                "state": "available",
                "tags": {
                    "Name": "eksctl-my-cluster-cluster/VPC",
                    "alpha.eksctl.io/cluster-name": "my-cluster",
                    "alpha.eksctl.io/cluster-oidc-enabled": "false",
                    "alpha.eksctl.io/eksctl-version": "0.167.0",
                    "aws:cloudformation:logical-id": "VPC",
                    "aws:cloudformation:stack-id": "arn:aws:cloudformation:eu-central-1:524196012679:stack/eksctl-my-cluster-cluster/ed6632d0-2a55-11f0-9ac9-02020b4a74e9",
                    "aws:cloudformation:stack-name": "eksctl-my-cluster-cluster",
                    "eksctl.cluster.k8s.io/v1alpha1/cluster-name": "my-cluster"
                },
                "vpc_id": "vpc-0df57cf76051bb1e8"
            }
        ],
        "warnings": [
            "packaging.version Python module not installed, unable to check AWS SDK versions",
            "Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information."
        ]
    }
}

TASK [Start an instance with a public IP address] *********************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: The network parameter has been deprecated, please use network_interfaces and/or network_interfaces_ids instead. This feature will be removed from amazon.aws in a release after 2026-12-01. Deprecation warnings can be disabled by
 setting deprecation_warnings=False in ansible.cfg.
changed: [localhost]

TASK [pause] **********************************************************************************************************************************************************************************************************************************************
Pausing for 60 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [localhost]

TASK [Ensure hosts file exists] ***************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [update hosts file] **********************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "backup": "",
        "changed": true,
        "diff": [
            {
                "after": "",
                "after_header": "hosts-jenkins-server (content)",
                "before": "",
                "before_header": "hosts-jenkins-server (content)"
            },
            {
                "after_header": "hosts-jenkins-server (file attributes)",
                "before_header": "hosts-jenkins-server (file attributes)"
            }
        ],
        "failed": false,
        "msg": "line added"
    }
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=amazon-linux aws_region=eu-central-1"

PLAY [Get server ip] **************************************************************************************************************************************************************************************************************************************

TASK [Get public_ip address of the ec2 instance] **********************************************************************************************************************************************************************************************************
ok: [localhost]

PLAY [Prepare server for Jenkins] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 3.66.218.117 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [3.66.218.117]

TASK [Include task for amazon-linux server] ***************************************************************************************************************************************************************************************************************
included: /Users/sgworker/Desktop/ansible_exercises/ansible-exercises/4-host-amazon.yaml for 3.66.218.117

TASK [Install Java 17] ************************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [Install Jenkins Repository] *************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [Import RPM key] *************************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [Install /etc/yum.repos.d/jenkins.repo] **************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [Install Docker] *************************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [Include task for ubuntu server] *********************************************************************************************************************************************************************************************************************
skipping: [3.66.218.117]

TASK [Check that nvm installed] ***************************************************************************************************************************************************************************************************************************
ok: [3.66.218.117]

TASK [Download installer] *********************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [shell] **********************************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [install node] ***************************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.66.218.117] => {
    "msg": {
        "changed": true,
        "cmd": "source /root/.nvm/nvm.sh && nvm install 8.0.0 && node --version",
        "delta": "0:00:02.481671",
        "end": "2025-06-25 08:40:22.625659",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-25 08:40:20.143988",
        "stderr": "Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...\n\r###########################################                               60.6%\r######################################################################## 100.0%\nComputing checksum with sha256sum\nChecksums matched!",
        "stderr_lines": [
            "Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...",
            "",
            "###########################################                               60.6%",
            "######################################################################## 100.0%",
            "Computing checksum with sha256sum",
            "Checksums matched!"
        ],
        "stdout": "Downloading and installing node v8.0.0...\nNow using node v8.0.0 (npm v5.0.0)\nCreating default alias: \u001b[0;32mdefault\u001b[0m \u001b[0;90m->\u001b[0m \u001b[0;32m8.0.0\u001b[0m (\u001b[0;90m->\u001b[0m \u001b[0;32mv8.0.0\u001b[0m)\nv8.0.0",
        "stdout_lines": [
            "Downloading and installing node v8.0.0...",
            "Now using node v8.0.0 (npm v5.0.0)",
            "Creating default alias: \u001b[0;32mdefault\u001b[0m \u001b[0;90m->\u001b[0m \u001b[0;32m8.0.0\u001b[0m (\u001b[0;90m->\u001b[0m \u001b[0;32mv8.0.0\u001b[0m)",
            "v8.0.0"
        ]
    }
}

PLAY [Start Jenkins] **************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [3.66.218.117]

TASK [Start Jenkins server] *******************************************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [Wait 10 seconds to check the Jenkins port] **********************************************************************************************************************************************************************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [3.66.218.117]

TASK [Check that application started with netstat] ********************************************************************************************************************************************************************************************************
changed: [3.66.218.117]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.66.218.117] => {
    "msg": {
        "changed": true,
        "cmd": [
            "netstat",
            "-plnt"
        ],
        "delta": "0:00:00.017139",
        "end": "2025-06-25 08:40:47.785759",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-25 08:40:47.768620",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "Active Internet connections (only servers)\nProto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    \ntcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2240/sshd: /usr/sbi \ntcp6       0      0 :::8080                 :::*                    LISTEN      29921/java          \ntcp6       0      0 :::22                   :::*                    LISTEN      2240/sshd: /usr/sbi ",
        "stdout_lines": [
            "Active Internet connections (only servers)",
            "Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    ",
            "tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2240/sshd: /usr/sbi ",
            "tcp6       0      0 :::8080                 :::*                    LISTEN      29921/java          ",
            "tcp6       0      0 :::22                   :::*                    LISTEN      2240/sshd: /usr/sbi "
        ]
    }
}

TASK [Print out Jenkins admin password] *******************************************************************************************************************************************************************************************************************
ok: [3.66.218.117]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [3.66.218.117] => {
    "msg": "NTMzMTk5Y2MyY2M0NDM0ZGIyYjRiZDA1YTAwMDNmNzUK"
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
3.66.218.117               : ok=19   changed=10   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```
![Bildschirmfoto 2025-06-25 um 10 39 16](https://github.com/user-attachments/assets/b93aae1d-ca66-4c93-91a1-0b25e4e1a22f)

![Bildschirmfoto 2025-06-25 um 10 39 34](https://github.com/user-attachments/assets/d7523878-1b7d-4c1d-8c4b-2b636dcd428a)

![Bildschirmfoto 2025-06-25 um 10 39 50](https://github.com/user-attachments/assets/75e06bfd-a0cd-41b2-8490-123c63a12224)


</details>

<details>
<summary>Solution 5: Install Jenkins as a Docker Container </summary>
 <br>

> EXERCISE 5: Install Jenkins as a Docker Container

- In addition to having different OS flavors as an option, your team also wants to be able to run Jenkins as a docker container. So you write another playbook that starts Jenkins as a Docker container with volumes for Jenkins home and Docker itself, because you want to be able to execute Docker commands inside Jenkins.

- Here is a reference of a full docker command for starting Jenkins container, which you should map to Ansible playbook:


```sh
docker run --name jenkins -p 8080:8080 -p 50000:50000 -d \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/bin/docker:/usr/bin/docker \
-v jenkins_home:/var/jenkins_home \
jenkins/jenkins:lts

```

- Your team is happy, because they can now use Ansible to quickly spin up a Jenkins server for different needs.


Step:1 In the first step, I had to remap the port to 9090 for Jenkins because that port was already in use on the EC2 instance. After that, I ran the playbook 5-install-jenkins-docker.yaml.
I carried over the Ubuntu EC2 instance from Task 4 because I had already provisioned it. 

```sh
- name: Start jenkins container
    # docker module used: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#ansible-collections-community-docker-docker-container-module
    community.docker.docker_container:
      name: jenkins
      image: jenkins/jenkins:lts
      volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - "{{ docker_result.stdout }}:/usr/bin/docker"
      - jenkins_home:/var/jenkins_home
      ports:
      - "9090:8080"
      - "50000:50000"
```
```sh


sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook -i hosts-jenkins-server 5-install-jenkins-docker.yaml --extra-vars "aws_region=eu-central-1"

PLAY [Get server ip] **************************************************************************************************************************************************************************************************************************************

TASK [Get public_ip address of the ec2 instance] **********************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Debug EC2 instance info] ****************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "ec2_result": {
        "changed": false,
        "failed": false,
        "instances": [
            {
                "ami_launch_index": 0,
                "architecture": "x86_64",
                "block_device_mappings": [],
                "boot_mode": "uefi-preferred",
                "capacity_reservation_specification": {
                    "capacity_reservation_preference": "open"
                },
                "client_token": "2eda7d5ca36a4d88bc6fec101f16b72b",
                "cpu_options": {
                    "core_count": 2,
                    "threads_per_core": 1
                },
                "current_instance_boot_mode": "legacy-bios",
                "ebs_optimized": false,
                "ena_support": true,
                "enclave_options": {
                    "enabled": false
                },
                "hibernation_options": {
                    "configured": false
                },
                "hypervisor": "xen",
                "image_id": "ami-092ff8e60e2d51e19",
                "instance_id": "i-00af4e029e6884ca2",
                "instance_type": "t2.medium",
                "key_name": "ansible-ssh-key",
                "launch_time": "2025-06-25T08:36:59+00:00",
                "maintenance_options": {
                    "auto_recovery": "default"
                },
                "metadata_options": {
                    "http_endpoint": "enabled",
                    "http_protocol_ipv6": "disabled",
                    "http_put_response_hop_limit": 2,
                    "http_tokens": "required",
                    "instance_metadata_tags": "disabled",
                    "state": "pending"
                },
                "monitoring": {
                    "state": "disabled"
                },
                "network_interfaces": [],
                "network_performance_options": {
                    "bandwidth_weighting": "default"
                },
                "operator": {
                    "managed": false
                },
                "placement": {
                    "availability_zone": "eu-central-1c",
                    "group_name": "",
                    "tenancy": "default"
                },
                "platform_details": "Linux/UNIX",
                "private_dns_name": "",
                "product_codes": [],
                "public_dns_name": "",
                "root_device_name": "/dev/xvda",
                "root_device_type": "ebs",
                "security_groups": [],
                "state": {
                    "code": 48,
                    "name": "terminated"
                },
                "state_reason": {
                    "code": "Client.UserInitiatedShutdown",
                    "message": "Client.UserInitiatedShutdown: User initiated shutdown"
                },
                "state_transition_reason": "User initiated (2025-06-25 09:30:12 GMT)",
                "tags": {
                    "Name": "jenkins-server",
                    "server": "Jenkins"
                },
                "usage_operation": "RunInstances",
                "usage_operation_update_time": "2025-06-25T08:36:59+00:00",
                "virtualization_type": "hvm"
            },
            {
                "ami_launch_index": 0,
                "architecture": "x86_64",
                "block_device_mappings": [
                    {
                        "device_name": "/dev/sda1",
                        "ebs": {
                            "attach_time": "2025-06-25T08:30:22+00:00",
                            "delete_on_termination": true,
                            "status": "attached",
                            "volume_id": "vol-0e2ba90402461a239"
                        }
                    }
                ],
                "boot_mode": "uefi-preferred",
                "capacity_reservation_specification": {
                    "capacity_reservation_preference": "open"
                },
                "client_token": "8063cc79ae39410197ce442029e8cfda",
                "cpu_options": {
                    "core_count": 2,
                    "threads_per_core": 1
                },
                "current_instance_boot_mode": "legacy-bios",
                "ebs_optimized": false,
                "ena_support": true,
                "enclave_options": {
                    "enabled": false
                },
                "hibernation_options": {
                    "configured": false
                },
                "hypervisor": "xen",
                "image_id": "ami-02003f9f0fde924ea",
                "instance_id": "i-0d0891d9ed855f49c",
                "instance_type": "t2.medium",
                "key_name": "ansible-ssh-key",
                "launch_time": "2025-06-25T08:30:22+00:00",
                "maintenance_options": {
                    "auto_recovery": "default"
                },
                "metadata_options": {
                    "http_endpoint": "enabled",
                    "http_protocol_ipv6": "disabled",
                    "http_put_response_hop_limit": 2,
                    "http_tokens": "required",
                    "instance_metadata_tags": "disabled",
                    "state": "applied"
                },
                "monitoring": {
                    "state": "disabled"
                },
                "network_interfaces": [
                    {
                        "association": {
                            "ip_owner_id": "amazon",
                            "public_dns_name": "ec2-35-159-141-25.eu-central-1.compute.amazonaws.com",
                            "public_ip": "35.159.141.25"
                        },
                        "attachment": {
                            "attach_time": "2025-06-25T08:30:22+00:00",
                            "attachment_id": "eni-attach-07e1fa1fd32160cb8",
                            "delete_on_termination": true,
                            "device_index": 0,
                            "network_card_index": 0,
                            "status": "attached"
                        },
                        "description": "",
                        "groups": [
                            {
                                "group_id": "sg-080da33bdf384bb77",
                                "group_name": "default"
                            }
                        ],
                        "interface_type": "interface",
                        "ipv6_addresses": [],
                        "mac_address": "0a:a7:79:3d:63:ff",
                        "network_interface_id": "eni-0314c12d4e7ecd707",
                        "operator": {
                            "managed": false
                        },
                        "owner_id": "524196012679",
                        "private_dns_name": "ip-172-31-3-34.eu-central-1.compute.internal",
                        "private_ip_address": "172.31.3.34",
                        "private_ip_addresses": [
                            {
                                "association": {
                                    "ip_owner_id": "amazon",
                                    "public_dns_name": "ec2-35-159-141-25.eu-central-1.compute.amazonaws.com",
                                    "public_ip": "35.159.141.25"
                                },
                                "primary": true,
                                "private_dns_name": "ip-172-31-3-34.eu-central-1.compute.internal",
                                "private_ip_address": "172.31.3.34"
                            }
                        ],
                        "source_dest_check": true,
                        "status": "in-use",
                        "subnet_id": "subnet-0e9565afb5dd37baf",
                        "vpc_id": "vpc-0aef6e3692d08e1df"
                    }
                ],
                "network_performance_options": {
                    "bandwidth_weighting": "default"
                },
                "operator": {
                    "managed": false
                },
                "placement": {
                    "availability_zone": "eu-central-1c",
                    "group_name": "",
                    "tenancy": "default"
                },
                "platform_details": "Linux/UNIX",
                "private_dns_name": "ip-172-31-3-34.eu-central-1.compute.internal",
                "private_dns_name_options": {
                    "enable_resource_name_dns_a_record": false,
                    "enable_resource_name_dns_aaaa_record": false,
                    "hostname_type": "ip-name"
                },
                "private_ip_address": "172.31.3.34",
                "product_codes": [],
                "public_dns_name": "ec2-35-159-141-25.eu-central-1.compute.amazonaws.com",
                "public_ip_address": "35.159.141.25",
                "root_device_name": "/dev/sda1",
                "root_device_type": "ebs",
                "security_groups": [
                    {
                        "group_id": "sg-080da33bdf384bb77",
                        "group_name": "default"
                    }
                ],
                "source_dest_check": true,
                "state": {
                    "code": 16,
                    "name": "running"
                },
                "state_transition_reason": "",
                "subnet_id": "subnet-0e9565afb5dd37baf",
                "tags": {
                    "Name": "jenkins-server",
                    "server": "Jenkins"
                },
                "usage_operation": "RunInstances",
                "usage_operation_update_time": "2025-06-25T08:30:22+00:00",
                "virtualization_type": "hvm",
                "vpc_id": "vpc-0aef6e3692d08e1df"
            }
        ]
    }
}

PLAY [Prepare server for Jenkins] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host 35.159.141.25 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [35.159.141.25]

TASK [Update apt repo cache] ******************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Install Docker] *************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Install pip3] ***************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Install Docker python module] ***********************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Start Docker Service] *******************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

PLAY [Start Jenkins container on ec2 instance] ************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Get location of docker executable] ******************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

TASK [Start jenkins container] ****************************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

PLAY [Set Docker permission] ******************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [35.159.141.25]

TASK [Set docker permission for Jenkins user] *************************************************************************************************************************************************************************************************************
changed: [35.159.141.25]

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
35.159.141.25              : ok=11   changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ssh -i ansible-ssh-key.pem ubuntu@35.159.141.25
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-1029-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jun 25 12:00:56 UTC 2025

  System load:  0.0               Processes:             126
  Usage of /:   66.1% of 6.71GB   Users logged in:       0
  Memory usage: 34%               IPv4 address for enX0: 172.31.3.34
  Swap usage:   0%

  => There is 1 zombie process.


Expanded Security Maintenance for Applications is not enabled.

26 updates can be applied immediately.
25 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


Last login: Wed Jun 25 09:47:31 2025 from 94.114.29.188
ubuntu@ip-172-31-3-34:~$ sudo docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED       STATUS       PORTS                                              NAMES
bb82b2882855   jenkins/jenkins:lts   "/usr/bin/tini -- /u…"   2 hours ago   Up 2 hours   0.0.0.0:50000->50000/tcp, 0.0.0.0:9090->8080/tcp   jenkins
```
</details>


<details>
<summary>Solution 6: Web server and Database server configuration </summary>
 <br>

> Use repository: https://gitlab.com/devops-bootcamp3/bootcamp-java-mysql

> EXERCISE 6: Web server and Database server configuration
Great, you have helped automate some IT processes in your company. Now another team wants your support as well. They want to automate deploying and configuring web server and database server on AWS. The project is not dockerized and they are using a traditional application setup.

The setup you and the team agreed on is the following: You create a dedicated Ansible server on AWS. In the same VPC as the Ansible server, you create 2 servers, 1 for deploying your Java application and another one for running a MySQL database. Also, the database should not be accessible from outside, only within the VPC, so the DB server shouldn't have a public IP address.

So your task is to:

- Provision and configure dedicated Ansible server

- Write Ansible playbook that provisions a dedicated ansible-control plane server
- Write Ansible playbook that configures the ansible server with all needed tools as well as copies all needed ansible playbooks and configuration for execution there
- Provision and configure databse and web servers

- Write Ansible playbook that provisions database and web servers.
- Write Ansible playbook that installs and starts MySQL server on the EC2 instance without public IP address. And deploys and runs the Java web application on another EC2 instance

> NOTES:

Use an existing mysql role for installing mysql on the database server, instead of writing the whole logic yourself
The last 2 playbooks for provisioning and configuring web and database servers will be executed from Ansible control server, because we can't access the database private IP address from outside VPC
Since the database server will have no public IP address, it will not have a direct internet access. But we will need to download and install some tools, like mysql service itself on the server, and you can do it via NAT gateway. So make sure to create database server in a "private" subnet, with NAT gateway instead of Internet gateway configuration.
Once all the playbooks executed successfully, check that the java application is running and accessible from browser at http://web-server-public-address:8080


Step 1: In the first step, I provisioned the Ansible control-plane server.

```sh
% ansible-playbook 6-provision-ansible-server.yaml                                                             
[WARNING]: Found both group and host with same name: localhost

PLAY [Provision Ansible server] ***************************************************************************************************************************************************************************************************************************

TASK [Start an instance with a public IP address] *********************************************************************************************************************************************************************************************************
[WARNING]: packaging.version Python module not installed, unable to check AWS SDK versions
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
[DEPRECATION WARNING]: The network parameter has been deprecated, please use network_interfaces and/or network_interfaces_ids instead. This feature will be removed from amazon.aws in a release after 2026-12-01. Deprecation warnings can be disabled by
 setting deprecation_warnings=False in ansible.cfg.
changed: [localhost]

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



```
![Bildschirmfoto 2025-06-25 um 14 10 21](https://github.com/user-attachments/assets/c159d9d9-1280-40bd-8be0-fe8fb47d28e8)


I created the aws2_ec2.yaml for my dynamic inventory:

```sh
plugin: amazon.aws.aws_ec2
regions:
  - eu-central-1
filters:
  tag:Name: ansible_server_control_plane
hostnames:
  - public-ip-address
keyed_groups:
  - key: tags.Name
    prefix: tag_Name_
compose:
  ansible_host: public_ip_address
```
I added under the deafult section in ansible.cfg the following properties for the dynamic inventory: 

```sh
# Enable the EC2 inventory plugin (uncomment)
enable_plugins = amazon.aws.aws_ec2

# Specify remote user for SSH (change 'ubuntu' as per your AMI)
remote_user = ubuntu

# Private key to connect to EC2 instances
private_key_file = /Users/sgworker/Desktop/ansible_exercises/ansible-exercises/ansible-ssh-key.pem
```


```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-inventory -i aws_ec2.yaml --list
[WARNING]: Found variable using reserved name: tags
{
    "_meta": {
        "hostvars": {
            "public-ip-address": {
                "ami_launch_index": 0,
                "ansible_host": "3.73.64.55",
                "architecture": "x86_64",
                "block_device_mappings": [
                    {
                        "device_name": "/dev/xvda",
                        "ebs": {
                            "attach_time": "2025-06-25T12:23:26+00:00",
                            "delete_on_termination": true,
                            "status": "attached",
                            "volume_id": "vol-0e862af3a4cbdaa22"
                        }
                    }
                ],
                "boot_mode": "uefi-preferred",
                "capacity_reservation_specification": {
                    "capacity_reservation_preference": "open"
                },
                "client_token": "ef1f535bdc5b49c1adf4942be62f16a3",
                "cpu_options": {
                    "core_count": 1,
                    "threads_per_core": 1
                },
                "current_instance_boot_mode": "legacy-bios",
                "ebs_optimized": false,
                "ena_support": true,
                "enclave_options": {
                    "enabled": false
                },
                "hibernation_options": {
                    "configured": false
                },
                "hypervisor": "xen",
                "image_id": "ami-092ff8e60e2d51e19",
                "instance_id": "i-0e15609111fa61bda",
                "instance_type": "t2.micro",
                "key_name": "ansible-ssh-key",
                "launch_time": "2025-06-25T12:23:25+00:00",
                "maintenance_options": {
                    "auto_recovery": "default"
                },
                "metadata_options": {
                    "http_endpoint": "enabled",
                    "http_protocol_ipv6": "disabled",
                    "http_put_response_hop_limit": 2,
                    "http_tokens": "required",
                    "instance_metadata_tags": "disabled",
                    "state": "applied"
                },
                "monitoring": {
                    "state": "disabled"
                },
                "network_interfaces": [
                    {
                        "association": {
                            "ip_owner_id": "amazon",
                            "public_dns_name": "ec2-3-73-64-55.eu-central-1.compute.amazonaws.com",
                            "public_ip": "3.73.64.55"
                        },
                        "attachment": {
                            "attach_time": "2025-06-25T12:23:26+00:00",
                            "attachment_id": "eni-attach-0fea3a0a931d0fd71",
                            "delete_on_termination": true,
                            "device_index": 0,
                            "network_card_index": 0,
                            "status": "attached"
                        },
                        "description": "",
                        "groups": [
                            {
                                "group_id": "sg-080da33bdf384bb77",
                                "group_name": "default"
                            }
                        ],
                        "interface_type": "interface",
                        "ipv6_addresses": [],
                        "mac_address": "0a:21:e6:16:0d:73",
                        "network_interface_id": "eni-01f1e91ef024e07c3",
                        "operator": {
                            "managed": false
                        },
                        "owner_id": "524196012679",
                        "private_dns_name": "ip-172-31-1-188.eu-central-1.compute.internal",
                        "private_ip_address": "172.31.1.188",
                        "private_ip_addresses": [
                            {
                                "association": {
                                    "ip_owner_id": "amazon",
                                    "public_dns_name": "ec2-3-73-64-55.eu-central-1.compute.amazonaws.com",
                                    "public_ip": "3.73.64.55"
                                },
                                "primary": true,
                                "private_dns_name": "ip-172-31-1-188.eu-central-1.compute.internal",
                                "private_ip_address": "172.31.1.188"
                            }
                        ],
                        "source_dest_check": true,
                        "status": "in-use",
                        "subnet_id": "subnet-0e9565afb5dd37baf",
                        "vpc_id": "vpc-0aef6e3692d08e1df"
                    }
                ],
                "network_performance_options": {
                    "bandwidth_weighting": "default"
                },
                "operator": {
                    "managed": false
                },
                "owner_id": "524196012679",
                "placement": {
                    "availability_zone": "eu-central-1c",
                    "group_name": "",
                    "region": "eu-central-1",
                    "tenancy": "default"
                },
                "platform_details": "Linux/UNIX",
                "private_dns_name": "ip-172-31-1-188.eu-central-1.compute.internal",
                "private_dns_name_options": {
                    "enable_resource_name_dns_a_record": false,
                    "enable_resource_name_dns_aaaa_record": false,
                    "hostname_type": "ip-name"
                },
                "private_ip_address": "172.31.1.188",
                "product_codes": [],
                "public_dns_name": "ec2-3-73-64-55.eu-central-1.compute.amazonaws.com",
                "public_ip_address": "3.73.64.55",
                "requester_id": "",
                "reservation_id": "r-0f21ff600b0846a61",
                "root_device_name": "/dev/xvda",
                "root_device_type": "ebs",
                "security_groups": [
                    {
                        "group_id": "sg-080da33bdf384bb77",
                        "group_name": "default"
                    }
                ],
                "source_dest_check": true,
                "state": {
                    "code": 16,
                    "name": "running"
                },
                "state_transition_reason": "",
                "subnet_id": "subnet-0e9565afb5dd37baf",
                "tags": {
                    "Name": "ansible_server_control_plane"
                },
                "usage_operation": "RunInstances",
                "usage_operation_update_time": "2025-06-25T12:23:25+00:00",
                "virtualization_type": "hvm",
                "vpc_id": "vpc-0aef6e3692d08e1df"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "aws_ec2",
            "tag_Name__ansible_server_control_plane"
        ]
    },
    "aws_ec2": {
        "hosts": [
            "public-ip-address"
        ]
    },
    "tag_Name__ansible_server_control_plane": {
        "hosts": [
            "public-ip-address"
        ]
    }
}

```
I configured the ansible server

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook 6-configure-ansible-server.yaml -i aws_ec2.yaml                                                          

PLAY [Install Ansible] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Found variable using reserved name: tags

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host public-ip-address is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [public-ip-address]

TASK [Update apt repo cache] ******************************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Install ansible and boto3] **************************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

PLAY [Install Ansible] ************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Install ansible role from galaxy] *******************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Install ansible collection for ec2 module] **********************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Ensure .aws dir exists] *****************************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Copy aws credentials] *******************************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Copy private ssh key for the app servers] ***********************************************************************************************************************************************************************************************************
ok: [public-ip-address] => (item=/Users/sgworker/Downloads/ansible-ssh-key.pem)

TASK [Copy ansible playbook and configuration files] ******************************************************************************************************************************************************************************************************
ok: [public-ip-address] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/./6-configure-app-servers.yaml)
ok: [public-ip-address] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/./6-inventory_aws_ec2.yaml)
ok: [public-ip-address] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/./6-vars.yaml)
ok: [public-ip-address] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/./6-provision-app-servers.yaml)
changed: [public-ip-address] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/./6-configure-ansible-server.yaml)
ok: [public-ip-address] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/./6-provision-ansible-server.yaml)

TASK [Copy ansible config file] ***************************************************************************************************************************************************************************************************************************
ok: [public-ip-address]

TASK [Copy java jar file] *********************************************************************************************************************************************************************************************************************************
changed: [public-ip-address]

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
public-ip-address          : ok=12   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

![Bildschirmfoto 2025-06-25 um 15 48 02](https://github.com/user-attachments/assets/a6ab22e9-ff77-4625-9a74-dbc3c0843f2f)

![Bildschirmfoto 2025-06-25 um 15 48 10](https://github.com/user-attachments/assets/09f58615-5dca-485e-865e-8fa171c8e9d0)

I provisioned the database and web server.

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-playbook 6-provision-app-servers.yaml                   
[WARNING]: Found both group and host with same name: localhost

PLAY [Provision Database and Web servers] *****************************************************************************************************************************************************************************************************************

TASK [Start an instance without a public IP address] ******************************************************************************************************************************************************************************************************
[WARNING]: packaging.version Python module not installed, unable to check AWS SDK versions
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
[DEPRECATION WARNING]: The network parameter has been deprecated, please use network_interfaces and/or network_interfaces_ids instead. This feature will be removed from amazon.aws in a release after 2026-12-01. Deprecation warnings can be disabled by
 setting deprecation_warnings=False in ansible.cfg.
changed: [localhost]

TASK [Start an instance with a public IP address] *********************************************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ansible-galaxy install geerlingguy.mysql           
Starting galaxy role install process
- downloading role 'mysql', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-mysql/archive/5.0.2.tar.gz
- extracting geerlingguy.mysql to /Users/sgworker/.ansible/roles/geerlingguy.mysql
- geerlingguy.mysql (5.0.2) was installed successfully
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
```

I logged in via SSH to the Ansible control-plane server and ran the playbook to provision the app and database servers.

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% ssh -i ansible-ssh-key.pem ubuntu@18.157.178.32
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-1029-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jun 25 14:21:25 UTC 2025

  System load:  0.0               Processes:             106
  Usage of /:   38.3% of 6.71GB   Users logged in:       0
  Memory usage: 27%               IPv4 address for enX0: 172.31.10.19
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

32 updates can be applied immediately.
31 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Wed Jun 25 14:17:59 2025 from 94.114.29.188
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ 
ubuntu@ip-172-31-10-19:~$ pwd 
/home/ubuntu
ubuntu@ip-172-31-10-19:~$ ls -al
total 21512
drwxr-x--- 6 ubuntu ubuntu     4096 Jun 25 14:21 .
drwxr-xr-x 3 root   root       4096 Jun 25 13:19 ..
-rw------- 1 ubuntu ubuntu       61 Jun 25 14:21 .Xauthority
drwx------ 6 ubuntu ubuntu     4096 Jun 25 13:21 .ansible
drwxrwxr-x 2 ubuntu ubuntu     4096 Jun 25 13:22 .aws
-rw-r--r-- 1 ubuntu ubuntu      220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu     3771 Mar 31  2024 .bashrc
drwx------ 2 ubuntu ubuntu     4096 Jun 25 13:20 .cache
-rw-r--r-- 1 ubuntu ubuntu      807 Mar 31  2024 .profile
drwx------ 2 ubuntu ubuntu     4096 Jun 25 13:19 .ssh
-rw-r--r-- 1 ubuntu ubuntu        0 Jun 25 13:20 .sudo_as_admin_successful
-rw-rw-r-- 1 ubuntu ubuntu     1787 Jun 25 13:47 6-configure-ansible-server.yaml
-rw-rw-r-- 1 ubuntu ubuntu     1765 Jun 25 14:17 6-configure-app-servers.yaml
-rw-rw-r-- 1 ubuntu ubuntu       80 Jun 25 14:17 6-inventory_aws_ec2.yaml
-rw-rw-r-- 1 ubuntu ubuntu      476 Jun 25 13:22 6-provision-ansible-server.yaml
-rw-rw-r-- 1 ubuntu ubuntu      938 Jun 25 13:22 6-provision-app-servers.yaml
-rw-rw-r-- 1 ubuntu ubuntu      223 Jun 25 13:22 6-vars.yaml
-r-------- 1 ubuntu ubuntu     1678 Jun 25 13:22 ansible-ssh-key.pem
-rw-rw-r-- 1 ubuntu ubuntu      367 Jun 25 13:22 ansible.cfg
-rw-rw-r-- 1 ubuntu ubuntu 21951687 Jun 25 13:47 build-tools-exercises-1.0-SNAPSHOT.jar
ubuntu@ip-172-31-10-19:~$ ansible-playbook -i 6-inventory_aws_ec2.yaml 6-configure-app-servers.yaml

PLAY [Configure Database server] **************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/variables.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Include OS-specific variables.] *************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => (item=/home/ubuntu/.ansible/roles/geerlingguy.mysql/vars/Debian.yml)

TASK [geerlingguy.mysql : Define mysql_packages.] *********************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_daemon.] ***********************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_slow_query_log_file.] **********************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_log_error.] ********************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_syslog_tag.] *******************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_pid_file.] *********************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_config_file.] ******************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_config_include_dir.] ***********************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_socket.] ***********************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Define mysql_supports_innodb_large_prefix.] *************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/setup-Debian.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Check if MySQL is already installed.] *******************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Update apt cache if MySQL is not yet installed.] ********************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Ensure MySQL Python libraries are installed.] ***********************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Ensure MySQL packages are installed.] *******************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Ensure MySQL is stopped after initial install.] *********************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Delete innodb log files created by apt package after initial install.] **********************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => (item=ib_logfile0) 
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => (item=ib_logfile1) 
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Check if MySQL packages were installed.] ****************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/configure.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Run mysql --version] ************************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Extract MySQL version] **********************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Copy my.cnf global MySQL configuration.] ****************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Verify mysql include directory exists.] *****************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Copy my.cnf override files into include directory.] *****************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Create slow query log file (if configured).] ************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Create datadir if it does not exist] ********************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Set ownership on slow query log file (if configured).] **************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Create error log file (if configured).] *****************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Set ownership on error log file (if configured).] *******************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Ensure MySQL is started and enabled on boot.] ***********************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/secure-installation.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Ensure default user is present.] ************************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Copy user-my.cnf file with password credentials.] *******************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Disallow root login remotely] ***************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => (item=DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1'))

TASK [geerlingguy.mysql : Get list of hosts for the root user.] *******************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Update MySQL root authentication via socket for localhost (Linux, MySQL ≥ 8.4)] *************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Update MySQL root password for localhost root account (5.7.x ≤ MySQL < 8.4)] ****************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Update MySQL root password for localhost root account (< 5.7.x).] ***************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Copy .my.cnf file with root password credentials.] ******************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Get list of hosts for the anonymous user.] **************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Remove anonymous MySQL users.] **************************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Remove MySQL test database.] ****************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/databases.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Ensure MySQL databases are present.] ********************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => (item={'name': 'my-app-db', 'encoding': 'latin1', 'collation': 'latin1_general_ci'})

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/users.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Ensure MySQL users are present.] ************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => (item={'name': 'my-user', 'host': '%', 'password': 'my-pass', 'priv': 'my-app-db.*:ALL'})

TASK [geerlingguy.mysql : ansible.builtin.include_tasks] **************************************************************************************************************************************************************************************************
included: /home/ubuntu/.ansible/roles/geerlingguy.mysql/tasks/replication.yml for ec2-18-185-49-114.eu-central-1.compute.amazonaws.com

TASK [geerlingguy.mysql : Ensure replication user exists on master.] **************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Check slave replication status.] ************************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Check master replication status.] ***********************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Configure replication on the slave.] ********************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [geerlingguy.mysql : Start replication.] *************************************************************************************************************************************************************************************************************
skipping: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [validate mysql service started] *********************************************************************************************************************************************************************************************************************
changed: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [ec2-18-185-49-114.eu-central-1.compute.amazonaws.com] => {
    "msg": [
        "mysql       3227  0.4 38.2 1629080 374612 ?      Ssl  14:12   0:02 /usr/sbin/mysqld",
        "root        4701  0.0  0.1   2800  1792 pts/1    S+   14:22   0:00 /bin/sh -c ps aux | grep mysql",
        "root        4703  0.0  0.2   7076  2048 pts/1    S+   14:22   0:00 grep mysql"
    ]
}

PLAY [Install Java] ***************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [Update apt repo cache] ******************************************************************************************************************************************************************************************************************************
ok: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [Add the AdoptOpenJDK APT repository] ****************************************************************************************************************************************************************************************************************
changed: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [Install OpenJDK 17] *********************************************************************************************************************************************************************************************************************************
ok: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [Set JAVA_HOME environment variable] *****************************************************************************************************************************************************************************************************************
ok: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

PLAY [Configure web server] *******************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
ok: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [Copy jar file to server] ****************************************************************************************************************************************************************************************************************************
changed: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [Start java application with needed env vars] ********************************************************************************************************************************************************************************************************
changed: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [validate java app started] **************************************************************************************************************************************************************************************************************************
changed: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com]

TASK [debug] **********************************************************************************************************************************************************************************************************************************************
ok: [ec2-52-59-45-207.eu-central-1.compute.amazonaws.com] => {
    "msg": [
        "ubuntu      7165 51.6  4.1 2320988 40384 ?       Sl   14:22   0:00 java -jar java-app.jar &",
        "ubuntu      7179  0.0  0.1   2800  1792 pts/0    S+   14:22   0:00 /bin/sh -c ps aux | grep java",
        "ubuntu      7181  0.0  0.2   7076  2048 pts/0    S+   14:22   0:00 grep java"
    ]
}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
ec2-18-185-49-114.eu-central-1.compute.amazonaws.com : ok=35   changed=1    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0   
ec2-52-59-45-207.eu-central-1.compute.amazonaws.com : ok=10   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```

fyi: Unfortunately, the image previously available under Team Members is no longer there, so I replaced it in Task 7 and Task 8 with my profile picture :).

![Bildschirmfoto 2025-06-25 um 16 21 46](https://github.com/user-attachments/assets/7a7f5f14-ba0a-4070-9c43-5ba857df24e6)
![Bildschirmfoto 2025-06-25 um 16 23 25](https://github.com/user-attachments/assets/c1896a86-597a-49de-8d2d-1a57492720b4)
![Bildschirmfoto 2025-06-25 um 16 32 11](https://github.com/user-attachments/assets/5db35aa2-7fab-41e6-adc0-365e3d607fca)
![Bildschirmfoto 2025-06-25 um 15 48 10](https://github.com/user-attachments/assets/ba8040ab-6185-4f9d-b77a-761bb8713cd7)
![Bildschirmfoto 2025-06-25 um 15 57 16](https://github.com/user-attachments/assets/5a388211-27a8-493e-bdb1-3f455e3050b9)
![Bildschirmfoto 2025-06-25 um 15 58 57](https://github.com/user-attachments/assets/9dc47060-6771-4bb3-97cb-bb02e9ef2326)
![Bildschirmfoto 2025-06-25 um 16 06 31](https://github.com/user-attachments/assets/17726c0f-7020-4a3b-a128-1d5d9d77bf17)
![Bildschirmfoto 2025-06-25 um 16 21 46](https://github.com/user-attachments/assets/b93a9117-3ac5-4fdc-b5f3-f9947a9e762a)
![Bildschirmfoto 2025-06-25 um 16 23 25](https://github.com/user-attachments/assets/59b2be65-3d11-45fc-b0a6-0499141fd991)
![Bildschirmfoto 2025-06-25 um 16 32 11](https://github.com/user-attachments/assets/a960d6e9-8d89-44a4-bdae-94ee52620877)

</details>


<details>
<summary>Solution 7: Deploy Java MySQL Application in Kubernetes </summary>
 <br>

> EXERCISE 7: Deploy Java MySQL Application in Kubernetes
After some time, the team decides they want to move to a more modern infrastructure setup, so they want to dockerize their application and start deploying to a K8s cluster.

However, K8s is a very new tool for them, and they don't want to learn kubectl and K8s configuration syntax and how all that works, so they want the deployment process to be automated so that it's much easier for them to deploy the application to the cluster without much K8s knowledge.

- So they ask you to help them in the process. You create K8s configuration files for deployments, services for Java and MySQL applications as well as configMap and Secret for the Database connectivity. You also want to access your web application from browser, so you will have to deploy nginx-ingress controller chart and create ingress component for your java app. And you deploy everything in a cluster using an Ansible automated script.

> Note: MySQL application will run as 1 replica and for the Java Application you will need to create and push an image to a Docker repo. You can create the K8s cluster with TF script or any other way you prefer.

!!!!! Notice: In Task 7, the MySQL ports in mysql.yaml were incorrect—they were set to 5432 (the default PostgreSQL port). I corrected them to 3306.

My Docker images are in my repository on Docker Hub: https://hub.docker.com/r/sg1905/demo-app/tags
![Bildschirmfoto 2025-06-26 um 18 48 58](https://github.com/user-attachments/assets/17b704dc-80e9-44a8-9ae1-750c939472b0)

I tested everything using Docker Desktop and ran Kubernetes on my Mac with Docker Desktop. Some adjustments were necessary.

```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   
          minikube         minikube         minikube         default


sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% export KUBECONFIG=~/.kube/config

ansible-playbook 7-deploy-on-k8s.yaml
[WARNING]: Found both group and host with same name: localhost

PLAY [Deploy manifests in k8s cluster] ********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Install docker python module] ***********************************************************************************************************************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: ModuleNotFoundError: No module named 'packaging'
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Failed to import the required Python library (packaging) on MacBook-Pro-3.local's Python /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12. Please read the module documentation and install it in the appropriate location. If the required library is installed, but Ansible is using the wrong Python interpreter, please consult the documentation on ansible_python_interpreter"}

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% pip3 install packaging
Defaulting to user installation because normal site-packages is not writeable
Collecting packaging
  Obtaining dependency information for packaging from https://files.pythonhosted.org/packages/20/12/38679034af332785aac8774540895e234f4d07f7545804097de4b666afd8/packaging-25.0-py3-none-any.whl.metadata
  Downloading packaging-25.0-py3-none-any.whl.metadata (3.3 kB)
Downloading packaging-25.0-py3-none-any.whl (66 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 66.5/66.5 kB 1.1 MB/s eta 0:00:00
Installing collected packages: packaging
Successfully installed packaging-25.0

[notice] A new release of pip is available: 23.2.1 -> 25.1.1
[notice] To update, run: pip3 install --upgrade pip


sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12 -m pip install kubernetes

Defaulting to user installation because normal site-packages is not writeable
Collecting kubernetes
  Obtaining dependency information for kubernetes from https://files.pythonhosted.org/packages/89/43/d9bebfc3db7dea6ec80df5cb2aad8d274dd18ec2edd6c4f21f32c237cbbb/kubernetes-33.1.0-py2.py3-none-any.whl.metadata
  Downloading kubernetes-33.1.0-py2.py3-none-any.whl.metadata (1.7 kB)
Requirement already satisfied: certifi>=14.05.14 in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from kubernetes) (2025.4.26)
Requirement already satisfied: six>=1.9.0 in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from kubernetes) (1.17.0)
Requirement already satisfied: python-dateutil>=2.5.3 in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from kubernetes) (2.9.0.post0)
Collecting pyyaml>=5.4.1 (from kubernetes)
  Obtaining dependency information for pyyaml>=5.4.1 from https://files.pythonhosted.org/packages/86/0c/c581167fc46d6d6d7ddcfb8c843a4de25bdd27e4466938109ca68492292c/PyYAML-6.0.2-cp312-cp312-macosx_10_9_x86_64.whl.metadata
  Downloading PyYAML-6.0.2-cp312-cp312-macosx_10_9_x86_64.whl.metadata (2.1 kB)
Collecting google-auth>=1.0.1 (from kubernetes)
  Obtaining dependency information for google-auth>=1.0.1 from https://files.pythonhosted.org/packages/17/63/b19553b658a1692443c62bd07e5868adaa0ad746a0751ba62c59568cd45b/google_auth-2.40.3-py2.py3-none-any.whl.metadata
  Downloading google_auth-2.40.3-py2.py3-none-any.whl.metadata (6.2 kB)
Collecting websocket-client!=0.40.0,!=0.41.*,!=0.42.*,>=0.32.0 (from kubernetes)
  Obtaining dependency information for websocket-client!=0.40.0,!=0.41.*,!=0.42.*,>=0.32.0 from https://files.pythonhosted.org/packages/5a/84/44687a29792a70e111c5c477230a72c4b957d88d16141199bf9acb7537a3/websocket_client-1.8.0-py3-none-any.whl.metadata
  Downloading websocket_client-1.8.0-py3-none-any.whl.metadata (8.0 kB)
Requirement already satisfied: requests in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from kubernetes) (2.32.3)
Collecting requests-oauthlib (from kubernetes)
  Obtaining dependency information for requests-oauthlib from https://files.pythonhosted.org/packages/3b/5d/63d4ae3b9daea098d5d6f5da83984853c1bbacd5dc826764b249fe119d24/requests_oauthlib-2.0.0-py2.py3-none-any.whl.metadata
  Downloading requests_oauthlib-2.0.0-py2.py3-none-any.whl.metadata (11 kB)
Collecting oauthlib>=3.2.2 (from kubernetes)
  Obtaining dependency information for oauthlib>=3.2.2 from https://files.pythonhosted.org/packages/be/9c/92789c596b8df838baa98fa71844d84283302f7604ed565dafe5a6b5041a/oauthlib-3.3.1-py3-none-any.whl.metadata
  Downloading oauthlib-3.3.1-py3-none-any.whl.metadata (7.9 kB)
Requirement already satisfied: urllib3>=1.24.2 in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from kubernetes) (2.4.0)
Collecting durationpy>=0.7 (from kubernetes)
  Obtaining dependency information for durationpy>=0.7 from https://files.pythonhosted.org/packages/b0/0d/9feae160378a3553fa9a339b0e9c1a048e147a4127210e286ef18b730f03/durationpy-0.10-py3-none-any.whl.metadata
  Downloading durationpy-0.10-py3-none-any.whl.metadata (340 bytes)
Collecting cachetools<6.0,>=2.0.0 (from google-auth>=1.0.1->kubernetes)
  Obtaining dependency information for cachetools<6.0,>=2.0.0 from https://files.pythonhosted.org/packages/72/76/20fa66124dbe6be5cafeb312ece67de6b61dd91a0247d1ea13db4ebb33c2/cachetools-5.5.2-py3-none-any.whl.metadata
  Downloading cachetools-5.5.2-py3-none-any.whl.metadata (5.4 kB)
Collecting pyasn1-modules>=0.2.1 (from google-auth>=1.0.1->kubernetes)
  Obtaining dependency information for pyasn1-modules>=0.2.1 from https://files.pythonhosted.org/packages/47/8d/d529b5d697919ba8c11ad626e835d4039be708a35b0d22de83a269a6682c/pyasn1_modules-0.4.2-py3-none-any.whl.metadata
  Downloading pyasn1_modules-0.4.2-py3-none-any.whl.metadata (3.5 kB)
Collecting rsa<5,>=3.1.4 (from google-auth>=1.0.1->kubernetes)
  Obtaining dependency information for rsa<5,>=3.1.4 from https://files.pythonhosted.org/packages/64/8d/0133e4eb4beed9e425d9a98ed6e081a55d195481b7632472be1af08d2f6b/rsa-4.9.1-py3-none-any.whl.metadata
  Downloading rsa-4.9.1-py3-none-any.whl.metadata (5.6 kB)
Requirement already satisfied: charset-normalizer<4,>=2 in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from requests->kubernetes) (3.4.2)
Requirement already satisfied: idna<4,>=2.5 in /Users/sgworker/Library/Python/3.12/lib/python/site-packages (from requests->kubernetes) (3.10)
Collecting pyasn1<0.7.0,>=0.6.1 (from pyasn1-modules>=0.2.1->google-auth>=1.0.1->kubernetes)
  Obtaining dependency information for pyasn1<0.7.0,>=0.6.1 from https://files.pythonhosted.org/packages/c8/f1/d6a797abb14f6283c0ddff96bbdd46937f64122b8c925cab503dd37f8214/pyasn1-0.6.1-py3-none-any.whl.metadata
  Downloading pyasn1-0.6.1-py3-none-any.whl.metadata (8.4 kB)
Downloading kubernetes-33.1.0-py2.py3-none-any.whl (1.9 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.9/1.9 MB 7.0 MB/s eta 0:00:00
Downloading durationpy-0.10-py3-none-any.whl (3.9 kB)
Downloading google_auth-2.40.3-py2.py3-none-any.whl (216 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 216.1/216.1 kB 4.6 MB/s eta 0:00:00
Downloading oauthlib-3.3.1-py3-none-any.whl (160 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 160.1/160.1 kB 4.7 MB/s eta 0:00:00
Downloading PyYAML-6.0.2-cp312-cp312-macosx_10_9_x86_64.whl (183 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 183.9/183.9 kB 5.2 MB/s eta 0:00:00
Downloading websocket_client-1.8.0-py3-none-any.whl (58 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 58.8/58.8 kB 2.0 MB/s eta 0:00:00
Downloading requests_oauthlib-2.0.0-py2.py3-none-any.whl (24 kB)
Downloading cachetools-5.5.2-py3-none-any.whl (10 kB)
Downloading pyasn1_modules-0.4.2-py3-none-any.whl (181 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 181.3/181.3 kB 5.0 MB/s eta 0:00:00
Downloading rsa-4.9.1-py3-none-any.whl (34 kB)
Downloading pyasn1-0.6.1-py3-none-any.whl (83 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 83.1/83.1 kB 3.0 MB/s eta 0:00:00
Installing collected packages: durationpy, websocket-client, pyyaml, pyasn1, oauthlib, cachetools, rsa, requests-oauthlib, pyasn1-modules, google-auth, kubernetes
  WARNING: The script wsdump is installed in '/Users/sgworker/Library/Python/3.12/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The scripts pyrsa-decrypt, pyrsa-encrypt, pyrsa-keygen, pyrsa-priv2pub, pyrsa-sign and pyrsa-verify are installed in '/Users/sgworker/Library/Python/3.12/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed cachetools-5.5.2 durationpy-0.10 google-auth-2.40.3 kubernetes-33.1.0 oauthlib-3.3.1 pyasn1-0.6.1 pyasn1-modules-0.4.2 pyyaml-6.0.2 requests-oauthlib-2.0.0 rsa-4.9.1 websocket-client-1.8.0

[notice] A new release of pip is available: 23.2.1 -> 25.1.1
[notice] To update, run: pip3 install --upgrade pip
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]

% ansible-playbook 7-deploy-on-k8s.yaml                             
[WARNING]: Found both group and host with same name: localhost

PLAY [Deploy manifests in k8s cluster] ********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************************************************************************
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Install docker python module] ***********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Create docker registry secret] **********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Add ingress-nginx Helm repo] ************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Update Helm repo] ***********************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Deploy nginx ingress controller] ********************************************************************************************************************************************************************************************************************
[WARNING]: The default idempotency check can fail to report changes in certain cases. Install helm diff >= 3.4.1 for better results.
ok: [localhost]

TASK [Deploy all k8s manifests] ***************************************************************************************************************************************************************************************************************************
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-7/db-secret.yaml)
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-7/mysql.yaml)
changed: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-7/java-app-ingress.yaml)
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-7/db-config.yaml)
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-7/java-app.yaml)

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

![Bildschirmfoto 2025-07-08 um 17 50 53](https://github.com/user-attachments/assets/20df6204-8b7a-4f6e-9b19-9f040db3f42d)

![Bildschirmfoto 2025-06-26 um 10 53 25](https://github.com/user-attachments/assets/08d9da39-03cc-4ea2-b12c-9849fdfade0b)
![Bildschirmfoto 2025-06-26 um 10 54 19](https://github.com/user-attachments/assets/a159a634-a01a-47e9-aab6-2785008f2e1e)

![Bildschirmfoto 2025-06-26 um 10 54 37](https://github.com/user-attachments/assets/68625b4a-d26a-4c34-b88b-2b5f444366f2)
![Bildschirmfoto 2025-06-26 um 10 54 56](https://github.com/user-attachments/assets/cf1567ea-d8bc-4c2d-ae15-0c4994215833)
![Bildschirmfoto 2025-06-26 um 10 55 13](https://github.com/user-attachments/assets/fcc5a072-0eb5-48f8-b94e-993c25de2a85)

![Bildschirmfoto 2025-06-26 um 10 57 49](https://github.com/user-attachments/assets/142b6b6d-4092-43e6-98ef-e0c090dc5a56)

</details>



<details>
<summary>Solution 8: Deploy MySQL Chart in Kubernetes </summary>
 <br>

> EXERCISE 8: Deploy MySQL Chart in Kubernetes

Everything works great, but the team worries about the application availability, so wants to run the MySQL DB in multiple replicas. So they ask you to help them solve this problem. Your task is to deploy a MySQL with 3 replicas from a helm chart using Ansible script in place of the currently running single MySQL instance.

- Amazing, now you have automated a lot of things in your project's workflows, which means less manual work and well documented DevOps processes! Really well done!

![Bildschirmfoto 2025-06-26 um 14 31 23](https://github.com/user-attachments/assets/3f6eb0f0-eb09-42d8-98bd-0ac7d2f6f5b3)

I had to add the following pv.yaml PersistentVolume configuration under kubernetes-manifests/exercise-8; without this file, the secondary replicas for MySQL would not start.”


my pv.yaml
```sh
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-secondary-pv-0
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  hostPath:
    path: /Users/sgworker/Desktop/ansible_exercises/ansible-exercises/mysql-secondary-0

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-secondary-pv-1
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  hostPath:
    path: /Users/sgworker/Desktop/ansible_exercises/ansible-exercises/mysql-secondary-1
```

```sh
ansible-playbook 8-deploy-on-k8s.yaml
[WARNING]: Found both group and host with same name: localhost

PLAY [Deploy manifests in k8s cluster] **************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
[WARNING]: Platform darwin on host localhost is using the discovered Python interpreter at /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12, but future installation of another
Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]

TASK [Install docker python module] *****************************************************************************************************************************************************************
ok: [localhost]

TASK [Deploy nginx ingress controller] **************************************************************************************************************************************************************
[WARNING]: The default idempotency check can fail to report changes in certain cases. Install helm diff >= 3.4.1 for better results.
ok: [localhost]

TASK [Deploy Mysql chart with 3 replicas] ***********************************************************************************************************************************************************
changed: [localhost]

TASK [Deploy Java application manifests] ************************************************************************************************************************************************************
changed: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-8/java-db-config.yaml)
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-8/java-app-ingress.yaml)
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-8/java-db-secret.yaml)
ok: [localhost] => (item=/Users/sgworker/Desktop/ansible_exercises/ansible-exercises/kubernetes-manifests/exercise-8/java-app.yaml)

PLAY RECAP ******************************************************************************************************************************************************************************************
localhost                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% kubectl apply -f  kubernetes-manifests/exercise-8/pv.yaml
persistentvolume/mysql-secondary-pv-0 created
persistentvolume/mysql-secondary-pv-1 created
```

```sh
% kubectl apply -f  kubernetes-manifests/exercise-8/pv.yaml
persistentvolume/mysql-secondary-pv-0 created
persistentvolume/mysql-secondary-pv-1 created
```
```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% kubectl get pods                                         
NAME                                   READY   STATUS    RESTARTS         AGE
java-app-deployment-66c7f44df4-vjz4z   1/1     Running   0                47m
mysql-release-primary-0                1/1     Running   0                47m
mysql-release-secondary-0              1/1     Running   0                47m
mysql-release-secondary-1              1/1     Running   0                34m
```
```sh
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]
% kubectl get pvc                                          
NAME                             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-mysql-release-primary-0     Bound    pvc-f925d3f3-9cd1-46ec-a010-64106f2e01be   8Gi        RWO            hostpath       <unset>                 47m
data-mysql-release-secondary-0   Bound    mysql-secondary-pv                         8Gi        RWO            standard       <unset>                 47m
data-mysql-release-secondary-1   Bound    mysql-secondary-pv-0                       8Gi        RWO            standard       <unset>                 34m
sgworker@MacBook-Pro-3.local /Users/sgworker/Desktop/ansible_exercises/ansible-exercises [main]


```
![Bildschirmfoto 2025-06-26 um 14 32 14](https://github.com/user-attachments/assets/21b9d741-6516-45b4-b027-b6d041aec800)
![Bildschirmfoto 2025-06-26 um 14 32 40](https://github.com/user-attachments/assets/e53f0677-256f-41f8-8682-93b5fdb3cee8)
![Bildschirmfoto 2025-06-26 um 14 33 50](https://github.com/user-attachments/assets/4fa69ee5-d630-4152-9d8f-a3b7c7678f0c)

![Bildschirmfoto 2025-06-26 um 15 15 39](https://github.com/user-attachments/assets/2b5d6fc8-749b-44dc-8f65-f9eba3ed577f)
![Bildschirmfoto 2025-06-26 um 15 15 49](https://github.com/user-attachments/assets/96296197-f47a-47f7-ac63-f19c8ad5b0eb)

![Bildschirmfoto 2025-06-26 um 15 16 02](https://github.com/user-attachments/assets/99e21521-0494-4863-b62f-9329eed77161)
![Bildschirmfoto 2025-06-26 um 15 16 22](https://github.com/user-attachments/assets/d7a1c182-a61f-4ac5-9dba-c05210e8d19e)
![Bildschirmfoto 2025-06-26 um 15 18 09](https://github.com/user-attachments/assets/4f6e459a-9530-46a5-a072-8031c7d698bd)
![Bildschirmfoto 2025-07-08 um 17 50 53](https://github.com/user-attachments/assets/b459b58a-5944-42fa-9fde-9de8ce614a94)

</details>


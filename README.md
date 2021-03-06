# GW-CybersecurityBootcamp2021-22
Repository for GW Cybersecurity Project files
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/](Diagrams/Project.drawio.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML files may be used to install only certain pieces of it, such as Filebeat.

  ![DVWA Playbook](https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/Ansible/DVWA/pentest.yml)

```
---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      restart_policy: always
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes
```

![ELK Playbook](https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/Ansible/install-elk.yml)

```
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azureuser
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker pyhon module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044
        #5601 is kibana port
        #9200 is elasticsearch
        #5044 is logstash

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
 ```
 
 ![Filebeat Playbook](https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/Ansible/Filebeat/filebeat-playbook.yml)

```
---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

    # Use command module
  - name: install filebeat .deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

    # Use copy module
  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: enable and configure system module
    command: filebeat modules enable system

    # Use command module
  - name: setup filebeat
    command: filebeat setup

    # Use command module
  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
```

 ![Metricbeat Playbook](https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/Ansible/Metricbeat/metricbeat-playbook.yml)

```
---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.6.1-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting in-bound access to the network.
- A load balancer intelligently distributes traffic from clients across multiple servers without the clients having to understand how many servers are in use or how they are configured. Because the load balancer sits between the clients and the servers it can enhance the user experience by providing additional security (DoS attack counter), performance, and resiliency.

What is the advantage of a jump box?
- A jump box is a secure computer that all admins first connect to before launching any administrative task or use as an origination point to connect to other servers or untrusted environments.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the Jumpbox Provisioner and system network (in this case, the 3 Webservers).
- What does Filebeat watch for? Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- What does Metricbeat record? Metricbeat is installed on target servers to periodically collect metric data from your target servers, this could be operating system metrics such as CPU or memory or data related to services running on the server. It can also be used to monitor other beats and ELK stack itself. Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash. 

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name       | Function  | IP Address | Operating System |
|------------|---------- |------------|------------------|
| Jumpbox    | Gateway   | 10.0.0.4   | Linux            |
| Web-1      | Webserver | 10.0.0.9   | Linux            |
| Web-2      | Webserver | 10.0.0.8   | Linux            |
| Web-3      | Webserver | 10.0.0.7   | Linux            |
| ELK-Server | Monitoring| 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jumpbox Provisioner can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- My personal IP address

Machines within the network can only be accessed by the Docker Container that is running on the Jumpbox Provisioner.
- Only the Jumpbox Provisioner is allowed to access the ELK VM via SSH connection. The IP of the Jumpbox, per the above table is 10.0.0.4. Only my personal IP address can access the ELK server / Kibana page via port 5601 (Kibana's port).

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jumpbox  | Yes                 | My Personal IP       |
| Web-1    | No                  | 10.0.0.4             |
| Web-2    | No                  | 10.0.0.4             |
| Web-3    | No                  | 10.0.0.4             |
| ELK      | Yes/No              | My IP:5601/10.0.0.4  |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. Ansible automates the process of deploying configurations on the ELK server, as well as multiple web-servers, and thus ensures any configuration script is handled identically on all servers within the ELK / Webserver groups within the Ansible host file. Automating server provisioning can eliminate mistakes, and it also allows automation and sequential execution of the independent steps; that said if a mistake is created within the Ansible playbook, this mistake could be carried across all servers configured via the script. The Ansible automation is capable of provisioning any number of hosts (within the limits of network IP space).

The playbook implements the following tasks:
- Installs docker.io to the ELK machine (VM)
- Installs pip3 (python3) to the ELK machine
- Installs the Docker python module
- Increase the virtual memory of the ELK machine
- Download and launch the Docker ELK container, configuring/publishing the ports for elasticsearch, logstash, and kibana

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/](Images/docker_ps.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
| Name     | IP Addresses |
|----------|--------------|
| Web-1    | 10.0.0.9     |
| Web-2    | 10.0.0.8     |
| Web-3    | 10.0.0.7     |
| ELK      | 10.1.0.4     | 
- Filebeat only installed on ELK Machine to monitor it as well


We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- As mentioned above, Filebeat monitors the logs or locations specified, collects that data, and forwards it to the ELK server

![https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/](Images/Filebeat.png)
- As mentioned above, Metricbeat periodically collects metric data from your target servers, this could be operating system metrics such as CPU or memory or data related to services running on the server. It can also be used to monitor other beats and ELK stack itself. Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash.

![https://github.com/Eric-Spencer/GW-CybersecurityBootcamp2021-22/blob/main/](Images/Metricbeat.png)

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook file (ex: filebeat-playbook.yml or install-elk.yml) file to the /etc/ansible or the /etc/ansible/roles folder, as appropriate.
- Update the configuration file to include the IP address of the ELK machine, as well as update the hosts file to contain the IPs of the Webservers or ELK server, respectively.
- Run the playbook, and navigate to the ELK server's public IP -- http://[your.VM.IP]:5601/app/kibana to check that the installation worked as expected.


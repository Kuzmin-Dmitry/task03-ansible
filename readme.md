Introduction
  The project consists of three playbooks which install ELK stack and Filebeat, manage the services and manage the indexes. For visibility the tasks for preparing and installing ELK + Filebeat are divided in separate files. For compatibility and independence from an Internet connection, the installation of packages performes from the local repository. If you have -rw- to ~/.ssh/ and /etc/ansible/ then playlists can be executed without elevating access rights but by default you need use sudo to execute it. To run scripts on remote computers and manage services you need elevation of rights. By default ELK and Filebeat use the root user (the root password must be the same).
  deploy.yml - install and configure ELK + Nginx + Java and Filebeat;
  check_ctatus.yml - start or restart ELK services or show the message if there are no such services;
  manage_indices.yml - manage the indexes of Logstash.

Preparation
  Playbooks were tested on Centos 7 systems with minimal configuration. Before running the playbooks you must install and start SSH client and server and create rules on the firewall to SSH and ELK (or disable it) on the target machines. You also need to configure the authorization via SSH between the Ansible server and hosts. You must specify the ip-addresses of the target machines, the path to the local repository, and the names of the corresponding packages in the variable file. You must add a group of target computers to the /etc/ansible/ hosts file. For example:
  [task-nodes]
  elk-node ansible_ssh_host = 192.168.1.2 ansible_connection = ssh
  filebeat-node ansible_ssh_host = 192.168.1.3 ansible_connection = ssh
  Caution: On Centos 7 machines when running the playbook with the --ask-pass option the error appears: "Using a SSH password is not possible because Host key checking is enabled and sshpass does not support this. To your known_hosts file to manage this host. ". The error is solved by adding the string host_key_checking = False to /etc/ansible/ansible.cfg.

Installing ELK + Nginx + Java on the server and Filebeat on the client
  deploy.yml - installs the ELK stack and dependencies on the server and then installs Filebeat on the client
  Running by default: sudo ansible-playbook deploy.yml --ask-pass

Check services
  check_status.yml - the playbook checks and returns the current status of the services from systemctl and tries to start them if the services are not running.
  Running by default: sudo ansible-playbook check_status.yml --ask-pass

Manage indexes
  manage_indices.yml - used with the index_action parameter with the following values:
    get-all or without parameters - returns a list of all indexes;
    delete-old - delete all indexes;
    delete-by-pattern - deletion of indexes by a template of the form "<index-name>*";
    purge-all - cleaning the contents of the indexes.
    Running by default: sudo ansible-playbook manage_indices.yml --extra-vars "-action <action name> [-pattern <pattern view>]"

Structure

ansible
├── check_status.yml
├── deploying.yml
├── manage_indices.yml
├── files
│   ├── check.sh.j2
│   └── index_management.sh.j2
├── group_vars
│   ├── common.yml
│   ├── deploy.yml
│   ├── manage.yml
│   └── vars.yml
├── roles
│   ├── elasticsearch
│   │   └── tasks
│   │       └── main.yml
│   ├── filebeat
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   ├── java
│   │   └── tasks
│   │       └── main.yml
│   ├── kibana
│   │   └── tasks
│   │       └── main.yml
│   ├── logstash
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── beats-input.conf.j2
│   │       ├── elasticsearch-output.conf.j2
│   │       └── syslog-filter.conf.j2
│   ├── nginx
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── kibanaAdmin.j2
│   │       └── kibana.conf.j2
│   └── preparation
│       └── tasks
│           └── main.yml
└── vars
    └── default-vars.yml

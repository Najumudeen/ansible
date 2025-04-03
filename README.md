# ansible

ANSIBLE


YAML
INVENTROY
Playbooks
Variables
Modules
Loop




Start with DRY run option

ansible-playbook playbook.yml —check


Ansible-playbook playbook.yml —start-at-task “start httpd service”

Tag

Tasks:
- Yum:
-   Name: httpd
-   State: installed
- Tags: install

ansible-playbook playbook.yml —tags “install”

ansible-playbook playbook.yml —skip—tags “install”


---
- name: Create a myfile.txt
  hosts: web1
  gather_facts: no
  tasks:
    - name: Create myfile.txt
      file:
        path: /root/myfile.txt
        state: touch

User Creation using Ansible

---
- hosts: web1
  tasks:
    - name: Create user angel
      user:
        name: angel

Ansible Facts

     Collect the information
     Collect the information about hostname, disk, volumes and space in the disk
     Using Setup Module Anisble collect the facts from target hosts
—-
- Name: print hello message
- Hosts: all
- Tasks:
- - debug:
-       Msg: hello from Ansible

All facts collected by Ansible stored in ansible_facts variables

Disable gather facts



- Name: print hello message
- Hosts: all
- gather_facts: no
- Tasks:
- - debug:
-       Msg: hello from Ansible




/etc/ansible/ansible.cfg

Gathering  = implicit # gather by facts

/etc/ansible/hosts is not mentioned not able to collect the facts

Getthering Information

ansible -m setup localhost | grep distribution_version

cd playbooks/ && ansible -i inventory -m setup web1 | grep distribution

---
- hosts: db1
  gather_facts: True
  tasks:
    - name: Get server architecture
      debug:
        var: ansible_facts.architecture


Ansible Configuration Files

/etc/ansible/ansible.cfg
[defaults]



# SSH timeout
Timeout        = 10
Forks            = 5

[inventory]

[privilege_escalation]

[paramiko_connection]

[ssh_connection]

[persistent_connection]

[colors]



/etc/ansible/ansible.cfg


/opt/ansible-web.cfg
     /opt/web-playbooks
    /opt/web-playbooks/ansible.cfg

/opt/db-playbooks
   /opt/db-playbooks/ansible.cfg

/opt/network-playbooks
  /opt/network-playbooks/ansible.cfg


$ANSIBLE_CONFIG=/opt/ansible-web.cfg ansible-playbook playbook.yml

Using ENVIRONMENT variable to change that 

/etc/ansible/ansible.cfg
Gathering                      = implicit                    ANSIBLE_GATHERING=explicit

You can use export to set the environment variables

Ansible Configuration Variables

Applicable  for only single playbook execution

Export ANSIBLE_GATHERING=explicit
Ansible-playbook playbook.yml



View Configuration

Ansible-config list            # lists all configuration

Ansible-config view.  # show the current config file

Ansible-config dump.   # show the current setting

Export ANSIBLE_GATHERING=explicit
Ansible-config dump | grep GATHERING
DEFAULT_GATHERING(env: ANSIBLE_GATHERING)= explicit


vi /home/thor/playbooks/ansible.cfg
remote_port = 2222

Control Node

You can install ansible on windows machines.

Ansible can manage windows node but you can not install ansible on windows.

Sudo pip install ansible

Default inventory file /etc/ansible/hosts

Default ansible config file /etc/ansbile/ansible.cfg

Update /home/thor/playbooks/ansible.cfg configuration file with new log path as given below:

cat <<EOF  > /home/thor/playbooks/ansible.cfg
log_path = /var/log/ansible/ansible.log
EOF

Static Inventory hosts files

Creating and distributing ssh Keys

Not using right way

/etc/ansible/hosts

Web1 ansible_host=172.20.1.100 ansible_ssh_pass=Passw0rd
Web2 ansible_host=172.20.1.101 ansible_ssh_pass=Passw0rd

Ssh-keygen
id_rsa id_rsa.pub

Cat ~/.ssh/authorized_keys

Key is associated with particular user.

Password based authentication

Create ssh key in the controller vm and copy to target vas

Ssh-copy-id -I id_rsa user1@server1

Web1 ansible_host=172.20.1.100
Web2 ansible_host=172.20.1.101 

If user .ssh key is not in default location then do the below 

Web1 ansible_host=172.20.1.100 ansible_user=user1 ansible_ssh_private_key_file=/some-path/privatekey
Web2 ansible_host=172.20.1.101 ansible_user=user1 ansible_ssh_private_key_file=/some-path/privatekey


ssh-keygen -t rsa -f ~/.ssh/ansible

We would like to establish password-less secure authentication between Ansible controller and web1 node. Use the keys generated in the previous step and do the needful.

Username is ansible and generate public name also ansible

ssh-copy-id -i /home/thor/.ssh/ansible ansible@web1


Use the ping module to test connectivity through Ansible - ansible -m ping -i inventory web1

ADHOC command

Ansible -m ping all

Ansible -a ‘cat /etc/hosts’ all

Run the ansible command to gather facts of the localhost and save the output in /tmp/ansible_facts.txt file.


Run an adhoc command to perform a ping connectivity to all hosts in the /home/thor/playbooks/inventory file and save the output in /tmp/ansible_all.txt file.

ANSIBLE_HOST_KEY_CHECKING=False ansible -m ping -i /home/thor/playbooks/inventory all > /tmp/ansible_all.txt

Use the command module and argument date

ansible -m command -a date -i inventory web1 > /tmp/ansible_date.txt

ansible all -a "hostname" -i inventory
ansible node00 -m copy -a "src=/etc/resolv.conf dest=/tmp/resolv.conf" -i inventory



---
- hosts: all
  tasks:
    - shell: cat /etc/redhat-release

export ANSIBLE_GATHERING=explicit
ansible all -m shell -a uptime -i inventory
ansible-playbook -i inventory playbook.yml -vv

Privilege Escalation

Users

Root => super user

Admin => 

Developer

Admin ssh -I id_ras admin@server1

Sudo yum install nginx 

Become Super user (sudo)

Become Method - sudo (pfexec, does, ksu, runas)

Becoming another user => su nginx

Inventory

Lamp-dev1 ansible_host=172.20.1.100 ansible_user=admin

Become:yes
become_user: nginx


FAQ

—-  it indicate the beginning go the yaml file

Msg: “{{ required }}” // use double curly bases , variable define inside
Var : dns_server_ip    // not use double curly bases with var used
When: ansible_host != ‘web’        //  Not use curly braces when used 
with_items: {{ required  }}        // Use curly braces with_items used


ansible_ssh_pass and ansible_password used both



Ansible Modules

Packages

   1. Yum

   2. Apt

   3. package:
      Name: httpd
      State: installed

  4. Service
        Name: httpd

Firewall Rules

  5. Firewalld
  
 Permanent: yes
 Immediate: yes

6. Storage

     1. lvg 
     2. lvol


6. Filesystem

  1. fstype: ext4
    dev: /dev/vg1/lvol1
Opts: -cc

  2. Mount

  7. File

8. Archive

      Format: zip| tar|bz2| xz | gz
 
9. Unarchive

        remote_src: yes

  10. Corn

11. Users and groups


---
- hosts: all
  gather_facts: no
  tasks:
    - name: Make changes in Apache config
      replace:
        path: /etc/httpd/conf/httpd.conf
        regexp: "^Listen 80"
        replace: "Listen 443"

    - name: Restart Apache
      service:
        name: httpd
        state: restarted


Update find.yml playbook as per below given code

We have a playbook ~/playbooks/find.yml that recursively finds files in /opt/data directory older than 2 minutes and equal or greater than 1 megabyte in size. It also copies those files under /opt directory. However it has some missing parameters so its not working as expected, take a look into it and make appropriate changes.

---
- hosts: web1
  tasks:
    - name: Find files
      find:
        paths: /opt/data
        age: 2m
        size: 1m
        recurse: yes
      register: file

    - name: Copy files
      command: "cp {{ item.path }} /opt"
      with_items: "{{ file.files }}"


Consider using insertbefore parameter to add line in the beginning of the file.


---
- name: Add block to index.html
  hosts: web1
  tasks:
   - blockinfile:
      owner: apache
      group: apache
      insertbefore: BOF
      path: /var/www/html/index.html
      block: |
       Welcome to KodeKloud!
       This is Ansible Lab.


Create httpd.yml playbook and add below given code

---
- name: replace port 80 to 8080
  hosts: web1
  tasks:
  - replace:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^(Listen)\s+80\s*$'
      replace: 'Listen 8080'
  - service: name=httpd state=restarted


Use format zip

---
- name: Zip archive opt.zip
  hosts: web1
  tasks:
   - archive:
       path: /opt
       dest: /root/opt.zip
       format: zip

Use src: local.zip

Use unarchive and file modules to complete the task.

On web1 node we have an archive data.tar.gz under /root directory, extract it under /srv directory by developing a playbook ~/playbooks/data.yml and make sure data.tar.gz archive is removed after that.

---
- name: Download and extract from URL
  hosts: web1
  tasks:
  -   unarchive:
       src: https://github.com/kodekloudhub/Hello-World/archive/master.zip
       dest: /root
       remote_src: yes

Create files.yml playbook and add below given code

- name: Compress multiple files
  hosts: web1
  tasks:
  - archive:
     path:
      - /root/file1.txt
      - /usr/local/share/file2.txt
      - /var/log/lastlog
     dest: /root/files.tar.bz2
     format: bz2

Some of the useful modules are yum, service, unarchive and replace.

- name: Install and configure nginx on web1
  hosts: web1
  tasks:
  - name: Install nginx
    yum: name=nginx state=installed
  - name: Start nginx
    service: name=nginx state=started enabled=yes

  - name: Extract nginx.zip
    unarchive: src=/root/nginx.zip dest=/usr/share/nginx/html remote_src=yes

  - name: Replace line in index.html
    replace:
     path: /usr/share/nginx/html/index.html
     regexp: This is sample html code
     replace: This is KodeKloud Ansible lab


Create a playbook ~/playbooks/lastlog.yml to add a cron job Clear Lastlog on node00 to empty the /var/log/lastlog logs file. The job must run at 12am everyday.

You can use the command echo “” > /var/log/lastlog to empty the lastlog file and schedule should be 0 0 * * *.


---
- name: Create a cron job to clear last log
  hosts: node00
  tasks:
   - name: Create cron job
     cron:
       name: "Clear Lastlog"
       minute: "0"
       hour: "0"
       job: echo "" > /var/log/lastlog


---
- name: Create a cron job to run free.sh script
  hosts: node00
  tasks:
   - name: Create cron job
     cron:
       name: "Free Memory Check"
       minute: "0"
       hour: "*/2"
       job: "sh /root/free.sh"


Take care about exact name of the cron Check Memory.


---
- name: remove cron job from node00
  hosts: node00
  tasks:
  - name: Remove cron job
    cron:
      name: "Check Memory"
      state: absent


Due to some disk space limitations, we want to cleanup the /tmp location on node00 host after every reboot. Create a playbook ~/playbooks/reboot.yml to add a cron named cleanup on node00 that will execute after every reboot and will clean /tmp location.

The command should be rm -rf /tmp/*.

Create reboot.yml playbook and it should look like as below.

---
- name: Cleanup /tmp after every reboot
  hosts: node00
  tasks:
   - cron:
      name: cleanup
      job: rm -rf /tmp/*
      special_time: reboot


Generated cron expression should be 5 8 * * 0


On node00 we want to keep the installed packages up to date, so we would like to run yum updates regularly. Create a playbook ~/playbooks/yum_update.yml and create a cron job as described below:
a. Do not add cron directly using crontab instead create a cron file /etc/cron.d/ansible_yum.
b. The cron must run on every Sunday at 8:05 am.
c. The name of the cron must be yum update.
d. Cron should be added for user root

Use command yum -y update

Create user and admin

---

- hosts: all
  gather_facts: no
  tasks:
    - group:
        name: 'admin'
        state: present
    - user:
        name: 'admin'
        uid: 2048
        group: 'admin'

Create add_user.yml playbook and add below given code


---
- hosts: all
  gather_facts: no
  tasks:
    - user:
        name: 'neymarsabin'
        expires: 1704067199

Create remove_user.yml playbook and add below given code


---
- hosts: all
  gather_facts: no
  tasks:
    - user:
        name: 'admin'
        state: absent
    - group:
        name: 'admin'
        state: absent



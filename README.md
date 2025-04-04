## ANSIBLE INTRODUCTION

1. `YAML`
2. `INVENTROY`
3. `Playbooks`
4. `Variables`
5. `Modules`
6. `Loop`

### How ansible playbook run with `DRY` option

```
ansible-playbook playbook.yml —check
```

```
ansible-playbook playbook.yml —start-at-task “start httpd service”
```

### Tag

### Tasks

```
yum:
  name: httpd
  state: installed
  tags: install
```

```
ansible-playbook playbook.yml —tags install
```

```
ansible-playbook playbook.yml —skip—tags install
```

```
---
- name: Create a myfile.txt
  hosts: web1
  gather_facts: no
  tasks:
    - name: Create myfile.txt
      file:
        path: /root/myfile.txt
        state: touch
```

### User Creation using Ansible

```
---
- hosts: web1
  tasks:
    - name: Create user angel
      user:
        name: angel
```

### Ansible Facts

- Collect the information
- Collect the information about hostname, disk, volumes and space in the disk
- Using Setup Module Anisble collect the facts from target hosts

```
name: print hello message
  hosts: all
  tasks:
  - debug:
      msg: hello from Ansible
```
> [!TIP]
> All facts collected by Ansible stored in ansible_facts variables

### Disable gather facts

```
name: print hello message
  hosts: all
  gather_facts: no
  tasks:
  - debug:
      msg: hello from Ansible
```

### config info

/etc/ansible/ansible.cfg

Gathering  = implicit # gather by facts

/etc/ansible/hosts is not mentioned not able to collect the facts

### Getthering Information

```
ansible -m setup localhost | grep distribution_version
cd playbooks/ && ansible -i inventory -m setup web1 | grep distribution
```

```
---
- hosts: db1
  gather_facts: True
  tasks:
    - name: Get server architecture
      debug:
        var: ansible_facts.architecture
```

### Ansible Configuration Files

```
/etc/ansible/ansible.cfg

[defaults]

## SSH timeout
Timeout          = 10
Forks            = 5

[inventory]

[privilege_escalation]

[paramiko_connection]

[ssh_connection]

[persistent_connection]

[colors]
```

### ansible cfg file location
```
/etc/ansible/ansible.cfg
/opt/ansible-web.cfg
/opt/web-playbooks
/opt/web-playbooks/ansible.cfg
/opt/db-playbooks
/opt/db-playbooks/ansible.cfg
/opt/network-playbooks
/opt/network-playbooks/ansible.cfg
```

$ANSIBLE_CONFIG=/opt/ansible-web.cfg ansible-playbook playbook.yml

### Using ENVIRONMENT variable to change that 

/etc/ansible/ansible.cfg

Gathering       = implicit      ## ANSIBLE_GATHERING=explicit

You can use export to set the environment variables

### Ansible Configuration Variables

Applicable  for only single playbook execution

```
export ANSIBLE_GATHERING=explicit
ansible-playbook playbook.yml
```
### View Configuration

```
ansible-config list     # lists all configuration

ansible-config view.    # show the current config file

ansible-config dump.    # show the current setting

export ANSIBLE_GATHERING=explicit

ansible-config dump | grep GATHERING

DEFAULT_GATHERING(env: ANSIBLE_GATHERING)= explicit
```


vi /home/thor/playbooks/ansible.cfg

remote_port = 2222

### Control Node

> [!NOTE]
> You can install ansible on windows machines.

ansible can manage windows node but you can not install ansible on windows.

```
sudo pip install ansible
```

### Default inventory file 
```
/etc/ansible/hosts
```

### Default ansible config file 
```
/etc/ansbile/ansible.cfg
```

### Update /home/thor/playbooks/ansible.cfg configuration file with new log path as given below:
```
cat <<EOF  > /home/thor/playbooks/ansible.cfg
log_path = /var/log/ansible/ansible.log
EOF
```

### Static Inventory hosts files

Creating and distributing ssh Keys

Not using right way

/etc/ansible/hosts

Web1 ansible_host=172.20.1.100 ansible_ssh_pass=Passw0rd
Web2 ansible_host=172.20.1.101 ansible_ssh_pass=Passw0rd

```
$ ssh-keygen
  id_rsa id_rsa.pub

$ cat ~/.ssh/authorized_keys
```
key is associated with particular user.

### Password based authentication

Create ssh key in the controller vm and copy to target vas

```
ssh-copy-id -I id_rsa user1@server1
```

web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101 

If user .ssh key is not in default location then do the below 

web1 ansible_host=172.20.1.100 ansible_user=user1 ansible_ssh_private_key_file=/some-path/privatekey
web2 ansible_host=172.20.1.101 ansible_user=user1 ansible_ssh_private_key_file=/some-path/privatekey

```
ssh-keygen -t rsa -f ~/.ssh/ansible
```

We would like to establish password-less secure authentication between ansible controller and web1 node. Use the keys generated in the previous step and do the needful.

Username is ansible and generate public name also ansible

```
ssh-copy-id -i /home/thor/.ssh/ansible ansible@web1
```

Use the ping module to test connectivity through Ansible - `ansible -m ping -i inventory web1`

### ADHOC command

```
ansible -m ping all
ansible -a ‘cat /etc/hosts’ all
```

Run the ansible command to gather facts of the localhost and save the output in /tmp/ansible_facts.txt file.

Run an adhoc command to perform a ping connectivity to all hosts in the /home/thor/playbooks/inventory file and save the output in /tmp/ansible_all.txt file.

`ANSIBLE_HOST_KEY_CHECKING=False ansible -m ping -i /home/thor/playbooks/inventory all > /tmp/ansible_all.txt`

### Use the command module and argument date

```
ansible -m command -a date -i inventory web1 > /tmp/ansible_date.txt

ansible all -a "hostname" -i inventory
ansible node00 -m copy -a "src=/etc/resolv.conf dest=/tmp/resolv.conf" -i inventory
```
```
---
- hosts: all
  tasks:
    - shell: cat /etc/redhat-release

export ANSIBLE_GATHERING=explicit

ansible all -m shell -a uptime -i inventory

ansible-playbook -i inventory playbook.yml -vv
```

### Privilege Escalation
```
Users
Root => super user
Admin
Developer
```
```
admin ssh -I id_ras admin@server1
sudo yum install nginx 
```

Become Super user (sudo)
Become Method - sudo (pfexec, does, ksu, runas)
Becoming another user => su nginx

##### Inventory file add the following line

```
lamp-dev1 ansible_host=172.20.1.100 ansible_user=admin
```

```
Become:yes
become_user: nginx
```

##### FAQ

> [!TIP]
> `---` 3 dashes indicate the beginning go the yaml file

```
msg: “{{ required }}”              // use double curly bases , variable define inside
var : dns_server_ip                // not use double curly bases with var used
when: ansible_host != ‘web’        //  Not use curly braces when used 
with_items: {{ required  }}        // Use curly braces with_items used
```

ansible_ssh_pass and ansible_password used both

### Ansible Modules

##### Packages

1. `yum`
2. `apt`
3. `package:
      name: httpd
      state: installed`
4. `service
     name: httpd`

##### Firewall Rules

5. Firewalld
     Permanent: yes
     Immediate: yes

6. Storage  
    lvg 
    lvol
  
7. Filesystem
   
  1. fstype: ext4

  dev: /dev/vg1/lvol1
  Opts: -cc

  2. Mount

8. `File`

9. `Archive

Format: zip| tar|bz2| xz | gz`
 
10. `Unarchive

remote_src: yes`

11. `Corn`

12. `Users and groups`

```
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
```

Update `find.yml` playbook as per below given code

We have a playbook ~/playbooks/find.yml that recursively finds files in /opt/data directory older than 2 minutes and equal or greater than 1 megabyte in size.<br/>
It also copies those files under /opt directory. However it has some missing parameters so its not working as expected, take a look into it and make appropriate changes.<br/>

```
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
```

Consider using insert before parameter to add line in the beginning of the file.

```
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
```

##### Create httpd.yml playbook and add below given code

```
- name: replace port 80 to 8080
  hosts: web1
  tasks:
  - replace:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^(Listen)\s+80\s*$'
      replace: 'Listen 8080'
  - service: name=httpd state=restarted
```

##### Use format zip

```
- name: Zip archive opt.zip
  hosts: web1
  tasks:
   - archive:
       path: /opt
       dest: /root/opt.zip
       format: zip
```

Use src: local.zip

Use unarchive and file modules to complete the task.

On web1 node we have an archive data.tar.gz under /root directory, extract it under /srv directory by developing a playbook ~/playbooks/data.yml and make sure data.tar.gz archive is removed after that.

```
- name: Download and extract from URL
  hosts: web1
  tasks:
  -   unarchive:
       src: https://github.com/kodekloudhub/Hello-World/archive/master.zip
       dest: /root
       remote_src: yes
```

##### Create files.yml playbook and add below given code

```
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
```
##### Some of the useful modules are yum, service, unarchive and replace.

```
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
```

##### Create a playbook ~/playbooks/lastlog.yml to add a cron job Clear Lastlog on node00 to empty the /var/log/lastlog logs file. The job must run at 12am everyday.

You can use the command echo “” > /var/log/lastlog to empty the lastlog file and schedule should be 0 0 * * *.

```
- name: Create a cron job to clear last log
  hosts: node00
  tasks:
   - name: Create cron job
     cron:
       name: "Clear Lastlog"
       minute: "0"
       hour: "0"
       job: echo "" > /var/log/lastlog
```

```
- name: Create a cron job to run free.sh script
  hosts: node00
  tasks:
   - name: Create cron job
     cron:
       name: "Free Memory Check"
       minute: "0"
       hour: "*/2"
       job: "sh /root/free.sh"
```

##### Take care about exact name of the cron Check Memory.

```
- name: remove cron job from node00
  hosts: node00
  tasks:
  - name: Remove cron job
    cron:
      name: "Check Memory"
      state: absent
```

Due to some disk space limitations, we want to cleanup the /tmp location on node00 host after every reboot.<br/>
Create a playbook ~/playbooks/reboot.yml to add a cron named cleanup on node00 that will execute after every reboot and will clean /tmp location.<br/>

The command should be rm -rf /tmp/*.

##### Create reboot.yml playbook and it should look like as below.

```
- name: Cleanup /tmp after every reboot
  hosts: node00
  tasks:
   - cron:
      name: cleanup
      job: rm -rf /tmp/*
      special_time: reboot
```

Generated cron expression should be 5 8 * * 0

On node00 we want to keep the installed packages up to date, so we would like to run yum updates regularly.<br/>
Create a playbook ~/playbooks/yum_update.yml and create a cron job as described below:

```
a. Do not add cron directly using crontab instead create a cron file /etc/cron.d/ansible_yum.
b. The cron must run on every Sunday at 8:05 am.
c. The name of the cron must be yum update.
d. Cron should be added for user root
```

Use command yum -y update

##### Create user and admin

```
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
```

##### Create add_user.yml playbook and add below given code

```
- hosts: all
  gather_facts: no
  tasks:
    - user:
        name: 'neymarsabin'
        expires: 1704067199
```

##### Create remove_user.yml playbook and add below given code

```
- hosts: all
  gather_facts: no
  tasks:
    - user:
        name: 'admin'
        state: absent
    - group:
        name: 'admin'
        state: absent
```

### Variable Precedence

Host variable take precedence over group variables.

```
Group Vars  |
Host Vars   |
Play Vars   |
Extra Vars  |
            \/

```

--extra-vars "dns_server=10.5.5.6"  # highest precedence

command lines variable override all the precedence

#### Variable Scope

Scope means accessability or visibility of a variable.

```
1. Variable Scopes - Host 
2. Variable Scopes - Play
3. Variable Scopes - Global   --extra-vars commandline

```

### Register Variables

Store the output of one task and use it later.

`register : result`

#### To Print the Variable
```
debug:
   var: result
```

The registered variable is stored in memory.

```
- hosts: all
  tasks:
     - shell: uptime
       register: uptime_result

     - debug: var=uptime_result.stdout
```

```
---
- hosts: all
  gather_facts: no  
  tasks:    
    - name: stat module help to find the file info
      stat:
        path: /var/run
      register: your_variable

    # for your reference, check the outputs of these
    - debug:
       var=your_variable.stat

    # your code goes here...
    - shell: echo "{{your_variable.stat}}" > /tmp/by_ansible
```

```
tasks:
    - name: alternative way to gather facts about remote host
      setup: filter='ansible_dist*'
      register: facts
    - debug: var=facts.ansible_facts.ansible_distribution
    - shell: echo "{{facts.ansible_facts.ansible_distribution}}" > /tmp/output.txt
```

### Magic Variabels

When Ansible playbooks starts, it first creates three sub processes for each host.<br/>
Before the tasks are run on each host, Ansible goes through a variable interpolation stage,<br/>
where it picks up variables from different sources and associates them to the respective hosts.<br/>
Each variable is only associated to the host on which it was defined. so it is unavilable on the others<br/>

```
hostvars['hostname'].dns_server  .dns server is variable name
hostvars['hostname'].ansible_host
hostvars['hostname'].ansible_facts.architecture
groups
group_names
inventory_hostname
```

```
---
- name: print_dns server
  hosts: all
  tasks:
    - shell: "echo {{hostvars['node01.host'].dns_server}} >> /tmp/variable.txt"
```

#### Copy tempate to different location
```
---
- hosts: node00.host
  gather_facts: false
  tasks:
    - name : hostinfo
      template:
        src: hostinfo.j2
        dest: /root/hostinfo
 ```

 ### Jinja2 in Ansible

 ### Built in Filters in Jinja2

 ```
 abs()
 attr()
 batch()
 join()
 lenght()
 replace()
 select()
 sort()
 tojson()
 trim()
 wordcount()
 ```
Converting YAML to json
Working with file names and directory paths in linux and windows
Passwords and regular expression.

##### Filters - file

```
{{ '/etc/hosts' | basename }}                     => hosts
{{ c:\windows\hosts" | win_basename }}            => hosta
{{ c:\windows\hosts" | win_splitdrive }}          => ["c:", "\windows\hosts"]
{{ c:\windows\hosts" | win_splitdrive | first }}  => "c:"
```

##### How jinj2 works in Playbook

Here we have a simple inventory file with a few variables

Before executing the playbook ansible runs the Playbook through the Jinj2.<br/>
templating engine, providing the set of variables it gathered through inventory parameters.<br/>
The Jinja2 templating engine spits out a new version of the Playbook with all the variable in place.<br/>
which is then used for the actual execution.<br/>

Jinja2 Documentation click [here](https://jinja.palletsprojects.com/en/2.10.x/templates/#builtin-filters)

Ansible Documentation click [here](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html)

```
{
  "names": [
    "Alpha",
    "Beta",
    "Charlie",
    "Delta",
    "Echo"
  ]
}

{% for name in names %}
{{ name }}
{% endfor %}
```
##### Output

Alpha
Beta
Charlie
Delta
Echo

```
{
  "name_servers": [
    "10.1.1.5",
    "10.1.1.6",
    "10.1.1.8",
    "10.8.8.1",
    "8.8.8.8"
  ]
}

{% for name_server in name_servers -%}
nameserver {{ name_server }}
{% endfor %}
```
##### Output

nameserver 10.1.1.5
nameserver 10.1.1.6
nameserver 10.1.1.8
nameserver 10.8.8.1
nameserver 8.8.8.8

```
{
  "hosts": [
    {
      "name": "web1",
      "ip_address": "192.168.5.4"
    },
    {
      "name": "web2",
      "ip_address": "192.168.5.5"
    },
    {
      "name": "web3",
      "ip_address": "192.168.5.8"
    },
    {
      "name": "db1",
      "ip_address": "192.168.5.9"
    }
  ]
}

{% for host in hosts -%}
{{ host.name }} {{ host.ip_address }}
{% endfor %}
```
##### Output

web1 192.168.5.4
web2 192.168.5.5
web3 192.168.5.8
db1 192.168.5.9

##### hostname contains web. Use Jina2 Blocks

```
{
  "hosts": [
    {
      "name": "web1",
      "ip_address": "192.168.5.4"
    },
    {
      "name": "web2",
      "ip_address": "192.168.5.5"
    },
    {
      "name": "web3",
      "ip_address": "192.168.5.8"
    },
    {
      "name": "db1",
      "ip_address": "192.168.5.9"
    }
  ]
}

{% for host in hosts -%}
  {% if "web" in host.name -%}
{{ host.name }} {{ host.ip_address -}}
  {% endif %}
{% endfor %}
```
##### Output

web1 192.168.5.4
web2 192.168.5.5
web3 192.168.5.8

##### Ansible Templates

```
playbook.yml

---
 hosts: web_servers
 tasks:
   - name: Copy index.html to remote servers
     copy:
       src: index.html
       dest: /var/www/nginx-default/index.html

index.html

<!DOCTYPE html>
<html>
<body>
This is a web server
</body>
</html>

index.html.j2 - template 

file extension is used .j2

<!DOCTYPE html>
<html>
<body>
This is {{ name }} server
</body>
</html>

---
 hosts: web_servers
 tasks:
   - name: Copy index.html to remote servers
     template:
       src: index.html.j2
       dest: /var/www/nginx-default/index.html

index.html.j2 - template 

> [!TIP]
> file extension is used .j2

<!DOCTYPE html>
<html>
<body>
This is {{ inventory_hostname }} server
</body>
</html>
```

When the playbook is run ansible first takes the Jinja2 template then does variable interpolation,converts the template file into files to be copied into each server and copies them to the target servers.<br/>

When the playbook is run Ansible creates three `separate subprocesses` for each host.<br/>
Each process createit's own set of parameters for each host by performing it's own `variable interpolation`.<br/>
It then proceeds to `gathering facts`from the host, so each subprocess now has a lot more information about the host.<br/>
Then the process starts executing playbooks Each subproces `execute the playbook` targeting the respetive host.<br/>
Each subprocess executes the template task for the respetive host by taking the index.html.j2 file and performing variable interpolation on it by replacing the variables with theri values.<br/>
Thus generating an index.html file from it. The template module then `copies over the files to the target hosts`.<br/>
The index.html file was just a simple example.<br/>

> [!TIP]
> You can use Jinja2 filters within this. Example: port {{ redis_port  | default('6379') }} if it is not sepcified explicitly.






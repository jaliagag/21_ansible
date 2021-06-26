# Jeff Geerling

third attempt <https://www.youtube.com/watch?v=goclfp6a2IQ&list=PL2_OBreMn7FqZkvMYt6ATmgC0KAGGJNAN>

inventory file: 

```ini
[example]
<ip>
```

replacing default ansible configuration with file ansible.cfg (same dir)

```ini
[defaults]
INVENTORY = inventory
```
Ansible does first **pre_tasks, then roles, then tasks and then post-tasks**.

`ansible-playbook -i inventory main.yam --syntax-check`

## ad-hoc

ad-hoc commands do not run just because we add **-a**; we are ommitting, and ansible allows this, writting the **-m command** module.

`ansible -i inventory example -a "free -h" -u jaliaga` (or `ansible example -a "free -h" -u jaliaga` if we have the ansible.cfg configured). **-K** to pass in the sudo password.

```ini
[example]
192.168.0.185

# group
[multi:backend]
192.168.0.186

# group
[multi:frontend]
192.168.0.187

# variables for group multi
[multi:vars]
ansible_ssh_user=jaliaga

```

`anisble multi -a 'hostname' -u jaliaga` would return 186 and 187. groups of servers defined in the config

ansible forks the command 5 times, to run as quickly as possible. to make it run one server at a time we can user the flag `-f 1` or even increase the default `-f 100`.

get all the information ansible can gather from a server: `ansible example -m setup`

ansible-doc <module> -- view information on a module

limit where the playbook/command is to be applied: `--limit "<ip|hostname>"`

Say that we've launched a bunch of jobs on the server and we want to see the status - `ansible -i inventory db -b -m async_status -a "<jid>"` (jid is for jobid)

shell module will work with pipes and redirections: `ansible -i inventory multi -b -m shell -a "tail /var/log/messages | grep ansible | wc -l"`

```ini
# on the ansible.cfg
[ssh_connection]
pipelining = True
```

Faster playbooks

## playbooks

`ansible-inventory --list -i inventory` what information ansible knows

```yaml
---

```

## handlers

handlers are tasks - the name of the handler is used to notify it to run - `notify: <name of the handler>` - if there are changes, whatever is it that the handler does, it is triggered. handlers are, by default triggered _at the end of the playbook_. to make sure that the handler is run mid-playbook-run, we can add an ansible meta function

```yaml
- name: make sure handlers are flushed immediately
  meta: flush_handlers
```

handlers won't run if the playbook fails; we can add `--force-handlers` to make sure that the handlers are execute, no matter the outcome of the playbook.

handlers can be used to make api calls (slack, for instance)

## variables

we can turn off become for any task by adding `become: false`

calling variables on another file:

```yaml
# file 1
- name: playbook1
  hosts: all
  become: true

  vars:
    - vars/main.yaml

# file 2
---
key: value
```

## vault

`ansible-vault encrypt <file>` - it asks for a password ('test'). it encrypts the file. when running a playbook, we have to add `--ask-vault-pass`. we can use `--vault-password-file <file>` if we have the password stored in a file. we can also run `ansible-vault decrypt <file>` to return it to its original text - we can edit the file, while encrypted `ansible-vault edit <file>`

## conditionals

## roles

## modules

- ping module: `ansible -i inventory example -m ping -u jaliaga`
- yum module: `ansible multi -m yum -a 'name=ntp state=present'`
- service module: `ansible multi -m service -a 'name=ntpd state=started enabled=yes'`
- set up a user for mysql: `ansible -i inventory db -b -m mysql_user "name=django host=% password=12345 priv=*.*:ALL state=present"`
- cron module: `ansible -i inventory multi -b -m cron -a "name=wharever hour=4 job=/home/script.sh"`
- git repo: `ansible -i inventory multi -b -m git -a "repo-github_url_goes_here destination=/opt/apps/destination update=yes version=1.2.4"`

## flags

- _-B_: maximum alloted time for a job to run; if it's exceeded, ansible stops the run.
- _-P 0_: runs the command on the server side as nohup


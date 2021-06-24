# Jeff Geerling

third attempt

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

## ad-hoc

`ansible -i inventory example -a "free -h" -u jaliaga` (or `ansible example -a "free -h" -u jaliaga` if we have the ansible.cfg configured). **-K** to pass in the sudo password.

## playbooks

tasks

## modules

- ping module: `ansible -i inventory example -m ping -u jaliaga`





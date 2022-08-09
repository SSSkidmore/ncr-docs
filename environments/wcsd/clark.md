
# WCSD Clark Environment

This environment consists of one teacher router environment and 15 student environments.
Each environment has a single router that serves as the gateway to a single Windows 10
client. The router has a `dnsmasq` service that provides DNS and DHCP to it's single
subnet. A table of users and internal subnets is below. All the Windows 10 clients
have a DHCP reservation on their respective 192.128.x.x network. The IP should 
always be 192.168.x.50.

```text
name    vlan    ip      lan
wcsd000 1999    172.21.0.9/24   192.168.9.0/24
wcsd001 2001    172.21.0.10/24  192.168.10.0/24
wcsd002 2002    172.21.0.11/24  192.168.11.0/24
wcsd003 2003    172.21.0.12/24  192.168.12.0/24
wcsd004 2004    172.21.0.13/24  192.168.13.0/24
wcsd005 2005    172.21.0.14/24  192.168.14.0/24
wcsd006 2006    172.21.0.15/24  192.168.15.0/24
wcsd007 2007    172.21.0.16/24  192.168.16.0/24
wcsd008 2008    172.21.0.17/24  192.168.17.0/24
wcsd009 2009    172.21.0.18/24  192.168.18.0/24
wcsd010 2010    172.21.0.19/24  192.168.19.0/24
wcsd011 2011    172.21.0.20/24  192.168.20.0/24
wcsd012 2012    172.21.0.21/24  192.168.21.0/24
wcsd013 2013    172.21.0.22/24  192.168.22.0/24
wcsd014 2014    172.21.0.23/24  192.168.23.0/24
wcsd015 2015    172.21.0.24/24  192.168.24.0/24
```


## Teacher Router Firewall
The teacher router will not forward packets for the following application
layer protocols:

- HTTP
- HTTPS
- LDAP
- SMTP

HTTP connections to the Debian repositories are allowed. This allows students
to install software with `apt`

## Teacher Router SAMBA Server
Files can be shared from the teacher router to the students using SAMBA.
The `/wcsd/public` directory is a publicly shared with all the Windows 10 
clients.

A student will have to enter `\\wcsd-tr.ncr` into Windows File Explorer.


## Teacher Router Ansible
The teacher router has an Ansible playbook for developer operations style orchestration
of the student routers and Windows 10 clients. The playbook is located in `/wcsd/playbook`.
Ad-hoc commands are the easiest way to run a shell command on all hosts

### Linux
```
cd /wcsd/playbook
ansible routers -m shell -a 'hostname' #Run `hostname` on all the student routers.
ansible routers -m shell -a 'apt install -y tmux' #Install tmux on all student routers.
```

### Windows
```
cd /wcsd/playbook
ansible win10 -m win_shell -a 'hostname' # Run `hostname` on each win10 client.
```

Ad-hoc commands are admittedly less useful for managing windows. The playbook
contains tasks for installing packages on Windows. The playbook we wrote includes
a task for installing 7zip. You can run it with the commands below.

```
cd /wcsd/playbook
ansible-playbook -l win10 site.yml
```

Example Output:
```
PLAY [routers] ******************************************************************************************************
skipping: no hosts matched

PLAY [win10] ********************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************
ok: [wcsd000-win10]
ok: [wcsd003-win10]
...

TASK [student-win10 : Install 7zip from HTTP] ***********************************************************************
ok: [wcsd000-win10]
ok: [wcsd002-win10]
...

PLAY RECAP **********************************************************************************************************
wcsd000-win10              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
wcsd001-win10              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```



The 7zip task is located in `/wcsd/playbook/roles/student-win10/tasks/main.yml`

```
- name: Install 7zip from HTTP
  ansible.windows.win_package:
    path: http://172.21.0.1/7z2201-x64.exe
    product_id: 7-Zip
    arguments: /S
    state: present
```

The `7z2201-x64.exe` installer is located on the teacher router in the `/var/www/html` directory.

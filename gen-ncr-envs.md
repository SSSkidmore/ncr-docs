# Generating environments on NCR 9/02/22
---

## Create environments: 
```bash
root@wcsd-vnet: cd /etc/wcsd # main directory

mkdir instances/matilainen # could add to script, mkdir $group

vim /etc/wcsd/gen_config.py
# edit DSTDIR: "/etc/wcsd/instnaces/matilainen" 
# could add to script, DSTDIR=/etc/wcsd/instances/$group

cat instances/containerMasterListTest.csv
# environments listed in the order they were created
# find last created environment: ccsd,lockett,student,160,0
cd instances/lockett && ls
cat ccsd-student160-server
cat ccsd-student160-client # an environment (student160) may have multiple instances (server,client)
# note the following values: 
# last id: "160"
# last wan_ip: 10.99.0.209/23
# last man_ip: 172.16.1.113/23
# last dnat_port: 6220
# last lan_vlan: 259

# and determine the following arguments: 
# --start-id=(last id + 1)=161
# --last-id=(start-id + (count - 1))=175 # count of students/environments - not instances
# --start-wan-ip=(last wan_ip + 1)=10.99.0.210/23
# --start-man-ip=(last man_ip +1)=172.16.1.114/23
# --start-dnat-port=(last dnat_port + 1)=6221
# check iptables for existing dnat port/man-ip: 
iptables -t nat -L --line-numbers | grep 172.16.1.114 
iptables -t nat -L --line-numbers | grep 6221
# --start-vlan=(last lan_vlan + 1)=260

# I think we could script this by modifying a master file whenever we generate 
# environments so that we pull next available values from the mast list rather 
# than having to manually math them whenever we gen new ones. 

cd instances/matilainen
../../gen_config.py --start-id=161 --last-id=175 --start-wan-ip=10.99.0.210/23 \
        --start-man-ip=172.16.1.114/23 --start-dnat-port=6221 --start-vlan=260 \
        --org="wcsd" --group="matilainen" --clients=0

# this generated the yml config files we need to create instances
# check a few files
cat wcsd-student001-server
# if need to make a quick fix to config files
#for FILE in *; do echo $FILE; done # check bash loop first
#for FILE in *; do sed -i 's/wcsd-/matilainen-/g' $FILE; done # replace a string

# modify add_server.sh rootfs src for kali or debian templates
vim add_server.sh
# uncomment for debian or kali servers
CLIENT_ROOTFS_SRC="/var/lib/machines/kali-server.tmpl/"
# save

# create server instances
for FILE in *; do echo $FILE; done # check bash loop first
for FILE in *; do ../../add_server.sh $FILE; done # create instances

# repeat prn for clients with add_client.sh

# add to containerlist.txt
cd containerScripts
echo wcsd-matilainen-student{001..015}-server| sed 's/ /\n/g' >> containerlist.txt

# check wan ip
ping wcsd-matilainen-student001-server

# create .man dns entries for wcsd-vnet
for FILE in instances/matilainen/*; do ./add_dns_entries.sh $FILE; done
systemctl restart dnsmasq
ping wcsd-matilainen-student001-server.man

cat /var/www/html/containerCheck.html

# from NCR remote web interface, connect to a server from a different group
# connected to ccsd-student01-server, open a terminal:
# check vnc connection to one of the man-ips just created (at port 5901):
vncviewer 172.16.1.114:5901 
# vnc password: student001
```
---
## Create Django users
```bash
root@ncr-0: machinectl shell ncr-remote
su gnt-remote && cd # become gnt-remote user
source venv/bin/activate
cd ncr
# generate user passwords
for X in {001..015}; do echo wcsd-matilainen-student$X,$(curl https://www.dinopass.com/password/strong) >> wcsdMatilainenCreds.csv; done
./addUsersFromCSV.sh wcsdMatilainenCreds.csv
# can also use interactive shell prn: ./manage.py shell

# check that users created in django online interface
# login at https://ncr-remote.cse.unr.edu/admin

# if can't login as django admin: 
# make your username a superuser and staff: 
(venv) gnt-remote@ncr-remote:~/ncr$ ./manage.py shell
In [1]: from django.contrib.auth.models import User
In [2]: User.objects.filter(is_superuser=True)
Out[2]: <QuerySet [<User: newellz2>, <User: jthom>, <User: sgerard_admin>, <User: tfinnegan>, <User: kbettridge>, <User: imclain>, <User: gsferrazza>]>
In [3]: User.objects.filter(is_staff=True)
Out[3]: <QuerySet [<User: newellz2>, <User: jthom>, <User: sgerard_admin>, <User: tfinnegan>, <User: kbettridge>, <User: imclain>, <User: gsferrazza>]>
In [4]: User.objects.get(username='sskidmore')
Out[4]: <User: sskidmore>
In [5]: user = User.objects.get(username='sskidmore')
In [6]: user.is_superuser = True
In [7]: user.is_staff = True
In [8]: user.save()
In [9]: User.objects.filter(is_superuser=True)
Out[10]: <QuerySet [<User: newellz2>, <User: sskidmore>, <User: jthom>, <User: sgerard_admin>, <User: tfinnegan>, <User: kbettridge>, <User: imclain>, <User: gsferrazza>]>
In [11]: exit
# login at https://ncr-remote.cse.unr.edu/admin
```
---
## Create ganeti user accounts and connections
```bash
# manually add connection group 
# from https://ncr-remote.cse.unr.edu select settings>Connections
# select New Group (note, this is a connection group, not user group)
# Location: ROOT, Type: Organizational; can clone from existing and change name
# TODO: Ask Kameron, should we be creating user groups manually from web interface too?

root@ncr-0:~/guacman
cd scripts
# create csv of environment data:
# find last id used
cat data/locket.csv
# adjust last number, i.e. 5899 in  $((10#$NUM*2+5899)) becomes 6220
# example with client: for NUM in {001..160}; do echo "ccsd-student$NUM,ccsd-student$NUM-server,$((10#$NUM*2+5899)),ccsd-student$NUM-client,$((10#$NUM*2+5900))" >> data/ccsd.csv; done
# server only: 
for NUM in {001..015}; do echo "wcsd-matilainen-student$NUM,wcsd-matilainen-student$NUM-server,$((10#$NUM+6220))" >> data/matilainen.csv; done
cat data/matilainen.csv # check
# create custom script from previous:
cp addCcsdToGuac.sh addMatilainenToGuac.sh
vim addMatilainenToGuac.sh # edits:
lines=`cat data/matilainen.csv`
--cgroup "Matilainen Containers"
# remove client connection lines
./addMatilainenToGuac.sh

machinectl shell ncr-remote
su gnt-remote
cd ~/ncr
cat wcsdMatilainenCreds.csv
# check user ganeti login at https://ncr-remote.cse.unr.edu/accounts/login/
# under Connecting to Your Virtual Machines, select Click here to connect

# check admin django connection from https://ncr-remote.cse.unr.edu:8443/#/
```
---
## Contact instructors
Provide: 
- login url: https://ncr-remote.cse.unr.edu/accounts/login/
- usernames/passwords from gnt-remote@ncr-remote:~/ncr/wcsdMatilainenCreds.csv
```bash
# TODO: Ask Kameron: 
# - anything else credentials wise that instructors need provided?
# - Do we have any std copy docs that we are providing instructors? (not seeing much here: https://github.com/Nevada-Cyber-Range/ncr-docs)
```

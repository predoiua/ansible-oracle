# Oracle for Ansible

This role downloads, installs and configures Oracle Database 11g Release 2 for
CentOS 6. The included tasks are a rough port of an internal shell script, which
was itself a rough port of Oracle's installation instructions and
recommendations.

## Requirements

- CentOS 6.4+
- Ansible 1.4
- Oracle Database 11gR2 [installation content](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/112010-linx8664soft-100572.html)

## Variables

See the [default variables](defaults/main.yml), which are extensively
commented.

## Example

    ---
    - name: set up an oracle database
      sudo: yes
      vars:
        oracle_path: /u01/app           # ORACLE_BASE will be /u01/app/oracle
        oracle_db_name: my_special_db   # ORACLE_SID will be my_special_db
        oracle_db_home: special_home    # ORACLE_HOME will be /u01/app/oracle/product/11.2.0/special_home
        oracle_db_user: devuser
        oracle_db_pass: AnAwesomeAndAmazingP4ssw0rd
        oracle_db_syspass: AMor3AwesomeAndAmazingP4ssw0rd
        oracle_installer_uri: http://my.host # Ansible will download http://my.host/linux.x64_11gR2_database_1of2.zip
      roles:
        # more roles here
        - oracle

## TODO

 - Handle multiple runs with different `oracle_db_home` and/or `oracle_db_name`
   vars, instead of skipping the whole installation process.
 - Optionally allow Oracle installer file downloads from S3 (e.g. with
   [the s3 module](http://docs.ansible.com/s3_module.html)).

##Docker

~~~host
docker run -it --rm --privileged -p 1521:1521 -v /media/predoiua/Data-ext/tmp:/temp/oracle centos:6.6 /bin/bash
~~~

~~~docker
if [[ ! -d /temp/oracle/database ]]; then echo "error. no DB kit"; exit 1; fi

echo "adding ansible user"
useradd -m -d /home/ansible -s /bin/bash ansible
mkdir -p /home/ansible/.ssh
chmod 700 /home/ansible/.ssh
touch /home/ansible/.ssh/authorized_keys
chown -R ansible:ansible /home/ansible
echo "ansible ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "ansible:ansible" | chpasswd

yum install -y rsyslog sudo
yum install -y openssh-server openssh-clients

vi /etc/ssh/sshd_config #TCPKeepAlive
vi /etc/fstab #set ,size=4G
umount /dev/shm
mount /dev/shm

service sshd start
~~~

~~~host
(mkdir roles;cd !$; ln -s .. oracle)
ssh-keygen -f "/home/predoiua/.ssh/known_hosts" -R 172.17.0.2
ssh-copy-id ansible@172.17.0.2

ansible -i hosts/ora.ini ora -m ping
ansible-playbook -i hosts/ora.ini ora.yml
~~~

https://docs.oseems.com/general/application/ssh/disable-timeout
http://dba.stackexchange.com/questions/48971/target-database-memory-exceed-available-shared-memory


/temp/oracle/database/runInstaller -silent -force -ignoreSysPrereqs -ignorePrereq -responseFile /temp/oracle/db_install.rsp


vi /temp/oracle/db_install.rsp
INVENTORY_LOCATION=/u01/app/oraInventory
ORACLE_HOME=/u01/app/oracle/product/11.2.0/oracle_db_home

(cd /u01/app/oraInventory; rm -rf *; cd /u01/app/oracle/product/11.2.0/oracle_db_home; rm -rf *)

cat /proc/sys/kernel/shmmax
vi /etc/sysctl.conf

sysctl -p

http://www.idevelopment.info/data/Unix/Linux/LINUX_AddGNOMEToCentOSMinimalInstall.shtml
yum -y groupinstall "Desktop" "Desktop Platform" "X Window System" "Fonts"
yum install -y xterm

ssh -l oracle -X -p 22 172.17.0.2


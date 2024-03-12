THIS IS PREREQ CLOUDERA FOR CENTOS 7.9


=======================================================
CREATE LOCAL REPO CENTOS 7.9
=======================================================
cd /etc/yum.repos.d
vi centos-dvd.repo
[dvd-BaseOS]
name=DVD for CENTOS
baseurl=file:///media/dvd
enabled=1
gpgcheck=0

save and exit

yum clean all
yum repolist all

=======================================================
install Python3.8
=======================================================
yum install gcc openssl-devel bzip2-devel libffi-devel zlib-devel -y
tar -zxf Python3.8.17
mv Python3.8.17 /opt/
cd /opt/Python3.8.17
sudo ./configure --enable-optimizations
sudo make altinstall

Host CM ->> pip3.8 install psycopg2-binary

=======================================================
FAIL System: tuned is running
FAIL System: tuned auto-starts on boot
=======================================================
sudo systemctl stop tuned
sudo systemctl disable tuned


=======================================================
FAIL System: /proc/sys/vm/swappiness should be 1. Actual: 30
=======================================================
sysctl vm.swappiness=1       <<---- TEMPORARY  (WILL BE LOST AFTER REBOOT)

vi /etc/sysctl.conf			<<---- PERMANENT CHANGE
Add or modify the line to set 
vm.swappiness=1

apply change without reboot
sysctl -p

CHECK
cat /proc/sys/vm/swappiness


==========================================================
FAIL System: /proc/sys/vm/overcommit_memory should be 1. Actual: 0
==========================================================
sysctl vm.overcommit_memory=1       <<---- TEMPORARY  (WILL BE LOST AFTER REBOOT)

vi/etc/sysctl.conf	          <<---- PERMANENT CHANGE
Add or modify the line to set
vm.overcommit_memory=1

Apply Changes without reboot
sudo sysctl -p

check
cat /proc/sys/vm/overcommit_memory


==========================================================
FAIL System: /sys/kernel/mm/transparent_hugepage/defrag should be disabled need to change permanent
==========================================================
1. Edit the GRUB configuration:
Open the GRUB configuration file in a text editor. You'll typically find it at 
vi /etc/default/grub
check line GRUB_CMDLINE_LINUX put this in append -->> transparent_hugepage=never
Save and exit
2. Update GRUB:
apply change
grub2-mkconfig -o /boot/grub2/grub.cfg
3. Disable THP defragmentation:
edit
vi /etc/rc.d/rc.local
add this
if [ -e /sys/kernel/mm/transparent_hugepage/defrag ]; then
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

save and exit
4. Set permissions:
rc.local executable
chmod +x /etc/rc.d/rc.local
5. REBOOT

==========================================================
FAIL System: /sys/kernel/mm/transparent_hugepage/enabled should be disabled. Actual: always
==========================================================
vi /etc/systemd/system/disable-thp.service

[Unit]
Description=Disable Transparent Huge Pages (THP)
DefaultDependencies=no
Before=sysinit.target local-fs.target
ConditionPathExists=/sys/kernel/mm/transparent_hugepage/enabled

[Service]
Type=oneshot
ExecStart=/bin/sh -c "echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled && echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag"
RemainAfterExit=true

[Install]
WantedBy=basic.target

====
systemctl daemon-reload
systemctl enable disable-thp.service

reboot

==========================================================
FAIL  System: chronyd is not running
==========================================================
yum install chrony
systemctl start chronyd
systemctl enable chronyd
systemctl status chronyd

==========================================================
FAIL  Network: IPv6 is not supported and must be disabled
==========================================================
vi /etc/sysctl.conf

# Disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

sysctl -p

cat /proc/sys/net/ipv6/conf/all/disable_ipv6

Update network configuration (Optional):

You may also want to ensure that IPv6 is disabled in network configuration files.
Check the files in /etc/sysconfig/network-scripts/ and remove any IPv6 related configuration from them.

systemctl restart network

check
ip addr


==========================================================
FAIL Network: firewalld should not be running
FAIL Network: firewalld should not auto-start on boot
==========================================================
systemctl stop firewalld
systemctl disable firewalld


==========================================================
FAIL Network: nscd is not running
FAIL Network: nscd does not auto-start on boot
==========================================================
yum install nscd

systemctl start nscd
systemctl enable nscd



==========================================================
Install Postgresql
==========================================================
sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install postgresql14-server postgresql14-contrib
sudo systemctl start postgresql-14
sudo systemctl enable postgresql-14
sudo vi /var/lib/pgsql/14/data/pg_hba.conf
host    all             all             0.0.0.0/0               md5
sudo vi /var/lib/pgsql/data/postgresql.conf
listen_addresses = '*'
sudo -u postgres psql


CREATE ROLE poc_scm LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_amon LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_rman LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_hue LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_hive LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_ranger LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_das LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_oozie LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_srs LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_smm LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_ssb LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_mve LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_rangerms LOGIN PASSWORD 'cloudera';
CREATE ROLE poc_yarn LOGIN PASSWORD 'cloudera';

CREATE DATABASE poc_scm OWNER poc_scm ENCODING 'UTF8';
CREATE DATABASE poc_amon OWNER poc_amon ENCODING 'UTF8';
CREATE DATABASE poc_rman OWNER poc_rman ENCODING 'UTF8';
CREATE DATABASE poc_hue OWNER poc_hue ENCODING 'UTF8';
CREATE DATABASE poc_metastore OWNER poc_hive ENCODING 'UTF8';
CREATE DATABASE poc_ranger OWNER poc_ranger ENCODING 'UTF8';
CREATE DATABASE poc_das OWNER poc_das ENCODING 'UTF8';
CREATE DATABASE poc_oozie OWNER poc_oozie ENCODING 'UTF8';
CREATE DATABASE poc_srs OWNER poc_srs ENCODING 'UTF8';
CREATE DATABASE poc_smm OWNER poc_smm ENCODING 'UTF8';
CREATE DATABASE poc_ssb OWNER poc_ssb ENCODING 'UTF8';
CREATE DATABASE poc_mve OWNER poc_mve ENCODING 'UTF8';
CREATE DATABASE poc_yarn OWNER poc_yarn ENCODING 'UTF8';
CREATE DATABASE poc_rangerms OWNER poc_rangerms ENCODING 'UTF8';

ALTER DATABASE poc_metastore SET standard_conforming_strings=off;
ALTER DATABASE poc_oozie SET standard_conforming_strings=off;


https://docs.cloudera.com/cdp-private-cloud-base/7.1.9/installation/topics/cdpdc-configuring-starting-postgresql-server.html
==========================================================
INSTALL ORACLE JDK
==========================================================
download Oracle JDK dari website bentuk TGZ
scp jdk-8u211-linux-x64.tar.gz root@192.168.90.196:/root
mkdir /usr/java
extract file tgz
tar -zxf jdk-8u211-linux-x64.tar.gz -C /usr/java
vi /etc/profile
add line 
export JAVA_HOME=/usr/java/jdk1.8.0_211
export PATH=$JAVA_HOME/bin:$PATH
source /etc/profile
java -version


==========================================================
KRB 5 WORKSTATION
==========================================================
sudo yum install sssd oddjob oddjob-mkhomedir oddjob-mkhomedir adcli samba samba-common-tools krb5-workstation




=======================================================
Configuring /tmp directory for cluster hosts
=======================================================
chmod 1777 /tmp




==========================================================
CLOUDERA MANAGER REPO SETTING
==========================================================
/etc/yum.repos.d/cloudera-manager.repo
[cloudera-manager]
name=Cloudera Manager 7.11.3
baseurl=http://192.168.90.18/repository/cm7.11.3.4/cm7.11.3.4/
gpgcheck=0

yum install cloudera-manager-server cloudera-manager-agent cloudera-manager-daemons



==========================================================
CONNECT DATABASE CM
==========================================================
/opt/cloudera/cm/schema/scm_prepare_database.sh -h 192.168.90.196 postgresql poc_scm poc_scm


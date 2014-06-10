zimbra_scratch_install
======================

Zimbra Scratch Installation

Installing Zimbra Collaboration Server 8 on CentOS 6.5 (http://mirror.centos.org/centos/6.5/os/x86_64/)

Steps:
1. Update system
#yum update

Install ssh service
#yum -y install openssh-server openssh-clients
#chkconfig sshd on
#service sshd start


2. Install prerequisites
yum install sudo sysstat libidn gmp libtool-ltdl libaio vixie-cron nc perl libstdc++.i686

3. Modify /etc/hosts file and Also, be sure to setup a DNS MX record in your zone file

127.0.0.1 localhost localhost.localdomain
#::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
115.146.72.28 server01.ict.com server01

4. Disable start-up services
4a. chkconfig postfix off
4b. service postfix stop
4c. chkconfig sendmail off
4d. service sendmail stop

5. /etc/sudoers – Comment “Defaults requiretty” out
# Defaults requiretty

6. Disable firewall or Allow zimbra ports
7a. chkconfig iptables off && chkconfig ip6tables off
or
7b. Configure Iptables Firewall
enable zimbra ports
#mkdir /opt/sec      - create a folder "sec"

#vi imailaccess

------------
#!/bin/bash
#
 iptables -F

 iptables -A INPUT -p tcp --dport 22 -j ACCEPT -s 110.140.70.10,110.140.70.11

 iptables -P INPUT DROP
 iptables -P FORWARD DROP
 iptables -P OUTPUT ACCEPT

 iptables -A INPUT -i lo -j ACCEPT

 iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

 iptables -t nat -A PREROUTING -p tcp --dport 80 -i eth0 -j REDIRECT --to-port 443

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 161 -j ACCEPT -s 110.140.70.10,110.140.70.11

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 162 -j ACCEPT -s 110.140.70.10,110.140.70.11

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 25 -j ACCEPT

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 110 -j ACCEPT

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 143 -j ACCEPT

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 389 -j ACCEPT

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT -s 115.140.72.10/28         | #firenet
 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT -s 115.115.164.10/28        | #JOCK


 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 465 -j ACCEPT

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 993 -j ACCEPT
 
 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 995 -j ACCEPT

 iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 7071 -j ACCEPT -s 110.140.70.10,110.140.70.11

------------
#chmod +x /opt/sec/imailaccess
#./imailaccess

7. /etc/selinux/config – Disable SeLinux
SELINUX=disabled
Note: Reboot is required

8. Download ZCS and Extract installation package to /usr/src
#yum install wget
#cd /usr/src
#wget http://files2.zimbra.com/downloads/8.0.7_GA/zcs-8.0.7_GA_6021.RHEL6_64.20140408123911.tgz
#tar xvf zcs-8.x.y.z.tgz

9. ./install.sh – To Install ZCS
Install zimbra-snmp [Y] n ->rest default value

9a. Change domain name
9b. Change admin password – Type “3″ and then “4″

10. To Check zimbra status
su – zimbra
zmcontrol status
12. To stop and start zimbra

su – zimbra
zmcontrol stop
zmcontrol start

11. Open your web browser and go to https://ZCS_IP_Address

12.
Configure > Class of Service
a. Define new class of service as per requirement

Configure > Domains > General Information
a. Time zone: 
b. Default Class of Service:

Configure > Domains > GAL
a. Most results returned by GAL search: 5

Configure >  Servers > MTA
a. Milter Server
enable
Milter server bind address: 






------
To restrict users to certain domains:
#vi /opt/zimbra/conf/zmconfigd/smtpd_recipient_restrictions.cf
check_sender_access hash:/opt/sec/restricted_senders

# vi /opt/sec/restricted_senders
trainer@ict.com   local_restrict
accounts@ict.com  local_restrict
->
OR
for all users in domain
ict.com   local_restrict
->

#vi /opt/zimbra/conf/zmconfigd.cf
Find the section labeled SECTION mta and enter the following two lines directly below
        POSTCONF        smtpd_restriction_classes       local_restrict
        POSTCONF        local_restrict  FILE    		postfix_check_recipient_access.cf

#vi /opt/zimbra/conf/postfix_check_recipient_access.cf
check_recipient_access hash:/opt/sec/local_domains, reject

#vi /opt/sec/local_domains
gmail.com   OK
hotmail.com     OK
ict.com  OK




#chmod +x /opt/sec/local_domains
#chmod +x /opt/sec/restricted_senders
#chown zimbra:zimbra /opt/sec/local_domains
#chown zimbra:zimbra /opt/sec/restricted_senders

#su zimbra
#postmap /opt/sec/restricted_senders
#postmap /opt/sec/local_domains

#zmmtactl stop 
#zmmtactl start

->
if you need to undo this configuration:
in zmmta.cf
postconf -e smtpd_restriction_classes=' '

zmmtactl reload
->

Ref: https://wiki.zimbra.com/wiki/Restrict_users_to_certain_domain
-------

To RestrictPostfixRecipients
http://wiki.zimbra.com/index.php?title=RestrictPostfixRecipients
------






-------------
change hostname zimbra
#vi /etc/hosts
#vi /etc/sysconfig/network

# su - zimbra

$ zmcontrol stop

$ /opt/zimbra/libexec/zmsetservername -force -n new.hostname.com
(you will receive some errors but ignore them)
OR
$ /opt/zimbra/libexec/zmsetservername -o old.hostname.com -n new.hostname.com
(it will not show any error and work fine)


Set Default Domain-
$ zmprov -l mcf zimbraDefaultDomainName newdomain.com


Generate New ssh key for zimbra and update it

$ zmsshkeygen

$ zmupdateauthkeys

$ zmcontrol start




